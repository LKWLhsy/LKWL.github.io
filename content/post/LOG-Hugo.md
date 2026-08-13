---
title: "Hugo Stack 部署与主题个性化记录"
description: "记录 Hexo 到 Hugo 的迁移、Stack 主题本地化、GitHub Pages 部署和站点个性化配置。"
date: 2026-08-11T13:12:11+08:00
categories:
    - LOG
tags:
    - Hugo
    - GitHub Pages
    - Stack
    - 主题定制
    - 部署
math: false
comments: true
draft: false
---

## 记录范围

本站从 Hexo 逐步迁移到 Hugo。本文记录部署和主题个性化阶段的实际变更；Hugo 弃用接口迁移单独记录在 [LOG-全面更新](../log-全面更新/)，摄影与书与影栏目记录在 [LOG-栏目扩展](../log-栏目扩展/)，日常操作提取到 [LOG-使用手册](../log-使用手册/)。

本次整理遵循以下原则：

- 站点功能、页面 URL 和 GitHub Pages 工作流保持不变。
- 自定义优先放在项目根目录，用 Hugo 覆盖主题同路径文件。
- 代码记录采用“修改文件 + 关键片段 + 行为变化”，不复制未修改的完整主题文件。
- 本文标题不手工编号，目录编号由 Stack 主题负责。

## Hexo 到 Hugo 的迁移背景

Hugo 的源文件由模板、配置、内容、资源和数据组成，构建后再发布静态文件。本站保留仓库名 `LKWL.github.io`，当前属于项目站点，因此迁移阶段使用：

```toml
baseurl = "https://lkwlhsy.github.io/LKWL.github.io/"
```

将来如果正式把 Hugo 切换为账户根站点，才需要重新确认 Pages 仓库和 `baseurl`，不能在迁移阶段提前改成根路径。

## Stack 主题本地化

### 修改目的

最初通过 Hugo Modules 从上游加载 Stack。远程模块便于升级，但构建依赖上游仓库和 Go 模块缓存。为了让主题源码可审阅、可修改，并避免上游下线导致站点无法构建，将锁定的 Stack v4.0.0-beta.15 完整复制到：

```text
themes/stack/
```

本地副本保留主题的 `LICENSE`、`README.md`、`theme.toml`、模板、SCSS、TypeScript、i18n 和主题数据。站点配置改为：

```toml
theme = "stack"
```

同时移除了站点根目录不再需要的远程模块导入、`go.mod` 和 `go.sum`。主题自身目录中的 `go.mod` 是主题源码元数据的一部分，不等同于站点根目录的远程依赖。

### 修改后的维护方式

基础主题功能可以编辑：

```text
themes/stack/
```

本站专属覆盖优先编辑项目根目录：

```text
layouts/
assets/
i18n/
static/
```

根目录中存在同路径文件时，Hugo 会优先使用根目录版本。例如，本站的主要覆盖文件包括：

```text
layouts/home.html
layouts/list.html
layouts/_partials/article/components/details.html
layouts/_partials/article/components/footer.html
layouts/_partials/article/components/header.html
layouts/_partials/article/components/tags.html
layouts/_partials/footer/custom.html
layouts/_partials/footer/footer.html
layouts/_partials/helper/card-image.html
layouts/_partials/sidebar/left.html
assets/scss/custom.scss
i18n/zh.toml
i18n/en.toml
```

因此，修改前应先判断目标文件属于“本站覆盖层”还是“主题基础实现”。

## GitHub Pages 部署

### 工作流配置

部署文件是：

```text
.github/workflows/deploy.yml
```

Pages 来源保持 `GitHub Actions`。工作流执行以下关键步骤：

1. 检出仓库并安装 Go、Node.js、Dart Sass 和 Hugo Extended。
2. 使用 Hugo 0.164.0 构建站点。
3. 通过 `configure-pages` 注入 Pages 的部署地址。
4. 上传 `public/` 构建产物。
5. 由 `deploy-pages` 发布到 GitHub Pages。

当前版本环境变量为：

```yaml
HUGO_VERSION: 0.164.0
GO_VERSION: 1.25.5
NODE_VERSION: 24.12.0
DART_SASS_VERSION: 1.97.1
```

### Hugo 没有独立的 `hugo d`

Hugo 负责生成静态站点，GitHub Actions 负责部署。对本站而言，触发线上部署的动作是推送 `main`：

```bash
git push origin main
```

常用本地命令：

```bash
# 包括草稿的本地预览
hugo server -D

# 正式构建和兼容性检查
hugo --gc --minify --panicOnWarning
```

`public/` 是生成目录，不应手动编辑，也不应作为源码提交。

### 发布一篇文章

普通文章：

```bash
hugo new content post/my-new-post.md
```

需要同目录图片时使用 Page Bundle：

```bash
hugo new content post/my-new-post/index.md
```

完成内容并确认 `draft: false` 后，检查并提交明确范围：

```bash
git status
git add content/post/my-new-post/
git commit -m "post: add my new post"
git push origin main
```

不建议把 `git add -A` 固定写入快捷命令，因为它可能把尚未完成的配置或用户的其他改动一起提交。

## Footer 配置化

### 修改目的

Footer 原先把版权、运行时间、主题署名和自定义内容写在模板中。现在结构仍由模板控制，但文字、链接和显示开关全部进入配置。

全局开关位于：

```text
config/_default/params.toml
```

关键配置：

```toml
[footer]
    enabled         = true
    since           = 2026
    launchDate      = "2026-08-10"
    runtimeLink     = "stats/"
    showCopyright   = true
    showRuntime     = true
    showCustomText  = true
    showPoweredBy   = true
    showPrivacyLink = true
    privacyLink     = "privacy/"
```

中文文字位于：

```text
config/_default/params.zh.toml
```

```toml
[footer]
    copyrightText      = "© {year} {title}"
    runtimeIcon        = "🚀"
    runtimeLoadingText = "正在计算运行时间..."
    runtimeText        = "已运行 {days} 天 {hours} 时 {minutes} 分"
    customText         = " "
    poweredByText      = "主题 <b><a href=\"https://github.com/CaiJimmy/hugo-theme-stack\" target=\"_blank\" rel=\"noopener\">Stack</a></b>由 <a href=\"https://jimmycai.com\" target=\"_blank\" rel=\"noopener\">Jimmy</a> 设计"
    privacyText        = "隐私政策"
```

模板覆盖文件是：

```text
layouts/_partials/footer/footer.html
```

模板读取 `.Site.Params.footer`，用 `replace` 替换 `{year}`、`{since}` 和 `{title}`，用 `safeHTML` 保留配置中的受控链接和换行标记。运行时间由 `launchDate` 计算，显示文本由 `runtimeText` 控制。

要完全隐藏某一块，只需关闭对应开关。例如：

```toml
showRuntime = false
showCustomText = false
showPrivacyLink = false
```

要改变显示顺序，编辑 `layouts/_partials/footer/footer.html` 中各个 `<section>` 的顺序；不要只调整 TOML 字段顺序。

## 暂停 English

English 配置没有删除，只是暂时禁用：

```text
config/_default/languages.toml
```

```toml
[en]
    locale   = "en-US"
    label    = "English"
    disabled = true
```

恢复时删除 `disabled = true`，再检查英文内容、菜单和链接。英文源文件仍保存在 `content/`，因此禁用不会丢失内容。

## 默认卡片封面

默认图片位于：

```text
assets/img/default-cover.jpg
```

全局开关和路径位于：

```toml
[cardImage]
    enabled = true
    default = "img/default-cover.jpg"
```

项目覆盖模板：

```text
layouts/_partials/helper/card-image.html
```

模板先尝试文章、分类或页面自己的图片；没有找到时才读取 `cardImage.default`。因此默认封面用于列表卡片，不会强制插入文章正文顶部。

## 经典 Stack 首页与侧栏

首页模板：

```text
layouts/home.html
```

首页通过：

```go-html-template
{{ partial "sidebar/right.html" (dict "Context" . "Scope" "homepage") }}
```

保留搜索、归档、分类和标签云侧栏。首页网格开关位于：

```toml
[homepage]
    grid = false
```

`false` 表示经典单列列表；`true` 才启用主题的首页网格布局。

普通列表页的右侧栏由 `layouts/list.html` 控制。摄影栏目使用自己的模板，因此可以单独隐藏右侧栏，不影响首页和普通列表。

## 文章尾部个性化信息框

### 全局配置

文章底部配置位于：

```toml
[article.footerBox]
    enabled           = true
    showPersonalLabel = true
    icon              = "messages"
```

中文文字位于：

```toml
[article.footerBox]
    personalLabel = "感谢阅读"
```

项目覆盖模板：

```text
layouts/_partials/article/components/footer.html
layouts/_partials/article/components/tags.html
assets/scss/partials/custom-components/_article-footer.scss
```

模板把标签、许可协议和个人标签放入统一的信息区域；文章 front matter 可以单独关闭许可或评论，例如：

```yaml
license: false
comments: false
```

标签模板使用 `.GetTerms "tags"` 生成可点击链接，并通过 `assets/scss/partials/custom-components/_article-footer.scss` 设置边框、图标、间距和响应式布局。

## LOG 分类与隐私政策

LOG 分类入口：

```text
content/categories/LOG/_index.zh.md
```

LOG 博文使用：

```yaml
categories:
    - LOG
```

隐私政策页面：

```text
content/privacy/index.zh.md
```

Footer 链接由：

```toml
showPrivacyLink = true
privacyLink = "privacy/"
```

控制。相对链接会通过 `relLangURL` 自动适配项目站点路径；如果以后要跳转到外部隐私政策，可以将 `privacyLink` 改成完整 `https://...` URL。

## 当前工作区说明

本次日志整理时，工作区还存在未提交改动。能够从本次对话明确追溯的摄影 Page Bundle、摄影短代码和栏目扩展属于本轮已记录内容；关于页、参数文件、Tutorial 分类删除等其他改动在本次审计中不改变、不重新归因。提交前应先运行：

```bash
git status --short
git diff --stat
```

确认这些改动是否都属于本次提交范围。

## 维护检查

主题或 Hugo 版本变化后，建议依次执行：

1. 检查配置和根目录覆盖文件。
2. 执行 `hugo --gc --minify --panicOnWarning`。
3. 检查首页、文章、列表、Footer、图片、RSS 和移动端。
4. 检查 GitHub Actions 使用同一 Hugo Extended 版本。
5. 确认无误后再提交并推送。

弃用接口的具体迁移记录见 [LOG-全面更新](../log-全面更新/)，日常操作见 [LOG-使用手册](../log-使用手册/)。
