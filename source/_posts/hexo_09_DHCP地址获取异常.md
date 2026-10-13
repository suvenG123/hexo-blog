---
title: 终端无法获取 DHCP 地址排查笔记
date: 2026-10-13 11:20:00
tags: [DHCP, 地址池, 中继, Option43]
categories: 网络设备调试
---

# 终端无法获取 DHCP 地址

## 现象
- 客户端网卡显示"正在获取网络地址"或自动配置 169.254.x.x
- 拿到的 IP 不在预期网段

## 排查步骤

### 1. 确认 DHCP 服务器状态（华为）
```shell
display dhcp server ip-in-use
display ip pool
```
- 地址池是否耗尽
- 是否有静态绑定占满了地址池

### 2. 确认全局 DHCP 开启
```shell
display current-configuration | include dhcp enable
```
- 全局必须先 `dhcp enable`
- 接口下要选择接口地址池还是全局地址池：
  - `dhcp select interface`（接口地址池）
  - `dhcp select global`（全局地址池）

### 3. 跨 VLAN / 跨中继场景
若 DHCP 服务器在别的网段，终端所在网关必须配置中继：
```shell
interface Vlanif10
 dhcp select relay
 dhcp relay server-ip 192.168.1.100
```
- 检查中继地址是否正确
- 检查 DHCP 服务器是否放行了 DHCP 广播（UDP 67/68）

### 4. 排除地址冲突
```shell
display dhcp server conflict all
```
- 若存在冲突地址，服务器会暂时跳过该 IP

### 5. 抓包验证
- 在网关端口抓包看 DISCOVER 是否到达服务器
- OFFER 是否返回客户端
