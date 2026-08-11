---
title: " Hugo 弃用接口全面更新记录"
description: "记录 Hugo 0.164.0 弃用警告的定位、配置与模板 API 迁移、GitHub Actions 版本统一以及严格构建验证。"
date: 2026-08-11T22:06:40+08:00
categories:
    - LOG
tags:
    - Hugo
    - Stack
    - GitHub
    - 主题维护
math: false
comments: true
draft: false
---

## 更新背景
> 全部基于：gpt-5.6-sol 
>
> 毕竟 Hugo 稀奇古怪的代码我也改不明白(

本次更新针对 Hugo 0.164.0 Extended 构建时出现的弃用警告。更新前，站点仍然能够完成构建，但配置和本地 Stack 主题中使用了 Hugo 已经标记为弃用的语言、数据和多站点接口。这些接口将在未来版本中删除，因此继续忽略警告只会把问题推迟到后续升级。

本次处理原则是：不隐藏警告、不降低 Hugo 版本、不改变页面设计和内容结构，只把旧接口迁移到 Hugo 官方提供的新接口，并通过严格构建证明迁移完成。

Hugo 官方参考资料：

- [语言配置](https://gohugo.io/configuration/languages/)
- [Language 对象](https://gohugo.io/methods/site/language/)
- [hugo.Data](https://gohugo.io/functions/hugo/data/)
- [hugo.Sites](https://gohugo.io/functions/hugo/sites/)

---

## 一、实施前基线

实施前先执行常规正式构建，并单独收集所有 `deprecated` 警告。基线环境为：

```text
Hugo: 0.164.0 Extended
Language: ZH
Pages: 29
Static files: 1
Processed images: 7
Build exit code: 0
```

警告主要分成三组：

1. `languageCode`、`languageName` 和 `languageDirection` 配置键已经弃用。
2. 模板中的旧 Language、Data 和 Sites 接口已经弃用。
3. GitHub Actions 声明了 Hugo 0.154.2，但安装步骤使用 `latest`，导致声明的版本实际上没有生效。

实施前 Git 工作区还包含此前完成但尚未提交的主题本地化、页脚、文章卡片、隐私政策和文章底部信息框修改。本次更新没有回退或覆盖这些内容。

---

## 二、语言配置迁移

### 修改位置

```text
config/_default/config.toml
config/_default/languages.toml
```

### 配置键对应关系

```text
languageCode      -> locale
languageName      -> label
languageDirection -> direction
```

### 根配置

旧配置：

```toml
languageCode = "zh-cn"
```

新配置：

```toml
locale = "zh-CN"
```

### 多语言定义

更新后的配置为：

```toml
[zh]
    locale    = "zh-CN"
    label     = "中文"
    direction = "ltr"
    title     = "LKWLhsy"
    weight    = 1

[en]
    locale    = "en-US"
    label     = "English"
    direction = "ltr"
    title     = "LKWLhsy"
    weight    = 2
    disabled  = true
```

此次修改没有重新启用 English。`zh` 和 `en` 仍然是 Hugo 使用的语言键，`zh-CN` 和 `en-US` 是用于 HTML、RSS 和本地化处理的区域标识。

---

## 三、项目根目录模板迁移

根目录 `layouts/` 中的模板优先级高于 `themes/stack/layouts/`。因此，必须同时检查项目覆盖模板和本地主题模板，不能只修改主题副本。

### 1. 统计页站点集合

修改位置：

```text
layouts/_default/stats.html
```

旧写法：

```go-html-template
{{ range .Sites }}
```

新写法：

```go-html-template
{{ range hugo.Sites }}
```

统计页仍然遍历所有启用站点并汇总 `post` 分区中的文章。由于 English 处于禁用状态，当前实际只汇总中文站点。

### 2. 翻译页面名称

修改位置：

```text
layouts/_partials/article/components/details.html
layouts/_partials/sidebar/left.html
```

旧写法：

```go-html-template
{{ .Language.LanguageName }}
```

新写法：

```go-html-template
{{ .Language.Label }}
```

### 3. 语言键判断

旧写法：

```go-html-template
{{ .Language.Lang }}
```

新写法：

```go-html-template
{{ .Language.Name }}
```

这一替换用于语言切换器和中英文界面文本判断。虽然 English 当前已禁用，仍提前迁移这些代码，避免以后重新启用时再次出现旧接口。

### 4. 代码展开按钮

完整扫描还发现以下项目覆盖文件保留了 `.Site.Language.Lang`：

```text
layouts/_partials/footer/custom.html
```

两处判断均更新为：

```go-html-template
{{ if eq .Site.Language.Name `en` }}
```

这两处分别控制“展开代码”和“收起代码”的中英文文字。

---

## 四、本地 Stack 主题迁移

主题已经完整保存在 `themes/stack/`，因此本次直接更新本地主题中的旧接口，不依赖上游仓库修复。

### 1. HTML 语言属性

修改位置：

```text
themes/stack/layouts/baseof.html
```

旧写法：

```go-html-template
<html lang="{{ .Site.LanguageCode }}" dir="{{ default `ltr` .Language.LanguageDirection }}">
```

新写法：

```go-html-template
<html lang="{{ .Site.Language.Locale }}" dir="{{ default `ltr` .Language.Direction }}">
```

中文页面最终应生成：

```html
<html lang="zh-CN" dir="ltr">
```

### 2. RSS 语言标识

修改位置：

```text
themes/stack/layouts/rss.xml
```

旧写法：

```go-html-template
{{ site.Language.LanguageCode }}
```

新写法：

```go-html-template
{{ site.Language.Locale }}
```

生成的 RSS 应包含：

```xml
<language>zh-CN</language>
```

### 3. PhotoSwipe 数据

修改位置：

```text
themes/stack/layouts/_partials/article/components/photoswipe.html
```

旧写法：

```go-html-template
{{ $style := .Site.Data.external.PhotoSwipe.Style }}
{{ $core := .Site.Data.external.PhotoSwipe.Core }}
{{ $lightbox := .Site.Data.external.PhotoSwipe.Lightbox }}
```

新写法：

```go-html-template
{{ $style := hugo.Data.external.PhotoSwipe.Style }}
{{ $core := hugo.Data.external.PhotoSwipe.Core }}
{{ $lightbox := hugo.Data.external.PhotoSwipe.Lightbox }}
```

数据来源、资源地址和加载顺序均未改变，只有读取数据的 Hugo API 发生变化。

### 4. 通用外部资源数据

修改位置：

```text
themes/stack/layouts/_partials/helper/external.html
```

旧写法：

```go-html-template
{{ $List := index .Context.Site.Data.external .Namespace }}
```

新写法：

```go-html-template
{{ $List := index hugo.Data.external .Namespace }}
```

### 5. 主题后备模板

以下主题文件当前可能被根目录同路径文件覆盖，但仍一并完成迁移：

```text
themes/stack/layouts/_partials/article/components/details.html
themes/stack/layouts/_partials/sidebar/left.html
```

其中完成了以下替换：

```text
.Language.LanguageName -> .Language.Label
.Language.Lang         -> .Language.Name
```

这样即使以后删除根目录覆盖文件，主题后备模板也不会恢复使用旧接口。

---

## 五、统一 GitHub Actions 的 Hugo 版本

修改位置：

```text
.github/workflows/deploy.yml
```

实施前虽然定义了：

```yaml
HUGO_VERSION: 0.154.2
```

但安装步骤使用的是：

```yaml
hugo-version: 'latest'
```

因此版本变量没有被使用，而且每次部署可能获得不同 Hugo 版本。本地 Stack 主题声明的最低 Hugo 版本已经是 0.157.0，旧的 0.154.2 也不再适合作为版本基准。

更新后统一为：

```yaml
env:
  HUGO_VERSION: 0.164.0
```

```yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v3
  with:
    hugo-version: ${{ env.HUGO_VERSION }}
    extended: true
```

这样本地和 GitHub Actions 使用相同的 Hugo 0.164.0 Extended。后续升级时只需要修改 `HUGO_VERSION`，并先在本地执行严格构建验证。

本次只修改工作流文件，没有执行 Git 提交、推送或线上部署，因此不会立即触发 GitHub Actions。

---

## 六、LOG 文章重组

原文件：

```text
content/post/LOG.md
```

更名为：

```text
content/post/LOG-Hugo.md
```

为避免已经发布的旧地址 `/post/log/` 失效，在文章 Front Matter 中增加：

```yaml
aliases:
    - /post/log/
```

本次全面更新记录保存为：

```text
content/post/LOG-全面更新.md
```

两篇文章的分类均为：

```yaml
categories:
    - LOG
```

---


## 七、本次更新后的维护规则

以后升级 Hugo 时建议按以下顺序进行：

1. 先修改本地 Hugo 版本并执行普通构建。
2. 阅读全部警告，禁止直接隐藏警告。
3. 按 Hugo 官方迁移说明更新配置和模板。
4. 执行 `hugo --gc --minify --panicOnWarning`。
5. 检查 HTML、RSS、图片、统计页和移动端页面。
6. 本地验证全部通过后，再同步修改 GitHub Actions 的 `HUGO_VERSION`。
7. 最后才提交和推送。

不建议通过把 Hugo 降到 0.155 以下来回避警告，因为本地 Stack 主题最低版本已经是 0.157.0，而且降级无法解决未来兼容问题。


> 本次更新的目标不是让警告暂时消失，而是让配置、本地主题和自动部署共同使用同一套仍受 Hugo 支持的接口。
