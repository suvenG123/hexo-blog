---
title: 思科路由器路由表缺失排查笔记
date: 2026-10-13 10:10:00
tags: [思科, 路由表, OSPF, EIGRP, 静态路由]
categories: 网络设备调试
---

# 思科路由器路由表缺失

## 现象
- 配置了静态路由或运行动态路由协议，但 `show ip route` 看不到目标网段

## 排查步骤

### 1. 确认出接口状态
```ios
show ip interface brief
```
- 出接口必须是 Up/Up 才会在路由表中生效

### 2. 静态路由检查
```ios
show ip route static
show running-config | section ip route
```
- 静态路由下一跳是否可达
- 若指定了出接口（点对点），需确保该接口 Up

### 3. OSPF 邻居状态
```ios
show ip ospf neighbor
show ip ospf interface brief
```
- 邻居必须达到 Full 状态才会交换 LSA
- 常见卡在 INIT：单通、区域 ID 不匹配、Hello/Dead 间隔不一致

### 4. EIGRP 邻居状态
```ios
show ip eigrp neighbors
show ip eigrp topology
```
- K 值不匹配、AS 号不一致、被动接口都会导致邻居建立失败

### 5. 检查分发列表 / 路由策略
```ios
show ip protocols
```
- 是否有 distribute-list 过滤了对应网段

## 验证
```ios
ping x.x.x.x source y.y.y.y
traceroute x.x.x.x
```
