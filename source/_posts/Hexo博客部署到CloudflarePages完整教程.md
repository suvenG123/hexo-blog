---
title: Hexo 博客部署到 Cloudflare Pages 完整教程
date: 2026-10-13 10:00:00
tags:
  - Hexo
  - Cloudflare
  - 教程
categories:
  - 教程
---

> **目标**：本地写 Markdown → 推到 GitHub → Cloudflare Pages 自动构建 → 用 readhaven567.top 访问

## 适用环境
- Windows 10 / 11
- Node.js v24.x
- Git
- GitHub 账号
- Cloudflare 免费账号
- GoDaddy 注册的域名 readhaven567.top

---

## 一、需要的软件和账号

| 软件 / 平台 | 作用 | 下载地址 |
|---|---|---|
| Node.js（LTS） | 运行 Hexo | https://nodejs.org |
| Git | 推送代码到 GitHub | https://git-scm.com |
| Hexo-CLI | 生成博客框架 | npm 安装 |
| GitHub | 存放博客源码 | https://github.com |
| Cloudflare | 托管静态站 + CDN + SSL | https://dash.cloudflare.com |
| GoDaddy | 域名注册商（只改 NS） | — |

> **注意**：不需要安装 `hexo-deployer-git`，也不需要 GitHub Pages。
> 因为我们走的是「Cloudflare Pages 拉 GitHub 源码自动构建」。

---

## 二、本地安装 Node.js 和 Git

### 1. 安装 Node.js
- 去 nodejs.org 下载 LTS 版本
- 一路下一步安装
- 安装完打开终端（Win+R → cmd）验证：