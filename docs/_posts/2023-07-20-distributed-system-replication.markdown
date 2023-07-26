---
layout: post
title:  "Distributed System Replication"
date:   2023-07-20 06:51:18 +0800
categories: distributed-system 
---
# 分布式系统中的复制技术
从整体看可以将分布式系统中的复制方法分为：同步复制，和异步复制。  
同步复制：当所有节点完成写请求时开始回复。它保证了所有节点的强一致性，即使发生数据丢失，也是全部节点都会丢失，但是性能堪忧。同步复制在节点发生故障时，无法再继续写，只能读。
异步复制：当前节点完成写请求时立刻回复。可以达到极致性能和高可用性，但是无法保证一致性，从不同节点读取的数据可能不一致。

## 强一致性复制技术
以下是从各个维度对比分布式复制算法很出名的一张图，很有参考价值。没有最好的算法，只有最适合使用场景的算法。
![lamport clock](../images/google-transact09.png)

### 主从复制（Master/Slaver）
一般主从复制，都是在 Master 上进行写操作，Master 将写日志分发给Slaver。
主从复制也有同步异步之分，具有上述同步异步方法的所有优缺点。  
Mysql 主从系统使用异步复制，从表现来看会出现读取数据不一致。当然这个系统是比较原始的，主节点持续发送bin log，不管从节点状态。   
假设使用前一节描述的简单同步复制方法，那么主从复制任然会有问题。  
主从结构在发生故障时，还可能遭受到脑裂。网络出现故障，可能会导致从节点提升为主节点，，也开始接受写请求。

### 两阶段提交
第一阶段：投票，协调者将更新发送给所有参与者，参与者处理更新，并决定接受更新或者拒绝。当决定接受时，参与者预写更新。  
第二阶段：决策，协调者决定结果，并通知所有参与者。如果所有参与者都接受更新，那么持久化预写后期。  
相对于主从同步，多了第二阶段决策，允许在系统的节点发生故障时回滚更新来保持一致性。
第二阶段非常依赖网络环境，如果发生网络分区，那么系统不可用。所以这是一种满足CA，不具备网络分区容忍性的算法。
传统的关系型数据库使用二阶段提交，通常他们的业务场景需要强一致性。

### 单副本一致性分区容错算法
单副本一致性分区容错算法，要求在发生网络分区时，只有一个副本活跃。这要求分区时，必须不能是对称的，这需要节点数为奇数。
大多数决策原则要求：当发生网络分区时，必须保证大多数节点分区可用，这就要保证 (N/2 + 1)个节点是正常的。

Paxos 阶段图:
```
[ Proposer ] -> Prepare(n)                                [ Followers ]
             <- Promise(n; previous proposal number
                and previous value if accepted a
                proposal in the past)

[ Proposer ] -> AcceptRequest(n, own value or the value   [ Followers ]
                associated with the highest proposal number
                reported by the followers)
                <- Accepted(n, value)
```
阶段1：准备（Prepare）

提议者（Proposer）向接受者（Followers）发送准备请求（Prepare(n)）。
准备请求包含一个提案号（proposal number）n，表示提议者想要引入的提案的编号。
接受者收到准备请求后，会检查自己是否已经接受过编号更大的提案。如果是，则回复一个Promise(n; previous proposal number and previous value if accepted a proposal in the past)消息。
如果接受者没有接受过编号更大的提案，它会承诺不再接受编号小于n的提案，并返回之前接受的最高提案编号和对应的值。  
阶段2：接受请求（AcceptRequest）  
如果提议者收到大多数接受者的Promise回复，并且没有收到拒绝回复，则可以发送一个接受请求（AcceptRequest(n, own value or the value associated with the highest proposal number reported by the followers)）。
接受请求包含提案号n和要提议的值（可能是自己的值，也可能是接受者返回的最高提案编号对应的值）。
接受者收到接受请求后，如果没有接受过编号更大的提案，则接受这个提案，并回复一个Accepted(n, value)消息。

### 弱一致性复制技术
强一致性分布式系统要求网络中副本必须保持一致，这导致这类系统往往需要多轮通信，在发生分区时停止写入。
实际上一些系统并不关注强一致性。它们更关注可用性、分区容忍性和性能。这些系统往往追求最终一致性。甚至某些系统依赖客户端解决副本冲突。这样的系统往往追求高性能，高可用，并能容忍网络分区。
达到最终一致性的核心问题是协调有冲突的副本。

