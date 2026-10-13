---
title: IPSec VPN 隧道建立失败排查笔记
date: 2026-10-13 10:30:00
tags: [VPN, IPSec, IKE, 隧道]
categories: 网络设备调试
---

# IPSec VPN 隧道建立失败

## 现象
- 两端配置完整，但 `display ike sa` / `display ipsec sa` 无建立成功的 SA

## 排查步骤（以华为为例）

### 1. IKE 第一阶段（主模式）
```shell
display ike sa
```
- 无 SA：检查 IKE proposal 参数（加密、认证、DH 组）两端是否一致
- 对端地址、预共享密钥是否正确

### 2. UDP 500/4500 是否放通
- 边界 ACL / NAT 是否放过 IKE 流量
- 若中间有 NAT，确认开启 NAT-T（UDP 4500）

### 3. IKE 第二阶段
```shell
display ipsec sa brief
```
- 第一阶段 OK 但第二阶段失败：检查 IPSec 策略、ACL 匹配的感兴趣流
- 封装模式（传输/隧道）、PFS 设置两端必须一致

### 4. 触发流量
- 隧道不会自动建立，需要有感兴趣流命中 ACL 才会发起协商
- 可在两端用 `ping -a 源IP 对端私网IP` 触发

## 思科对比命令
```ios
show crypto isakmp sa
show crypto ipsec sa
debug crypto isakmp
debug crypto ipsec
```
