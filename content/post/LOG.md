---
title: "Hugo Stack 主题本地化与发布记录"
description: "记录 Stack v4 主题本地化、离线构建验证，以及通过 GitHub Actions 发布 Hugo 博客的流程。"
date: 2026-08-11T13:12:11+08:00
categories:
    - Hugo
tags:
    - Hugo
    - Stack
    - GitHub Pages
    - 主题定制
math: false
comments: true
draft: false
---

## 背景

本站正在从 Hexo 迁移到 Hugo。最初使用 Hugo Modules 从 GitHub 加载 Stack v4 主题，这种方式便于升级，但构建依赖上游仓库和 Go 模块缓存。

为了确保主题即使停止维护或上游仓库不可用，本站仍能继续构建，并且可以直接修改模板、样式和脚本，我将当前使用的主题完整保存在博客源码仓库中。

## 本次修改

### 1. 本地化 Stack 主题

将当前锁定的 Stack v4.0.0-beta.15 复制到：

```text
themes/stack/
```

本地主题包含 193 个文件，保留了原主题的 `LICENSE`、`README.md` 和 `theme.toml`。本地副本与原模块缓存逐文件 SHA-256 对比一致。

站点配置改为直接加载本地主题：

```toml
theme = "stack"
```

同时移除了远程主题导入和站点根目录中不再需要的 `go.mod`、`go.sum`。GitHub Actions、Hugo 版本和站点 URL 均未改变。

### 2. 保留 Hugo 的覆盖层

Hugo 会优先使用项目根目录中的同路径文件。因此，现有定制仍保留在根目录，未强制移动进主题：

```text
layouts/
assets/
i18n/
static/
```

当前会覆盖本地主题的主要文件包括：

```text
layouts/home.html
layouts/_partials/article/components/details.html
layouts/_partials/footer/custom.html
layouts/_partials/footer/footer.html
layouts/_partials/sidebar/left.html
assets/scss/custom.scss
i18n/en.toml
i18n/zh.toml
```

如果修改上述组件，应编辑根目录版本；其他基础主题组件可以直接编辑 `themes/stack/`。

### 3. 更新站点信息

- 中文和英文站名统一为 `LKWLhsy`。
- 中文和英文菜单中的 GitHub 链接指向个人主页。
- Hugo 构建继续使用当前项目站点地址；正式替换原 Hexo 根站点时，再将 `baseURL` 改为 `https://lkwlhsy.github.io/`。

## 验证结果

主题切换后，在以下条件下重新构建：

- Go 不在 `PATH` 中；
- 模块代理关闭；
- 使用全新的 Hugo 缓存；
- 不访问上游主题仓库。

构建结果：

```text
中文页面：16
英文页面：15
输出文件：46
迁移前后文件列表差异：0
迁移前后 CSS/JavaScript 哈希差异：0
构建退出状态：0
```

首页、英文首页、搜索、归档、关于和统计页面均成功生成。现有 Hugo 弃用警告仍然保留，但不影响本次构建。

## Hugo 博客发布指令

本站使用 GitHub Actions 构建并部署 GitHub Pages。Hugo 本身没有针对这种工作流的 `hugo d`；真正触发部署的命令是：

```bash
git push origin main
```

### 新建文章

创建普通 Markdown 文章：

```bash
hugo new content post/my-new-post.md
```

如果文章需要同目录图片，推荐使用页面包：

```bash
hugo new content post/my-new-post/index.md
```

编辑 frontmatter，将草稿状态改为：

```yaml
draft: false
```

### 本地预览

预览包括草稿在内的内容：

```bash
hugo server -D
```

浏览器访问：

```text
http://localhost:1313/
```

### 正式构建

```bash
hugo --gc --minify
```

生成结果位于 `public/`，该目录由 Git 忽略，不需要手动提交。

### 提交并发布一篇新文章

先检查工作区：

```bash
git status
```

只暂存本次文章，而不是无条件暂存所有文件：

```bash
git add content/post/my-new-post.md
```

如果使用页面包：

```bash
git add content/post/my-new-post/
```

提交并推送：

```bash
git commit -m "post: add my new post"
git push origin main
```

推送成功后，GitHub Actions 会自动执行 Hugo 构建并部署。可以在仓库的 **Actions** 页面查看 `build` 和 `deploy` 状态。

### 接近 `hexo d` 的连续命令

确认只需要提交指定文章时，可以使用：

```bash
hugo --gc --minify && \
git add content/post/my-new-post.md && \
git commit -m "post: add my new post" && \
git push origin main
```

不建议把 `git add -A` 固定写入发布命令，因为它可能把尚未完成的配置、文章或其他文件一起提交。

## 后续维护

主题源码现在由本站仓库保存。后续可以直接修改本地主题，但升级 Hugo 后仍应重新执行完整构建和页面检查。

Google Fonts、KaTeX、Waline 和表情资源目前仍可能来自外部服务；主题源码本地化不等于所有浏览器端资源已经离线化。
