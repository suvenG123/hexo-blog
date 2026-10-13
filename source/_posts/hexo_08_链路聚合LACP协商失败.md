---
title: 链路聚合 LACP 协商失败排查笔记
date: 2026-10-13 11:10:00
tags: [链路聚合, Eth-Trunk, LACP, Port-channel]
categories: 网络设备调试
---

# 链路聚合（Eth-Trunk / Port-channel）协商失败

## 现象
- 多根网线互联，但只有一根在跑流量
- 或聚合接口显示 Down

## 排查步骤（华为 Eth-Trunk）

### 1. 查看聚合组状态
```shell
display eth-trunk 1
```
- 查看成员口数量、选中（Selected）数量
- 若成员口 Unselected：查看原因字段

### 2. 模式必须一致
- 两端要么都手工负载分担（manual），要么都 LACP 动态模式
- 一端 manual、一端 LACP 会导致只有一个口 Up

```shell
interface Eth-Trunk1
 mode lacp-static
```

### 3. 成员口配置必须一致
所有加入 Eth-Trunk 的物理口必须：
- 相同的 VLAN 配置（PVID、Trunk 放行）
- 相同的速率、双工模式
- 相同的端口类型（都 Access 或都 Trunk）
- 不能有其他业务配置（如端口隔离）

### 4. LACP 系统优先级与接口优先级
```shell
display lacp eth-trunk
```
- 若一端优先级更高，会主动决定选中端口
- 可通过 `lacp priority` 调整

## 思科 Port-channel
```ios
show etherchannel summary
show lacp neighbor
```
- 同样要求两端模式一致：`on / desirable / auto / active / passive`
- 推荐两端都是 `active`（LACP）
