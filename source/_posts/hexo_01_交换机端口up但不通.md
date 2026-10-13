---
title: 交换机端口Up但业务不通排查笔记
date: 2026-10-13 10:00:00
tags: [交换机, VLAN, STP, 端口排查]
categories: 网络设备调试
---

# 交换机端口 Up 但业务不通

## 现象
- `display interface` 显示端口物理状态为 Up、协议 Up
- 但该端口下终端 ping 网关丢包或完全不通

## 排查步骤

### 1. 确认 VLAN 与端口模式
```shell
display port vlan
display interface GigabitEthernet0/0/1
```
- 检查端口是否加入了正确的 VLAN
- Access 端口与终端 PVID 是否一致
- Trunk 端口是否放行了对应 VLAN

### 2. 检查 MAC 地址表
```shell
display mac-address vlan 10
```
- 若学不到终端 MAC：检查终端网卡、网线、端口 error 计数
- 若 MAC 在错误端口：可能存在环路或私接路由器

### 3. 检查 STP 状态
```shell
display stp interface GigabitEthernet0/0/1
```
- 端口被 STP 阻塞（DISCARDING）会出现 Up 但不通

### 4. 检查端口错包
```shell
display interface GigabitEthernet0/0/1 | include error|CRC|drop
```
- CRC 错误高：双工/速率不匹配或线缆质量问题
- 广播包暴涨：疑似环路

## 常见原因速查
| 现象 | 可能原因 |
|------|----------|
| Up/Up 但无 MAC | VLAN 配置错误、STP 阻塞 |
| 偶发丢包 | 双工不匹配、流量拥塞 |
| 端口频繁 Up/Down | 网线故障、对端设备电源不稳 |
