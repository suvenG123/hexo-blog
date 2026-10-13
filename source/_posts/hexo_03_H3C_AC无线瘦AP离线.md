---
title: H3C AC 旁挂瘦 AP 离线排查笔记
date: 2026-10-13 10:20:00
tags: [H3C, 无线, AC, AP, CAPWAP]
categories: 网络设备调试
---

# H3C AC 旁挂瘦 AP 离线

## 现象
- AC 上 `display wlan ap all` 显示 AP 状态为 Idle/Fault
- AP 上线失败，SSID 不广播

## 排查步骤

### 1. 确认 AP 与 AC 二层/三层连通
```shell
display dhcp server ip-in-use
ping <AC管理IP>
```
- AP 需从 DHCP 获取到包含 Option 43 的地址
- 三层组网时检查 Option 43 是否指向 AC

### 2. 检查 AP 模板与注册状态
```shell
display wlan ap all
display wlan ap name <ap-name>
```
- State: Run 正常；Idle 表示未注册；Fault 表示版本不匹配或认证失败

### 3. 检查 CAPWAP 链路
```shell
display capwap ap all
display capwap statistics
```
- UDP 5247/5246 端口是否被防火墙拦截
- DTLS 隧道建立是否成功

### 4. 版本与授权
- AP 与 AC 软件版本不匹配会导致下载镜像失败
- AC 的 AP License 数量是否已满

## 快速恢复
```shell
reset wlan ap name <ap-name>
```
让 AP 重新向 AC 发起注册。
