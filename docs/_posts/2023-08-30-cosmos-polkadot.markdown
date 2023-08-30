---
layout: post
title:  "Cosmos 和 Polkadot 杂谈"
date:   2023-08-30 17:51:18 +0800
categories: distributed-system 
---

# Cosmos 和 Polkadot
以下是很早之前对 Cosmos 和波卡的认识，那时候写在知乎上。几年过去了 op/zk rollup 风头正劲。只看 op rollup，发展这么久，依然需要`辅助轮`，需要信任前提。那么在区块链的世界当中，如果某个存在把信任作为前提，那么是没有生命力的。

Cosmos 和波卡的最大区别就在与：Cosmos没有保证各个 Zone 的正确性，显然如果某 Zone 作恶，那么相关的 Zone 是检测不到的。  
Polkadot 会对平行链的状态转移做验证，而不仅是信任Pos投票。
分析下波卡的做法, 发现和 Op rollup 有相似的地方：
1. RelayChain 持有所有平行链的 wasm, state_root;
2. ParaChain 的区块必须要通过RelayChain验证。那么RelayChain 需要parachain 提供这些信息完成验证：
{
    ParaChainBlock {主要是 state_root, txs}
    ChangeStorages {交易访问到所有存储}
    ChangeStoragesPooof {交易访问到所有存储的merkle 证明}
}
3. RelayChain 验证：  
    a. 利用 ChangeStoragesPooof 和保存的 parachain state_root, 验证 parachain 提交的 ChangeStorages 是否正确。   
    b. 利用 parachain wasm 和 ChangeStorages 实例化一个wasm 虚拟机。  
    c. wasm 虚拟机执行交易。  
    d. 验证执行交易后的状态根是否和提交的状态根一致。  

个人认为波卡这套验证方式和Op Rollup 很相似，区别在于波卡每个块都验证，而OP是被动的等待挑战者来验证。波卡每个paraBlock都验证，带来了以下好处：
Parachain 可以丢掉共识，充分利用了波卡运行良好的POS共识系用。
每个Parachain都可以无条件互信。
丢掉了Relayer服务，节省了交易费。（但实际上波卡的卡槽也是一笔开支）。