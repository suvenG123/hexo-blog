---
title: 内网通但无法访问外网 NAT 故障排查笔记
date: 2026-10-13 11:00:00
tags: [NAT, 外网, 出口, ACL, 会话表]
categories: 网络设备调试
---

# 内网通但无法访问外网（NAT 故障）

## 现象
- 内网互 ping 正常
- ping 内网网关正常
- 但 ping 公网 IP（如 223.5.5.5）不通
- 或能 ping 通 IP 但打不开网页（DNS 问题）

## 排查步骤

### 1. 先确认是 NAT 问题还是路由问题
```shell
ping -a 内网源IP 公网IP
```
- 若不通：在出口路由器上 `ping -a 公网接口IP 公网IP`
- 出口自己能 ping 通 → 内网没走 NAT，或 NAT 规则没命中

### 2. 检查 NAT 配置（华为）
```shell
display nat outbound
display nat address-group
display acl 3000
```
- ACL 是否匹配了内网网段
- Outbound 接口是否正确应用
- 地址池是否用尽（`display nat session all`）

### 3. 检查 Easy IP / PAT
```shell
interface GigabitEthernet0/0/1
 nat outbound 3000
```
- Easy IP 方式直接复用出接口公网 IP

### 4. 检查会话表
```shell
display nat session destination 223.5.5.5
```
- 有会话但回包不到：检查回程路由
- 无会话：ACL 没匹配，或流量没走到出接口

### 5. DNS 问题
- 能 ping 通 IP 但域名解析失败：
```shell
display dhcp server lease   # 看下发的 DNS
nslookup www.baidu.com
```

## 思科对比
```ios
show ip nat translations
show ip nat statistics
show access-lists
```
