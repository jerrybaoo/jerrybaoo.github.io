---
layout: post
title:  "POS 区块链 Staking 奖励分配"
date:   2023-07-17 10:14:18 +0800
categories: blockchain 
---
# POS区块链奖励分配
pos staking 的奖励来自每个块的通胀，所以每个块 active validators 都可以根据自己的 vote power 获得奖励。由于 validator 的佣金率是随时可以改变的，所以必须在validator奖励的时候，根据当前的佣金率计算validator和 delegator 分别获得的奖励，并记录。假设当前块某个validator 获得 10token 奖励，它的佣金率是 20%， 那么 8token 所有delegator 共同参与分配，而 2token 归validator 独享。

## Distribute Reward
每个validator 维护自己的周期。这里的周期并不是固定时间长度，而是每次validator 上的delegations 变化都会创建一个新周期。
一个周期包含三个信息，startBlockHeight, endBlockHeight, rewardRatio.
同时每个delegation 都会包含startHeight, endBlockHeight,startDelegateAmount, 用于记录delegation 对应的 validator 周期。每次delegation 变化都会结算奖励，并重新初始化delegation。

简单描述下奖励分配过程：  
新创建的周期 reward ratio 为0。  
此周期奖励ratio为上一周期奖励ratio，叠加此周期奖励ratio. 第0周期有 10token, 那么当另外一个staker delegate 10token 时，此前一共获得了 5token奖励，这时候要创建一个新周期，并确定第0奖励ratio 为 5/10 = 0.5。那么当staker 再delegate时，第一周期获得了 8token 奖励，那么那么第一周期的ratio
为 8/（10 + 10）+ 0.5 = 0.9。  
如果第0 周期的人，此时取出奖励，那么 reward = （0.9-0） * 10 = 9  
如果第1 周期的人，此时取出奖励，那么 reward = （0.9-0.5） *10 = 4

## Slash
假设从区块链启动开始，即存在一个Validator 和一些 delegations存在与其上。
如果delegator1 和 delegator2 分别在第10和第15个块进行质押. 显然这里会形成两个周期。  
1. period0 {[0, 10)}. 从validator 启动到 delegator1 进行质押。
2. period1 {[10, 15)}, delegator1 质押后到 delegator2 开始质押。
那么如果在period1 时间段中，validator被slash, 那么validator 总的质押量被消减。此时具体的delegation 并不更改（更改可能很耗时）。
这样在 period1 之后的奖励分配当中，必须要根据delegator1 实际的质押量进行分配。那么在之后的period2中， delegator1的奖励为 period2RewardRatio * startDelegationAmount * (1-slashFactor). 
