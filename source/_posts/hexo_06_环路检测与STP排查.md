---
title: 二层环路检测与 STP 排查笔记
date: 2026-10-13 10:50:00
tags: [环路, STP, 广播风暴, BPDU]
categories: 网络设备调试
---

# 二层环路检测与 STP 排查

## 现象
- 全网突然变慢，广播风暴
- 交换机 CPU 占用率飙高
- 端口指示灯疯狂闪烁

## 快速判断
```shell
display interface | include rate
display cpu-usage
```
- 广播/组播包速率接近线速：基本可判定环路

## 排查步骤

### 1. 查看 STP 状态
```shell
display stp brief
display stp topology-change
```
- 根桥是否正确
- 端口角色是否符合预期（ROOT / DESI / ALTE）

### 2. 定位环路端口
```shell
display stp interface GigabitEthernet0/0/1
```
- 若端口同时被 STP 阻塞但仍有大量流量，说明 STP 未生效（可能被关闭）

### 3. 检查是否有端口未参与 STP
```shell
display stp | include Protocol
```
- 若 `Protocol*: stp` 为 disabled，说明该端口被配置为 `stp edged-port` 且连接了另一台交换机

### 4. 紧急处置
```shell
shutdown GigabitEthernet0/0/X
```
- 先拔/关怀疑链路，恢复业务
- 再慢慢定位私接交换机、网线误接

## 预防
- 所有接终端端口配置 `stp edged-port enable`
- 开启 BPDU 保护：`stp bpdu-protection`
- 开启环路检测：`loopback-detection enable`
