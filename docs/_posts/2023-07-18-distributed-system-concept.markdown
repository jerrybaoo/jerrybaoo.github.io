---
layout: post
title:  "Distributed System Concept"
date:   2023-07-17 02:14:18 +0800
categories: distributed-system 
---
# 分布式系统概念

使用多台计算机解决单台计算机无法解决的**同一问题**的系统。使用分布式系统是由成本（资金和时间）决定的。如果使用单台超级计算机的成本低于使用分布式系统，那么分布式系统将不会得到关注。

### 目标

1. 可扩展性
    既然是分布式系统，那么必然存在多台计算协作。多体现在：数量，地理位置，管理人员。
2. 性能
    性能体现在两个方面：解决问题使用的资源和时间。
3. 可用性
   可用性是一个衡量标准。在分布式系统当中通过提高容错性达到高可用性。
显然单个组件/机器没有容错性，它要么是好的，要么是坏的。那么在分布式系统中，要通过良好实现达到高可用性。

### 阻碍

随着系统规模增大导致，  
节点的数量的增长：
- 增加通信需求，降低性能
- 增加失败可能性，降低可用性

节点的距离变长：
- 降低性能

讨论问题必须要抓住主要矛盾和次要矛盾。那么模型可以认为是主要矛盾的概括。比如当讨论错误模型时，那么通信方式就是次要矛盾，此时主要矛盾是:
1. 节点崩溃
2. 节点发送错误消息
3. 节点行为不一致

### 数据存储方式

分区和复制是指的是系统数据的存储方式。分区通常针对特定问题，而复制是更通用的解决方法。
分区将大数据集拆分成小数据集分布在不同节点上，通过允许分区独立失败提高可用性。
复制需要维护节点一致性，这是很多问题的根源。

## 系统模型

系统模型：设施分布式系统的的环境和设施一系列假设。假设越少系统越健壮。

通常会对以下方面做出假设：

1. 节点
2. 通信链路
3. 时序

### 节点模型

在分布式系统当中，对节点有以下特点

1. 节点执行程序
2. 节点能存储数据到易失性存储（内存），并且在恢复后能从稳定存储（磁盘）中读取数据
3. 节点拥有可靠或不可靠时钟。（分布式系统当中不应该具有全局可靠时钟？）

节点必须运行确定性算法（不存在无法验证的随机性）。

一般情况下，只考虑节点发生可恢复的崩溃错误，而不考虑拜占庭错误。

### 通信链路模型

节点之间通过通信链路传递信息。通常会对链路做出如下假设：

1. 消息在链路传递中是 FIFO 式的。（链路并不一定就是裸的网络环境，可能是在真是网络上面运行的组件。因此对于节点来说，链路可以确保消息的是顺序的）
2. 网络是可靠的，消息不会丢失。（不太可能，）

当通信链路发生故障，而节点没有发生故障时，那么就会导致**网络分区（network partition）。**

发生网络分区后，客户端仍然能访问节点。

## 时序（timing/ordering problem）

当讨论时序问题时，通常将系统分为，同步系统，异步系统。

同步系统需要使用适当的算法和协议来确保事件的顺序、消息的有序传递或操作的正确性。因此强一致性需求系统，通常需要同步系统。

异步系统中，节点的状态会不一致，因此他们追求的是最终一致性。

最简单的mysql 主从结构是异步系统，从节点之间可能会存在不一致，但是他们有最终一致性）。那么mysql 主从同步系统，使用 raft 这种强一致性系统，转发日志，那么显然这将是同步系统。

### 共识算法

共识算法特征：

1. 一致性（Agreement）：该属性要求所有正确的进程必须就同一个值达成一致。它确保系统中的所有进程都收敛到一个共同协定的值。
2. 完整性（Integrity）：完整性属性保证了正确的进程最多只能决定一个值。如果一个进程决定了一个值，那么该值必须是由系统中的某个进程提出的，而不是任意选择的。
3. 终止性（Termination）：终止性属性确保所有的进程最终都会做出决策。它防止一致性过程无限期地运行，并确保在有限的时间内做出决策。
4. 有效性（Validity）：有效性属性指出如果所有正确的进程都提议了相同的值V，那么所有正确的进程必须决定值V。它确保所有正确的进程最终会就所有正确提议的值达成一致和决定。

### FLP

异步系统中，不存在同时满足 Termination， Agreement， Validity 的算法。

在异步系统中，无法区分一个进程是在处理消息还是处于停滞状态（即没有收到任何消息）。这种不确定性导致无法保证在有限时间内达成共识。

### CAP

1. 一致性（Consistency）：系统中的所有节点同时看到相同的数据。换句话说，系统表现得就像是数据的单一一致副本。
2. 可用性（Availability）：系统在面对故障时能够继续操作并响应客户端请求。每个请求都会得到响应，尽管响应可能不包含最新的数据。
3. 分区容忍性（Partition Tolerance）：系统在面对网络分区或节点故障时仍能继续操作并保持一致性和可用性的保证。

理解CAP的方式是反证法。例如，如果系统发生网络分区，如果系统还是可用，那么分区内的写操作不一样，显然会导致分区内数据不一样，导致无法满足一致性。我们可以说这是AP系统无法满足一致性。实际上严格的数据证明也是使用了类似的思路。

那么实际上我们会得到三种分布式系统：

CA：典型代表是两阶段提交。

两阶段提交算法概述：

1. Prepare
- 协调者向所有的参与者发送准备请求。
- 参与者会根据自身状态检查自身是否可以执行事务。
- 如果参与者可以执行事务，那么它会通知协调者。
1. Commit
- 如果协调者收到所有参与者准备就绪的消息，那么它会通知参与者开始提交。
- 参与者提交完成后，回复提交完成给协调者。
- 协调者收到所有参与者提交完成后，通知参与者将结果持久化。

CP：raft 属于cp模型，当发生领导选举时，系统可能不可用，

PA：显然PA系统允许分区不一致，那么为了系在可用，可能最终要解决分区冲突，得到**最终一致性**。

### 一致性（Consistency）

1. 早期分布式系统中，通常是分布式数据库,一般设计成CA系统。但是随着时代发展，节点的地理分布决定了越来越需要考虑网络分区。
2. 如果系统不需要强一致性，即不要求在同一时刻观察所有节点，得到的数据一致，而是允许一定的延时达到最终一致性。那么可以设计出满足cap的系统。
3. 强一致性一般会损失性能，(想想二阶段提交)
4. 如果想系统在网络分区时保持可用性，那么可以考虑是否可以放弃强一致性。

### 一致性模型

程序和系统之间的约定，系统保证只要按预先的步骤操作，对数据的存储操作是可预测的。

强一致性模型：

1. 线性一致性。任何时间观测节点，得到的数据都是一致的
2. 顺序一致性。所有节点的数据操作顺序是一致的

弱一致性模型：

1. 客户端为中心的一致性。保证客户端永远看不到数据项的旧版本，例如当客户端切换节点时，如果发现节点中数据版本是旧的，那么使用本地缓存的新数据。分布式文件系统使用这个比较合适，本地客户端先缓存文件，同步到节点，可能是一个缓慢的动作。
2. 最终一致性。显然系统通过某种机制解决各个节点的冲突，例如基于时间戳，谁的时间戳新，就使用这个节点的数据。