---
title: 网络设备通用排障思路与命令速查模板
date: 2026-10-13 11:30:00
tags: [排障思路, 命令速查, 方法论, 值班手册]
categories: 网络设备调试
---

# 网络设备通用排障思路模板

> 遇到问题时按此顺序排查，避免东一榔头西一棒子。

## 一、分层排障模型（OSI 自下而上）

| 层级 | 检查项 | 常用命令 |
|------|--------|----------|
| 物理层 | 线缆、光模块、端口灯 | `display interface` |
| 数据链路层 | VLAN、MAC、STP、聚合 | `display mac-address`、`display stp brief` |
| 网络层 | IP、路由、ARP | `display ip routing-table`、`display arp` |
| 传输层 | TCP/UDP、端口、ACL | `display acl all`、`display firewall session` |
| 应用层 | DNS、HTTP、业务 | `nslookup`、`curl`、`traceroute` |

## 二、排障四问
1. **什么业务不通？**（源 IP、目的 IP、端口）
2. **什么时候开始的？**（最近做过什么变更）
3. **是单点还是全网？**（一台终端 / 一个部门 / 全部）
4. **是偶发还是必现？**（丢包率、时间段）

## 三、常用诊断命令速查

### 华为 / H3C
```shell
display interface brief
display ip routing-table
display arp
display logbuffer
display cpu-usage
display memory
ping -c 100 -s 1500 <目标>
tracert <目标>
```

### 思科
```ios
show ip interface brief
show ip route
show arp
show logging
show processes cpu
show memory statistics
ping <目标> repeat 100 size 1500
traceroute <目标>
```

## 四、抓包要点
- 先在离故障点最近的接口抓
- 关注握手包（SYN / SYN-ACK / ACK）
- 关注 ICMP 超时、ICMP 不可达
- 关注 RST 包（端口被拒绝）

## 五、变更前必做
- `display current-configuration` 备份
- 记录变更前的端口、路由、会话状态
- 预留回退方案（`undo` 命令写好）

## 六、记录模板
```
【故障现象】
【影响范围】
【排查过程】
【根因】
【解决方案】
【预防措施】
【责任人 / 时间】
```
