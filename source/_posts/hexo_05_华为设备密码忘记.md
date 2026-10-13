---
title: 华为设备 Console 密码忘记恢复笔记
date: 2026-10-13 10:40:00
tags: [华为, 密码恢复, Console, BootROM]
categories: 网络设备调试
---

# 华为设备 Console 密码忘记恢复

## 适用场景
- 忘记 Console 口登录密码，无法进入设备配置
- 设备物理可接触（机房本地操作）

## 恢复步骤

### 1. 重启设备，按 Ctrl+B 进入 BootROM
- 串口波特率 9600，重启后在出现 `Press Ctrl+B to enter BootROM menu` 时快速按键
- 默认 BootROM 密码通常为 `Admin@huawei.com`（不同版本可能不同）

### 2. 选择跳过配置启动
```BootROM
BootROM  Menu:
  1. Startup switch
  2. Password recovery
  ...
```
- 选择 `Password recovery` 或直接选跳过配置文件启动

### 3. 进入系统后重置密码
```shell
system-view
user-interface console 0
 authentication-mode password
 set authentication password cipher NewPass@123
 return
save
```

### 4. 注意事项
- 跳过配置启动后原配置仍在闪存中，需手动 `display saved-configuration` 确认
- 恢复完成后必须 `save` 保存新配置
- 此操作会中断业务，仅在业务维护窗口执行

## 思科对比
- 思科：Break 键进入 ROMMON，修改配置寄存器值为 0x2142 跳过启动配置
- 启动后 `copy startup-config running-config`，再改密码
- 最后把配置寄存器改回 0x2102 并保存
