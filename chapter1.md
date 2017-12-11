# Casper 简介与概览

Casper 是知名开源区块链项目以太坊 \(Ethereum\) \[1\] 的共识算法，是以太坊转型为全面 PoS \(Proof-of-Stake\) 的基础理论支持和实现，同时也是以太坊 2.0 计划 \[2\] 的一部分。

## 起源

早在比特币 \(Bitcoin\) 诞生不久的2011年，就有对 PoW \(Proof-of-Work\) 挖矿出块的讨论，持反对观点的人认为这是一种无意义、浪费资源的行为。有人提出 PoS 权益证明，以程序算法的形式出块并分配收益，而不是通过 CPU、GPU 等硬件计算出某种符合条件的值来出块。

在2016年9月的以太坊 DevCon 2 上，以太坊创始人 Vitalik 分享了他的最新研究成果《以太坊紫皮书》\[3\]。在紫皮书中，Vitalik 提出了以太坊 2.0 的一系列目标，其中就包括以太坊的 PoS 方案 Casper \(另一个主要功能是通过分片提升系统的可扩展性\)。

在2017年的 DevCon3 上，Vitalik 和以太坊基金会的其他小伙伴也有大量的议题是关于 Casper 的最新进展 \[4\]。

## 原理与目标

前面说到，PoW 机制需要消耗大量的电力去维持区块链系统出块，根据2017年底的数据统计 \[5\]，比特币挖矿目前占用全球 0.13% 的用电量，超过 159 个国家的年均用电量，并且在过去1个月内涨幅超过 29.98%。按照目前的增长速度 。2020年2月比特币挖矿将消耗全球所有的电量。

PoW 也是现在绝大部分区块链系统的基础机制。在 PoW 模式下的，用户消耗真实的电力用于产生区块获得分红。而在 PoS 模式下，用户使用区块链系统本身的虚拟代币进行挖矿，参与 PoS 的代币数量相当于 PoW 算力，达到了同样的出块效果却不用消耗真实的电力。（有点像股权分红）

以太坊的紫皮书中有如下几个 Casper 最终形态的设计目标：

1. \(Efficiency via proof of stake\) 通过 PoS 提升效率：共识机制不需要挖矿，降低电力浪费，并且可以满足大量可持续的以太币发行需求。
2. \(Fast block time\) 快速的出块率：在不影响安全的前提下，出块速度达到最大。
3. \(Finality\) 不可修改性：一旦出块并发布，一定时间之后这个块将会 “finalized” \(编程语言中的 不可修改 关键词\)。
4. \(Scalability\) 可扩展性：区块链不必在全部节点行运行，相对的，所有的节点只需要保持区块的一部分数据。这样既可以提高单个节点的处理能力，也能提高整体吞吐率。
5. \(Decentralization\) 去中心化：这个系统不需要依赖任何普通用户无法部署的“超级节点”。
6. \(Inter-shard communication\) 跨分片通信：应用在不同的分片上能够互相通信。特别的，一个应用能够跨越多个分片存在。
7. \(Recovery from Connectivity Catastrophe\) 从连接中断中恢复：这个系统需要能够在一半以上的节点突然掉线的情况下恢复并运行。
8. \(Censorship resistance\) 抗审查：这个协议能够防止大部分的见证人联合起来提供的虚假交易信息。

简单地说，Casper 设计为一个投注对赌模型。每个参与 PoS 的用户投入以太币进入资金池，每次下注猜测最终加入到主链的块，猜中即可获得分红，猜错会扣除一部分手续费。长期下来，每个用户的收益与其投入的代币数量成正比。

## 总统选举例子

为了便于读者理解，我们以美国总统大选为例。





## 参考

\[1\] Casper 项目地址：[https://github.com/ethereum/casper](https://github.com/ethereum/casper)  
\[2\] The Ethereum Killer is Ethereum 2.0: Vitalik Buterin's Roadmap: [https://bitcoinmagazine.com/articles/ethereum-killer-ethereum-20-vitalik-buterins-roadmap/](https://bitcoinmagazine.com/articles/ethereum-killer-ethereum-20-vitalik-buterins-roadmap/)  
\[3\] Ethereum.org 2.0 Mauve Paper: [https://docs.google.com/document/d/1maFT3cpHvwn29gLvtY4WcQiI6kRbN\_nbCf3JlgR3m\_8](https://docs.google.com/document/d/1maFT3cpHvwn29gLvtY4WcQiI6kRbN_nbCf3JlgR3m_8)  
\[4\] Youtube DevCon3: [https://www.youtube.com/watch?v=Yo9o5nDTAAQ](https://www.youtube.com/watch?v=Yo9o5nDTAAQ)  
\[5\] 比特币“挖矿”耗电惊人：[http://www.jfdaily.com/news/detail?id=72346](http://www.jfdaily.com/news/detail?id=72346)

