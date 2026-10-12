# Hexo 博客部署到 Cloudflare Pages 完整教程（小白版）

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
  ```
  node -v
  npm -v
  ```
- 能看到版本号就算成功

### 2. 安装 Git
- 去 git-scm.com 下载
- 安装时默认选项即可
- 验证：
  ```
  git --version
  ```

### 3. 配置 Git 用户名
```
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

---

## 三、安装 Hexo 并初始化博客

### 1. 全局安装 Hexo-CLI
```
npm install -g hexo-cli
```
验证：
```
hexo -v
```

### 2. 新建博客目录
```
D:
mkdir hexo-blog
cd hexo-blog
```

### 3. 初始化 Hexo
```
hexo init
npm install
```

### 4. 本地预览
```
hexo clean
hexo generate
hexo server
```
浏览器打开 **http://localhost:4000**

---

## 四、写文章基础命令
```
hexo new "文章标题"
```
编辑 `source/_posts/文章标题.md`

---

## 五、创建 GitHub 仓库
1. 登录 GitHub → **New repository**
2. 仓库名：`hexo-blog`
3. 选 **Public**
4. **不要勾** Add README
5. **Create repository**

---

## 六、把本地博客推到 GitHub
```
cd D:\hexo-blog
git init
git add .
git commit -m "初始化Hexo博客"
git branch -M main
git remote add origin https://github.com/suvenG123/hexo-blog.git
git push -u origin main
```

---

## 七、Cloudflare Pages 部署
1. 登录 dash.cloudflare.com
2. **Workers 和 Pages** → **创建应用程序** → 选 **Pages**
3. **连接到 Git** → 选 `hexo-blog` 仓库
4. 构建设置：
   - **项目名称**：`hexo-blog`
   - **生产分支**：`main`
   - **框架预设**：无
   - **构建命令**：`npx hexo generate`
   - **输出目录**：`public`
   - **根目录**：留空
   - **环境变量**：`NODE_VERSION = 24`
5. 保存并部署 → 等变绿

---

## 八、GoDaddy 改 NS
1. Cloudflare 添加站点 `readhaven567.top` → 拿两个 NS 地址
2. GoDaddy → 域名 → DNS → Nameservers → Change
3. 删掉原来的 `ns63/ns64.domaincontrol.com`
4. 填 Cloudflare 给的两个 NS → Save
5. 关闭 DNSSEC
6. 等状态变 **Active**

---

## 九、绑定自定义域名
Pages 项目 → **自定义域** → **设置域** → `readhaven567.top`

等 Active → 访问 **https://readhaven567.top**

---

## 十、修改 Hexo url 配置
`_config.yml` 里：
```yaml
url: https://readhaven567.top
root: /
```
然后：
```
hexo clean && hexo generate
git add . && git commit -m "绑定正式域名" && git push
```

---

## 十一、日常流程
```
hexo new "标题"
hexo clean && hexo generate
git add . && git commit -m "更新" && git push
```

---

## 十二、常见报错

| 问题 | 解决 |
|---|---|
| 部署红色失败 | 看日志，检查构建命令和环境变量 |
| .pages.dev 能开域名打不开 | NS 没生效或自定义域没绑 |
| ERR_CONNECTION_CLOSED | 域名绑到了 Worker，解绑 |
| 重定向过多 | SSL 模式改 Full |
| 手机能电脑不能 | `ipconfig /flushdns` |
