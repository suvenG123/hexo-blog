---
title: One IP部署与Cloudflare域名配置笔记
date: 2026-10-14 19:30:00
tags: [cloudflare]
categories: [部署]
---

# one-ip 部署与 Cloudflare 域名配置笔记

> 记录一次完整的部署过程：从 Fork 仓库、导入 Cloudflare Pages，到绑定自定义域名、理清 GoDaddy 与 Cloudflare 的分工。
> 日期：2026-10-14

---

## 目录

- [一、整体流程回顾](#一整体流程回顾)
- [二、在 Cloudflare 创建项目（不是点「创建应用程序」）](#二在-cloudflare-创建项目不是点创建应用程序)
- [三、为什么必须先 Fork](#三为什么必须先-fork)
- [四、配置页正确填法（踩过的坑）](#四配置页正确填法踩过的坑)
- [五、绑定自定义域名](#五绑定自定义域名)
- [六、GoDaddy 与 Cloudflare 的分工（核心概念）](#六godaddy-与-cloudflare-的分工核心概念)
- [七、改 NS 的完整操作步骤](#七改-ns-的完整操作步骤)
- [八、以后怎么更新](#八以后怎么更新)
- [九、常见问题排查表](#九常见问题排查表)

---

## 一、整体流程回顾

```
Fork 仓库（GitHub）
    ↓
导入 Cloudflare Pages（总览页 → 导入现有 Git 存储库）
    ↓
填写配置 + 构建变量 → 保存并部署
    ↓
验证 workers.dev / pages.dev 地址能打开
    ↓
绑定自定义域名 ip.readhaven567.top
    ↓
等待 SSL 证书签发 → Active
```

最终成果：

| 项目 | 地址 |
|---|---|
| Hexo 博客 | `https://readhaven567.top` |
| IP 查询工具 | `https://ip.readhaven567.top` |

两个项目在 Cloudflare 里是**独立的沙箱**，互不影响。

---

## 二、在 Cloudflare 创建项目（不是点「创建应用程序」）

**入口不在「创建应用程序」按钮。**

右上角蓝色的「创建应用程序」是「从零新建」，走的是选框架、配路由的向导，不适合导入现成仓库。

**正确入口有两个：**

**方式 A：从项目列表页**
1. 进入 `Workers 和 Pages` 总览页
2. 找到「导入现有 Git 存储库 / Import an existing Git repository」卡片
3. 点卡片右侧的蓝色 **「开始使用」**

**方式 B：从已有项目行**
1. 把鼠标移到某个项目行里的 `suven123/hexo-blog`（含 GitHub 图标）上
2. 停留半秒，浮出 **「导入仓库 / Import repository」**
3. 若不浮出，点该行最右侧的 `…` 菜单

> 注意：必须在**总览页**操作。「项目详情页」里看不到导入入口。

---

## 三、为什么必须先 Fork

项目文档原文：

> 上方按钮使用原项目地址；需要保留 Fork 关系和更新工作流时，请按教程导入你的 Fork。

| | 点「一键部署」橙色按钮 | 导入自己的 Fork |
|---|---|---|
| 部署能否成功 | 能 | 能 |
| 代码归属 | 原作者地址 | 自己账号下 |
| 以后改代码 | ❌ 无权限 | ✅ 可随意改 |
| Sync fork 同步更新 | ❌ 不可用 | ✅ 一键同步 |
| 适用场景 | 只想跑一次看看 | 长期使用、要更新 |

**Fork 操作步骤：**
1. 打开 `https://github.com/zhihui-hu/one-ip`
2. 点**右上角**的 `Fork`（star 按钮旁边，分叉图标）
3. 确认三项：
   - Owner = 你的账号 ✅
   - Repository name = `one-ip`（**不要改名**） ✅
   - Copy the DEFAULT branch only 保持勾选 ✅
4. 点 `Create fork`，等十几秒
5. 地址栏变成 `https://github.com/你的用户名/one-ip` 即成功

> **坑：不要改仓库名。** 改名后 `wrangler.toml` 里的 `name = "one-ip"` 对不上，构建会失败。

---

## 四、配置页正确填法（踩过的坑）

### 正确配置

| 项目 | 值 |
|---|---|
| 项目名称 | `one-ip`（必须与 `wrangler.toml` 里的 `name` 一致） |
| 生产分支 | `main` |
| 框架预设 | 无 / 保持默认 |
| 构建命令 | `pnpm build` |
| 构建输出目录 | `dist` |
| 根目录 | 留空 |

### 构建变量（必须加）

进入 **构建变量 / Build variables**，添加两条：

```
NODE_VERSION    24
PNPM_VERSION    10.32.1
```

**原因：** Cloudflare 默认 pnpm 为 10.11.1，项目要求 10.32.1。不加会在装依赖阶段报 `ERR_PNPM_*` 错误。

### 踩过的坑

**坑 1：项目名称多了连字符**
`one-ip` ≠ `one-ip`（后者多一个连字符）。Cloudflare 会拿项目名与 `wrangler.toml` 里的 `name` 比对，对不上直接报错。

**坑 2：`pnpm deploy` 填错位置**
`pnpm deploy` 是**本地部署**用的命令（读 `wrangler.toml` 的 `publish_dir = "dist"` 并上传）。Cloudflare Pages 导入流程里**没有「部署命令」这一栏**，只有「构建命令」和「构建输出目录」。硬塞进去会报「无效的构建输出目录」。

> `pnpm deploy` 的用法：本地执行，需先 `pnpm build`，把 `dist/` 上传到边缘节点。

### 部署过程日志

```
Installing dependencies   ← 最耗时，3–5 分钟，别以为卡住了
Running build command
Deploying to Cloudflare
✓ Deployment complete
```

成功后显示 **Active**，并给出地址 `https://one-ip-xxx.pages.dev`。

---

## 五、绑定自定义域名

### 前置条件

`readhaven567.top` 的 NS 早已指向 Cloudflare（证据：Hexo 博客显示「自动部署已启用」、域名 `readhaven567.top`）。**因此无需去 GoDaddy 做任何操作。**

### 步骤

1. 部署完成后点 **「继续处理项目」**
2. 左侧/顶部找 **「自定义域 / Custom domains」**
3. 点 **「添加自定义域」**，输入 `ip.readhaven567.top`
4. Cloudflare 自动完成三件事：
   - 在 DNS 里加一条 CNAME 指向 `one-ip-xxx.pages.dev`
   - 设为代理（橙色云）
   - 申请 HTTPS 证书
5. 等待 1–3 分钟，状态从 **Pending** → **Active**

> **不要**自己去 DNS 页面手动加 `ip` 的 A 或 CNAME 记录，否则会冲突报错。

### 状态说明

| 状态 | 含义 | 处理 |
|---|---|---|
| 正在验证 | 验证 DNS + 签发证书中，必经过程 | 等待，勿操作 |
| Pending 超 1 小时 | 偏慢但正常 | 继续等 |
| Pending 超 24 小时 | 可能出问题 | 去自定义域点「重试」 |
| Active | 成功 | 验证访问 |

### 顺手开启的免费功能

**① 强制 HTTPS**
`readhaven567.top` → SSL/TLS → Edge Certificates → 开启 **Always Use HTTPS**

**② 跳转 www 到主站（可选）**
Redirect Rules → 新增：匹配 `www.readhaven567.top/*`，301 重定向到 `https://readhaven567.top/$1`

### Pages 与 Worker 的入口区别

| 类型 | 自定义域入口 |
|---|---|
| Pages | 项目页 → 自定义域 |
| Worker | Worker → Settings → Domains & Routes |

---

## 六、GoDaddy 与 Cloudflare 的分工（核心概念）

**改 NS = 把「DNS 的回答权」交给 Cloudflare，不是交出服务器权限。**

### 域名的两层

| 层级 | 负责方 | 职责 |
|---|---|---|
| 域名注册 | GoDaddy | 记录域名归属、到期时间 |
| DNS 解析 | Cloudflare（改 NS 后） | 回答子域名指向哪里 |

### 改 NS 后，什么没交出去

| 事项 | 仍在谁手里 |
|---|---|
| 域名所有权 | GoDaddy |
| 续费 | GoDaddy |
| 改注册信息 | GoDaddy |
| **服务器权限（SSH/文件/数据库）** | **完全无关，一个都没动** |

Cloudflare 拿到的只有两样：
1. **DNS 解析权**
2. **流量代理权**（因记录设为橙色云；若设灰色云则只做解析）

### 访问链路

```
用户访问 ip.readhaven567.top
    ↓
GoDaddy：只负责域名注册 + NS 指向 Cloudflare（早已设好）
    ↓
Cloudflare：回答 CNAME → one-ip-xxx.pages.dev
    ↓
IP 查询工具
```

### 验证 NS 是否已指向 Cloudflare

```bash
dig NS readhaven567.top
```

返回中出现 `xxx.ns.cloudflare.com` 即确认。

### 在 GoDaddy 加记录为什么没用

NS 改走后，GoDaddy 的 DNS 面板成了「摆设」——里面的记录永远不会被读到。硬加反而有风险：万一将来把 NS 改回 GoDaddy，会与 Cloudflare 侧的记录冲突。

### GoDaddy 以后只剩三件事

1. 续费域名
2. 改注册信息
3. 换回其他 DNS（改 NS）

---

## 七、改 NS 的完整操作步骤

### 第一段：Cloudflare 添加域名（拿 NS 地址）

1. [dash.cloudflare.com](https://dash.cloudflare.com) → **Add a site**
2. 输入 `readhaven567.top`（不带 `www.` 和 `https://`）→ Continue
3. 选 Free → Continue to activation
4. 确认扫描出的 DNS 记录
5. **记下页面给的两个 Nameserver**（每个账户随机分配）：
   ```
   candy.ns.cloudflare.com     ← 示例，以自己页面为准
   mario.ns.cloudflare.com
   ```
6. **先别点「完成」，留着这个页面**

### 第二段：GoDaddy 改 NS（核心操作）

1. [account.godaddy.com](https://account.godaddy.com) → My Products → Domains → 点域名
2. 进入管理页 → 找 **DNS / Nameservers** 栏
3. 点 **Edit / 更改**（默认显示「Use GoDaddy nameservers / Default」）
4. 切到 **Custom / 自定义**
5. 删掉原地址，粘贴 Cloudflare 给的两个 NS
6. 点 **Save** → 确认框选 **Continue**（别点取消）

> **坑：**
> - 粘贴时**不要**在末尾加 `.`
> - 弹确认框一定要选继续

### 第三段：回到 Cloudflare 点「完成」

1. 切回未关闭的页面
2. 点 **Done, check nameservers**
3. 显示 **Active** 即成功
4. 若仍 Pending：点 **Check nameservers now**，或等待（5 分钟～24 小时，国内通常 10–30 分钟）

### 第四段：补回原有 DNS 记录（有网站/邮箱才需做）

**⚠️ 改 NS 会让所有解析失效约 5–30 分钟。** 操作前先截图保存旧记录。

NS 改走后，GoDaddy 里的 A/CNAME/MX **不会自动迁移**，需手动在 Cloudflare 补回：

`readhaven567.top` → DNS → Records → Add record：

| 类型 | 名称 | 内容 | 代理 |
|---|---|---|---|
| A | `@` | 原 IP | Proxied（橙云） |
| CNAME | `www` | 原目标 | Proxied |
| MX | `@` | 邮箱服务商给的地址 | DNS only（灰云） |

> **MX 记录最容易漏**，漏了邮箱立刻停摆。优先级数字须与原配置一致。

---

## 八、以后怎么更新

### 同步上游更新

1. 打开 `https://github.com/你的用户名/one-ip`
2. 页面提示「This branch is behind zhihui-hu:one-ip by X commits」
3. 点 **Sync fork → Update branch**
4. 合并完成后，Cloudflare 检测到 main 有新提交，**自动重新构建部署**

### 定时同步（可选）

使用仓库内的 **Sync upstream** 工作流，无需 PAT。通过 GitHub 的 `merge-upstream` 接口合并，遇冲突时停止并保留你的提交。

### 部署方式二选一

> **不要同时开启 Workers Builds 和 GitHub Actions 部署，会重复发布。**

- **Workers Builds**：main 收到提交时自动构建部署（当前使用）
- **GitHub Actions**：需在仓库 Actions 设置中添加部署凭证；推送到 main 或运行 `Build and deploy one-ip` 触发

### 可选功能：开启验证体验

1. Cloudflare → **Turnstile** → 添加站点 → 组件模式选「托管」
2. 拿到站点密钥和密钥
3. one-ip 项目 → 设置 → 变量和机密，加三条（类型选 Secret）：
   ```
   TURNSTILE_SITE_KEY     站点密钥
   TURNSTILE_SECRET       密钥
   TURNSTILE_HOSTNAMES    ip.readhaven567.top
   ```
4. 重新部署一次

---

## 九、常见问题排查表

| 现象 | 原因 | 处理 |
|---|---|---|
| 仓库列表找不到 one-ip | Fork 未成功 | 去 GitHub 确认 `你的用户名/one-ip` 是否存在 |
| 构建报 `ERR_PNPM_*` | 未加构建变量 | 补 `NODE_VERSION=24`、`PNPM_VERSION=10.32.1` |
| 构建报项目名不匹配 | 项目名称与 `wrangler.toml` 的 `name` 不一致 | 改成 `one-ip` |
| 「无效的构建输出目录」 | 把 `pnpm deploy` 塞进了输出目录框 | 清空，输出目录填 `dist` |
| 自定义域一直 Pending | 证书签发排队 | 等，超 24 小时点「重试」 |
| 访问报 522 / 523 | 证书签发卡住 | 自定义域页面点「重试」 |
| `dig` 返回 `198.18.x.x` | 本地代理开了 fake-IP 模式并缓存旧记录 | 关代理再试 |
| 邮箱收不到邮件 | MX 记录丢失 | 补回原 MX 记录 |
| `dig NS` 返回 GoDaddy 地址 | NS 未改成功 | 回 GoDaddy 确认保存是否生效 |
| GoDaddy 报「无效的名称服务器」 | 地址多了空格或点 | 重新粘贴，确保格式正确 |

---

## 附：项目说明摘要

- 项目：one-ip —— IP 查询、网络诊断、浏览器检测与 AI 服务状态工具箱
- 技术栈：Cloudflare Workers + Static Assets，`/api/*` 接口需要 Worker
- 基础功能**无需 API Key**
- 部署后可通过 `GET /api/ip/health` 查询 IP 健康度，无需 Key
- 信誉分 0–100，越高越好：75–100 good / 45–74 moderate / <45 poor
- 第三方服务有限流和跨域限制，部分结果可能空白或报 429，属正常现象
- 分享截图前请遮盖 IP、具体位置、运营商/ASN 及浏览器指纹标识
