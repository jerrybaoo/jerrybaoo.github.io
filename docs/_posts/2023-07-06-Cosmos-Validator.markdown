---
layout: post
title:  "Cosmos SDK Validator Status"
date:   2023-07-06 21:14:18 +0800
categories: cosmos pos staking
---
# Cosmos Validator Status
Cosmos SDK 所代表的 POS 是比较典型的POS 系统。Validator 是POS 系统中的核心，因此讨论下，validator 的不同行为造成的影响。   
在下面的图表中，方框表示 Validator 可能处于的状态，线段表示状态变化的原因，箭头表示迁移方向。
![validator status](https://github.com/jerrybaoo/jerrybaoo.github.io/raw/main/docs/images/validator-status.png)

Validator 的状态由三个字段表示：

1. **unbonded/bonded/unbonding**：unbonded 表示 Validator 不参与投票，bonded 表示Validator参与投票，unbonding 表示 Validator 正在从 bonded 状态迁移到 unbonded 状态。
2. **jailed**：表示 Validator 被暂时监禁，在一定的时间窗口内不出块或者自抵押（创建 Validator 需要自己先抵押）小于阈值时，Validator 将被暂时监禁。要解除暂时监禁状态，需要满足监禁周期和自抵押数量两个条件，同时还需要主动发起 MsgUnjail 交易。监禁周期默认为 600 秒。
3. **tombstoned**：表示 Validator 被永久监禁，这是由于双签行为导致的一种非常严重的惩罚，而且无法解除。

默认情况下，Validator DownTime（离线时间过长）会被罚没 0.01% 的资金，而 DoubleSigning（双签）将会被罚没 5% 的资金。