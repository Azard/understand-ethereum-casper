# Casper 数据结构与投注出块

上一章讲到了 Casper 的基本情况，这一章讲一讲 Casper 的基础数据结构和投注流程。

为此，我们首先创建一个最小的 PoS 算法能够满足第一章设计目标中的第一点 \(PoS\) 和第二点 \(快速出块\)。

如果把 PoS 比喻为一个大赌场，那么每个参与 PoS 的验证人就是赌徒，赌徒当然需要将代币作为赌资进行“投注” \(deposit\) 才能进行参与。

## 验证人池

我们所接触的最重要的数据结构是验证人池，可以将验证人池理解为一个保存了所有参与 PoS 的验证人的集合，在使用 Go 重写 Casper 后，一个验证人可以用如下 Go 代码表示 \[3\]。

```go
type Validator struct {
    // 投入的代币数量
    Deposit *big.Rat
    // validator 加入的 dynasty
    Dynasty_start uint64
    // validator 退出的 dynasty
    Dynasty_end uint64
    // validator 的签名地址
    Addr string
    // 收回代币的地址
    Withdrawal_addr string
    // 该 validator 提交的上一个时间戳
    Prev_commit_epoch uint64
}
```

包括了每个验证人参与 PoS 的代币数量，加入 PoS 的 dynasty \(可以理解为周期版本号，验证人池变更的最小时间单位\)，退出的 dynasty，验证人的签名地址，收回代币的地址，以及该验证人参与 PoS 的前一个时间戳。

验证人池则由若干 Validator 组成，可以用集合 \(Set\) 表示。

## 投注

现存在一个“Casper 合约”，这个合约会保存并跟踪 “验证人池” \(validator set\)，该 Casper 合约被包含在创世块 \(genesis block\) 中并且没有权限要求 \(公开的\)，调用这个 Casper 合约是验证一个区块头部的第一步。因此，初始化状态的验证人池被定义在创世块并且能够被如下函数 \(算子\) 修改 \(后文为了避免歧义，我们称 Casper 算法的重要操作函数为 Casper 算子\) ：

**deposit\(bytes validation\_code, bytes32 randao, address withdrawal\_address\)**

该算子接受的参数如下：

* validation\_code: 一串以太虚拟机 \(EVM\) 代码，用于验证区块和被其签名的一致性消息，可以看做是公钥。
* randao: 32个 byte 的哈希值用于选取领导 \(leader selection，详见下文\) 。
* withdrawal\_address: 用于收回投注资金的地址。

准确的说，validation\_code 实现了一个智能合约，输入是 block header hash 和一个签名，签名有效返回1，否则返回0。这个机制允许每个验证人使用任意的签名方式，例如多重签名，或者 Lamport 签名抵抗量子计算机攻击。这段智能合约会使用 CALL\_BLACKBOX 虚拟机操作码保证黑盒运行，不会被外部状态影响。

randao 是一个经常了连续 sha3 计算的随机值，可以认为是一个真正的纯随机32位字符串。每个验证人的 randao 都被保存在 Casper 合约中。

如果全部的参数都被接收，验证人池会在下下个时间戳增加一个验证人。例如，如果 deposit 在 n 时刻被调用，那么验证人池会在 n+2 时刻添加验证人，validation\_code 的哈希值被用作验证人 ID \(又称为 vchash\)，每个验证人都会有一个独立的 ID。

## 出块

Casper 合约还有三个相关算子：

**startWithdrawal\(bytes32 vchash, bytes sig\): **开始执行收回流程，需要传入验证人 ID 和能通过 validation\_code 的私钥。**  
finishWithdrawal\(bytes32 vchash\): **收回代币到 deposit 时指定的收回地址，包括出块奖励和作弊惩罚。**  
getValidator\(uint256 skips\): **返回第 skips 的 validator 的 validation code，该验证人被指定创建下一个区块。如果一个 validator 无法创建下一个区块，则选择排在后面的一个 validator。

选择出块人是通过完全的伪随机算法选择的，随机种子是一个全局的 globalRandao。

到目前为止，一个区块必须包含如下的额外字段：

&lt;vchash&gt; &lt;randao&gt; &lt;sig&gt;

![](/assets/blockTable.png)![](/assets/blockTree.png)

创建具体区块所需要的最短时间可以简单计算为：GENESIS\_TIME + BLOCK\_TIME \* &lt;区块高度&gt; + SKIP\_TIME \* &lt;创世区块后所有的验证者 skip 数&gt;。

在实际过程中，这意味着，一旦发布了某个区块，那么下一个区块的 0-skip 验证者会在 BLOCK\_TIME 秒之后发布，同理，1-skip 验证者则在 BLOCK\_TIME + SKIP\_TIME 秒之后发布，以此类推。

如果一个验证者发布一个区块太早，其他的验证者会忽视该区块，直到在规定的时间之后，才会处理该区块（较小的 BLOCK\_TIME 和较长的 SKIP\_TIME 之间的不对称性，可以确保在正常情况下，区块的平均保留时间可以非常短，而在网络延迟更长的情况下，也可以保证网络的安全性\)。

如果一个验证者创建了一个包括在链内的区块，他们得到的区块奖励相当于此周期内活动验证者集合内以太的总量乘上REWARD\_COEFFICIENT \* BLOCK\_TIME。因此，如果验证者总是正确的履行职责，REWARD\_COEFFICIENT 本质上成为验证者的“预期每秒收益率”，乘上3200万得到近似的年化收益率。如果一个验证者创建了一个不包括在链内的区块，之后，在将来的任意时间 \(直到验证者调用 withdraw 函数为止\) 该区块标头可以作为一个 “dunkle” 通过 Casper 合约的 includeDunkle 函数被包在链内；这使得验证者损失了相当于区块奖励的钱数（以及向包括 dunkle 在内的当事方提供一小部分罚款作为激励）。因此，验证者应当在确定该区块在链内的可能性超过50%，才实际创建该区块。验证者的累计保证金，包括奖励和罚款，存储在Casper合约内。

“dunkle”机制的目的是为了解决权益证明中的“零赌注”的问题，其中，如果没有罚款，只有奖励，那么验证者将被物质激励，试图在每个可能的链上创建区块。在工作量证明场景下，创建区块需要成本，并且只有在“主链”上创建区块才有利可图。Dunkle机制试图复制工作量证明中的经济理论，针对创建非主链上区块进行人工罚款，来代替工作量证明中电费用的“自然罚款”。

假设一个固定大小的验证者集合，我们可以很容易的定义分叉选择规则：计算区块数，最长链胜出。假设验证者集合可以变大和缩小，但是，该规则就不太适用了，因为作为少数支持的分叉的速度一个时期以后将像多数支持的分叉一样。因此，我们可以用计算的区块数代替定义的分叉选择规则，给每个区块一个相当于区块奖励的权重。因为区块奖励与积极验证的以太总量成正比，这确保了更积极验证以太的链得分增长速度更快。

我们可以看出，这条规则可以用另一种方式很方便的理解：基于价值损失的分叉选择模型。该原则是：我们选择验证者赌最多价值的链，就是说，验证者承认除了上述的链外，所有其他链都会损失大量的资金。我们可以等同认为，这条链是验证者失去资金最少的链。在这样一个简单的模式中，很容易看出这如何简单的对应着区块权重为区块奖励的最长链。该算法是尽管简单，但用于 PoS 的实现来说也足够高效。

## 参考

\[1\] 最新以太坊紫皮书: https://docs.google.com/document/d/1maFT3cpHvwn29gLvtY4WcQiI6kRbN\_nbCf3JlgR3m\_8  
\[2\] 老版以太坊紫皮书中文版: http://8btc.com/thread-40113-1-1.html  
\[3\] 复杂美 Casper-Go 项目源代码

