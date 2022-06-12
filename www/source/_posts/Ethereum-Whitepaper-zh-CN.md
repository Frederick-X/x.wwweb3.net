---
title: Ethereum Whitepaper 中文版
date: 2022-06-03 01:22:41
update: 2018-02-02 23:45:00
banner: /Ethereum-Whitepaper-zh-CN/hero.webp
categories: [web3]
tags: [以太坊]
ident: true
mathjax: true
---



以太坊项目于2015年启动。这篇介绍性论文最初由以太坊的创始人Vitalik Buterin于2014年发布。与诸多社区驱动的开源项目相似，以太坊自发布以来持续迭代。

尽管已过去多年，但我们仍然保留这篇论文，因为它仍然是以太坊及其愿景的有用参考和准确表示。

<!-- more -->

以太坊白皮书原始链接：

{% link https://ethereum.org/zh/whitepaper Ethereum&nbsp;Whitepaper %}

[以PDF格式打开以太坊白皮书](https://ethereum.org/669c9e2e2027310b6b3cdce6e1c52962/Ethereum_Whitepaper_-_Buterin_2014.pdf)

## 新一代智能合约与去中心化应用平台

中本聪（Satoshi Nakamoto）在2009年开发的比特币被视为货币领域的重大发展，是首个既没有背书或“内在价值”（“[intrinsic value](https://bitcoinmagazine.com/business/op-ed-debunking-bitcoin-myths-the-intrinsic-value-fallacy)”）又没有集中发行人或控制人的数字资产的案例。而比特币的另一个更重要的部分，即承载其分布式共识的底层区块链技术，正引起人们的广泛关注。区块链技术的常见应用包括使用链上数字资产表示自定义货币和金融工具("[colored coins](https://docs.google.com/a/buterin.com/document/d/1AnkP_cVZTCMLIzw4DvsW6M8Q2JC0lIzrTLuoWu2z1BE/edit)")，底层物理设备的所有权("[smart property](https://en.bitcoin.it/wiki/Smart_Property)")，域名等不可替代资产 ("[Namecoin](http://namecoin.org/)")，以及更复杂的应用程序，包括由实现任意规则的代码直接控制数字资产("[smart contracts](http://www.fon.hum.uva.nl/rob/Courses/InformationInSpeech/CDROM/Literature/LOTwinterschool2006/szabo.best.vwh.net/idea.html)") 甚至是基于区块链的分布式自治组织（DAO, [decentralized autonomous organizations](http://bitcoinmagazine.com/7050/bootstrapping-a-decentralized-autonomous-corporation-part-i/) ）。以太坊的愿景是打造一个具有内置成熟的图灵完备编程语言的区块链。该语言可用于创建“契约”，用于对任意状态转换函数进行编码。以太坊能够使用户能够通过代码编写逻辑来创建上述提及的以及其它未曾想到的系统。

## 比特币及现有概念简介

### 历史

去中心化数字货币的概念以及诸如财产登记等应用已经存在了几十年。20世纪80年代和90年代的匿名电子现金协议主要基于于一种被称为盲签（Chaumian blinding）的加密原语，提供了具有高度隐私的货币。但因为这项技术依赖于一个集中的中介，导致该未能获得足够多的重视。1998年，Wei Dai的 [b-money](http://www.weidai.com/bmoney.txt)首次提出通过引入解决计算难题与分布式共识来创造货币，但并未给出分布式共识的实施细节。2005年，Hal Finney引入了“可重复使用的工作证明”（"[reusable proofs of work](https://nakamotoinstitute.org/finney/rpow/)"）的概念。它同时借鉴了b-money和Adam Back的难以计算的的Hashcash问题，创造了加密货币（cryptocurrency）的概念。但是由于它依赖于可信的计算作为后端，还是未能达到理想的效果。2009年，中本聪将通过公钥加密管理所有权的技术与跟踪谁拥有硬币的共识算法相结合，称为“工作证明”（“proof-of-work”），首次在实践中实施了分布式货币。

工作证明背后的机制是空前的突破，因为它同时解决了两个问题。其一，它提供了一个简单且适度有效的共识算法，允许网络中的节点为比特币账本的状态共同商定一组更新。其二，它给出了一个允许自由进入共识过程的机制，解决了决定谁可以影响共识的问题，同时能够避免sybil攻击。为此，它用经济壁垒取代了正式的参与壁垒，例如要求在特定名单上注册为唯一实体，共识投票过程中单个节点的权重与节点带来的计算能力成正比。从那时起，有人还提出了一种称为权益证明的替代方法，即计算节点的权重与其货币持有量成比例，而非计算资源。对这两种方法的优劣的讨论超出了本文的范围，但应该指出，这两种方法都可以用作加密货币的支柱。

### 比特币作为一个状态转移系统

![ethereum-state-transition](Ethereum-Whitepaper-zh-CN/ethereum-state-transition.png)

从技术角度来看，比特币等加密货币的账本可以被视为一个状态转移系统，包括所有现有比特币的所有状态组成的“状态”集合，和“状态转换函数”。状态转换函数接受一个状态和一个事务作为输入，并输出一个新的状态。

以经典的银行系统为例，“状态”就是一个资产负债表，转换是一个将$X$美元从$A$转移到$B$的请求，状态转移函数将$A$帐户中的价值减少$X$美元，将$B$帐户中的价值增加$X$美元。  若$A$中的余额少于$X$美元，状态转移函数将返回Error。以上过程可以形式化定义如下：

$$x^2$$

```
APPLY(S, TX) -> S' or ERROR
```

在上述定义的银行系统中：

```
APPLY({ Alice: $50, Bob: $50 },"send $20 from Alice to Bob") = { Alice: $30, Bob: $70 }
```

但：

```
APPLY({ Alice: $50, Bob: $50 },"send $70 from Alice to Bob") = ERROR
```





