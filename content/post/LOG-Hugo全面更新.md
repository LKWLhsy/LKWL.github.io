---
title: "Hugo 弃用接口更新记录"
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
aliases:
    - /post/log-全面更新/
---

>本次 Hugo 的搭建与迁移所涉及的 **代码工作** 全部由 `gpt-5.6 Sol-medium` 进行
>
>毕竟我也看不懂 Hugo 的这些新编译语言 (
>
>不过全流程框架以及内容均按本人规划设计，可以放心阅读。

> WARNING: 内含大量 ai 生成内容，尽管已经经过大量人工审核修改，但本人编程水平太低难免遗留错误，请批判的阅读。

---
## 更新背景

本次更新针对 Hugo 0.164.0 Extended 构建时出现的弃用警告。更新前，站点仍然能够完成构建，但配置和本地 Stack 主题中使用了 Hugo 已经标记为弃用的语言、数据和多站点接口。这些接口将在未来版本中删除，因此继续忽略警告只会把问题推迟到后续升级。


Hugo 官方参考资料：

- [语言配置](https://gohugo.io/configuration/languages/)
- [Language 对象](https://gohugo.io/methods/site/language/)
- [hugo.Data](https://gohugo.io/functions/hugo/data/)
- [hugo.Sites](https://gohugo.io/functions/hugo/sites/)

---

## 实施前基线

迁移前使用 Hugo 0.164.0 Extended 完成普通构建，同时收集 `deprecated` 信息，得到以下历史快照：

**历史构建输出（不是配置文件）：**

```text
Hugo: 0.164.0 Extended
Language: ZH
Pages: 29
Static files: 1
Processed images: 7
Build exit code: 0
```

当时的问题分成三组：

1. `config/_default/` 仍使用 `languageCode`、`languageName` 和 `languageDirection`。
2. 项目覆盖模板和本地 Stack 主题仍通过旧的 Language、Data 和 Sites 接口读取数据。
3. `.github/workflows/deploy.yml` 声明 Hugo 0.154.2，但安装步骤使用 `latest`，导致声明变量未真正控制 CI 版本。

普通构建退出码为 0 只说明旧接口尚未被删除，并不代表这些接口可以继续长期使用。

---

## 语言配置迁移

### 配置键与职责

实际修改了两个配置文件：

| 文件 | 修改目的 |
| --- | --- |
| `config/_default/config.toml` | 设置站点默认区域标识 |
| `config/_default/languages.toml` | 定义各语言的区域、显示名称、文字方向和启用状态 |

实际对应关系为：

| 旧键 | 当前键 | 用途 |
| --- | --- | --- |
| `languageCode` | `locale` | BCP 47 区域标识，例如 `zh-CN` |
| `languageName` | `label` | 给访客显示的语言名称 |
| `languageDirection` | `direction` | HTML 文字方向，例如 `ltr` |

### 根配置

旧代码所在文件：`config/_default/config.toml`

```toml
languageCode = "zh-cn"
```

当前代码所在文件：`config/_default/config.toml`

```toml
locale = "zh-CN"
```

这里的 `locale` 是站点默认区域标识。它不是 Hugo 的内容语言键；内容语言键仍由 `defaultContentLanguage = "zh"` 和 `languages.toml` 中的 `[zh]` 决定。

### 多语言定义

旧代码所在文件：`config/_default/languages.toml`

```toml
[zh]
    languageCode      = "zh-cn"
    languageName      = "中文"
    languagedirection = "ltr"

[en]
    languageCode      = "en-us"
    languageName      = "English"
    languagedirection = "ltr"
    disabled          = true
```

当前代码所在文件：`config/_default/languages.toml`

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

此次迁移没有重新启用 English。`zh` 和 `en` 是 Hugo 的语言键，模板中的 `.Language.Name` 返回这类键；`中文` 和 `English` 是 `.Language.Label`；`zh-CN` 和 `en-US` 是 `.Language.Locale`。三个字段不能互换。

配置生效关系为：

```text
config/_default/config.toml
  → defaultContentLanguage = "zh"
  → config/_default/languages.toml 中的 [zh]
  → Language.Name / Label / Locale / Direction
  → HTML、RSS、语言切换器和翻译链接
```

---

## 项目根目录模板迁移

项目根目录 `layouts/` 中的同路径模板优先于 `themes/stack/layouts/`。所以兼容性扫描必须先确认实际生效的根覆盖文件，再检查本地主题后备文件。

### 统计页站点集合

旧代码所在文件：`layouts/_default/stats.html`

```go-html-template
{{- range .Sites -}}
{{- $allPosts = $allPosts | append (where .RegularPages "Section" "post") -}}
{{- end -}}
```

当前代码所在文件：`layouts/_default/stats.html`

```go-html-template
{{- range hugo.Sites -}}
{{- $allPosts = $allPosts | append (where .RegularPages "Section" "post") -}}
{{- end -}}
```

本站实际迁移的旧调用只有 `.Sites`。早期日志中补充出现的 `.Site.Sites` 和 `.Page.Sites` 是同类旧接口的概念说明，不是提交 `dc50f23` 中实际修改过的代码，不能把三者都写成本站变更。

迁移只改变站点集合的入口。后面的 `.RegularPages`、`Section == "post"`、追加顺序和统计方式均未修改。English 当前禁用，因此 `hugo.Sites` 实际只返回中文站点；以后重新启用 English 时，该统计页会继续汇总所有启用语言的 `post` 页面。

### 文章翻译名称

旧代码所在文件：`layouts/_partials/article/components/details.html`

```go-html-template
{{ range $Page.Translations }}
    <a href="{{ .RelPermalink }}" class="link">{{ .Language.LanguageName }}</a>
{{ end }}
```

当前代码所在文件：`layouts/_partials/article/components/details.html`

```go-html-template
{{ range $Page.Translations }}
    <a href="{{ .RelPermalink }}" class="link">{{ .Language.Label }}</a>
{{ end }}
```

这里只需要给读者显示“中文”或“English”，所以必须使用 `Label`，不能用返回 `zh`/`en` 的 `Name`，也不能用返回 `zh-CN`/`en-US` 的 `Locale`。

### 左侧栏语言键和显示名称

旧代码所在文件：`layouts/_partials/sidebar/left.html`

```go-html-template
{{- $currentLanguageCode := .Language.Lang -}}
{{ if ne .Language.Lang $currentLanguageCode }}
    <a href="{{ .RelPermalink }}">{{ .Language.LanguageName }}</a>
{{ end }}
```

当前代码所在文件：`layouts/_partials/sidebar/left.html`

```go-html-template
{{- $currentLanguageCode := .Language.Name -}}
{{ if ne .Language.Name $currentLanguageCode }}
    <a href="{{ .RelPermalink }}">{{ .Language.Label }}</a>
{{ end }}
```

同一文件中的移动端语言缩写、桌面语言链接和多语言下拉列表都遵守相同分工：比较语言时使用 `.Language.Name`，显示完整名称时使用 `.Language.Label`。虽然 English 已禁用，这些分支仍保留，以便以后重新启用时不恢复旧接口。

### 代码展开按钮

旧代码所在文件：`layouts/_partials/footer/custom.html`

```go-html-template
expandBtn.dataset.expand = "{{ if eq .Site.Language.Lang `en` }}Expand Code{{ else }}展开代码{{ end }}";
expandBtn.dataset.collapse = "{{ if eq .Site.Language.Lang `en` }}Collapse Code{{ else }}收起代码{{ end }}";
```

当前代码所在文件：`layouts/_partials/footer/custom.html`

```go-html-template
expandBtn.dataset.expand = "{{ if eq .Site.Language.Name `en` }}Expand Code{{ else }}展开代码{{ end }}";
expandBtn.dataset.collapse = "{{ if eq .Site.Language.Name `en` }}Collapse Code{{ else }}收起代码{{ end }}";
```

这两处只判断语言键，因此从 `Lang` 迁移到 `Name`。按钮的高度判断、展开、折叠和滚动逻辑均未改变。

### 根覆盖层的实际调用关系

当前实际生效关系如下：

```text
themes/stack/layouts/baseof.html
  → partial "sidebar/left.html"
  → layouts/_partials/sidebar/left.html（根目录覆盖）

themes/stack/layouts/_partials/article/article.html
  → partial "article/components/header"
  → layouts/_partials/article/components/header.html（根目录覆盖）
  → partial "article/components/details"
  → layouts/_partials/article/components/details.html（根目录覆盖）

themes/stack/layouts/_partials/footer/include.html
  → partial "footer/custom.html"
  → layouts/_partials/footer/custom.html（根目录覆盖）
```

因此只修改 `themes/stack/` 中的同名模板不会改变这三处当前页面行为；根覆盖文件才是运行时优先使用的版本。

### Language 字段边界

| 用途 | 旧写法 | 当前写法 |
| --- | --- | --- |
| HTML 或 RSS 区域标识 | `.Site.LanguageCode`、`.Language.LanguageCode` | `.Site.Language.Locale`、`.Language.Locale` |
| 语言键 `zh`/`en` | `.Language.Lang` | `.Language.Name` |
| 访客可见名称 | `.Language.LanguageName` | `.Language.Label` |
| 文字方向 | `.Language.LanguageDirection` | `.Language.Direction` |

迁移必须按语义逐项替换。把所有字段统一改成 `Locale` 会让界面显示 `zh-CN`；把 RSS 改成 `Name` 则只会输出 `zh`。

---

## 本地 Stack 主题迁移

主题源码已完整保存在 `themes/stack/`，所以本次直接维护本地主题，不等待或下载上游修复。主题层修改分为“当前直接生效的基础模板”和“被根目录覆盖、但仍需要保持可用的后备模板”。

### HTML 语言属性

旧代码所在文件：`themes/stack/layouts/baseof.html`

```go-html-template
<html lang="{{ .Site.LanguageCode }}" dir="{{ default `ltr` .Language.LanguageDirection }}">
```

当前代码所在文件：`themes/stack/layouts/baseof.html`

```go-html-template
<html lang="{{ .Site.Language.Locale }}" dir="{{ default `ltr` .Language.Direction }}">
```

该文件没有根目录同路径覆盖，因此它是当前所有 HTML 页面的基础模板。中文页面最终生成：

**生成结果来源：** `themes/stack/layouts/baseof.html`

```html
<html lang="zh-CN" dir="ltr">
```

### RSS 语言标识

旧代码所在文件：`themes/stack/layouts/rss.xml`

```go-html-template
<language>{{ site.Language.LanguageCode }}</language>
```

当前代码所在文件：`themes/stack/layouts/rss.xml`

```go-html-template
<language>{{ site.Language.Locale }}</language>
```

该文件同样没有根目录覆盖。中文 RSS 最终生成：

**生成结果来源：** `themes/stack/layouts/rss.xml`

```xml
<language>zh-CN</language>
```

### PhotoSwipe 外部资源数据

旧代码所在文件：`themes/stack/layouts/_partials/article/components/photoswipe.html`

```go-html-template
{{ $style := .Site.Data.external.PhotoSwipe.Style }}
{{ $core := .Site.Data.external.PhotoSwipe.Core }}
{{ $lightbox := .Site.Data.external.PhotoSwipe.Lightbox }}
```

当前代码所在文件：`themes/stack/layouts/_partials/article/components/photoswipe.html`

```go-html-template
{{ $style := hugo.Data.external.PhotoSwipe.Style }}
{{ $core := hugo.Data.external.PhotoSwipe.Core }}
{{ $lightbox := hugo.Data.external.PhotoSwipe.Lightbox }}
```

数据仍来自本地主题的 `themes/stack/data/external.toml`，字段名、资源 URL、动态导入和加载顺序均未改变。Hugo 会把主题数据合并到 `hugo.Data`；这里只把模板入口从 Site 对象迁移到 Hugo 全局命名空间。

### 通用外部资源 helper

旧代码所在文件：`themes/stack/layouts/_partials/helper/external.html`

```go-html-template
{{- $List := index .Context.Site.Data.external .Namespace -}}
```

当前代码所在文件：`themes/stack/layouts/_partials/helper/external.html`

```go-html-template
{{- $List := index hugo.Data.external .Namespace -}}
```

`.Namespace`、脚本或样式类型判断、SRI 和 `crossorigin` 输出均未修改。模板仍读取同一份 `external` 数据，只是不再经过 `.Context.Site.Data`。

### 主题后备模板与根覆盖

旧主题后备代码所在文件：`themes/stack/layouts/_partials/article/components/details.html`

```go-html-template
<a href="{{ .RelPermalink }}" class="link">{{ .Language.LanguageName }}</a>
```

当前主题后备代码所在文件：`themes/stack/layouts/_partials/article/components/details.html`

```go-html-template
<a href="{{ .RelPermalink }}" class="link">{{ .Language.Label }}</a>
```

当前优先生效文件：`layouts/_partials/article/components/details.html`

```go-html-template
<a href="{{ .RelPermalink }}" class="link">{{ .Language.Label }}</a>
```

旧主题后备代码所在文件：`themes/stack/layouts/_partials/sidebar/left.html`

```go-html-template
{{- $currentLanguageCode := .Language.Lang -}}
{{ if eq .Language.Lang $target.Language.Lang }}
<option value="{{ $target.RelPermalink }}" {{ if eq $target.Language.Lang $currentLanguageCode }}selected{{ end }}>{{ .Language.LanguageName }}</option>
```

当前主题后备代码所在文件：`themes/stack/layouts/_partials/sidebar/left.html`

```go-html-template
{{- $currentLanguageCode := .Language.Name -}}
<option value="{{ $target.RelPermalink }}" {{ if eq $target.Language.Name $currentLanguageCode }}selected{{ end }}>{{ .Language.Label }}</option>
```

当前优先生效文件：`layouts/_partials/sidebar/left.html`

```go-html-template
{{- $currentLanguageCode := .Language.Name -}}
<option value="{{ .RelPermalink }}" {{ if eq .Language.Name $currentLanguageCode }}selected{{ end }}>{{ .Language.Label }}</option>
```

这两个主题文件当前确定被根目录同路径文件覆盖，并非“可能被覆盖”。仍然迁移主题副本，是为了在将来删除根覆盖层时，后备模板不会重新暴露旧接口。

---

## 统一 GitHub Actions 的 Hugo 版本

### 旧版本变量没有控制安装步骤

旧代码所在文件：`.github/workflows/deploy.yml`

```yaml
env:
  HUGO_VERSION: 0.154.2
```

旧代码所在文件：`.github/workflows/deploy.yml`

```yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v3
  with:
    hugo-version: 'latest'
    extended: true
```

这两个片段彼此脱节：`HUGO_VERSION` 虽然存在，但安装步骤没有引用它。CI 会跟随 `latest` 变化，无法保证与本地复现环境一致。本地 Stack 的 `themes/stack/theme.toml` 又声明 `min_version = "0.157.0"`，所以原来未生效的 0.154.2 也不能作为当前主题基线。

### 当前版本锁定

当前代码所在文件：`.github/workflows/deploy.yml`

```yaml
env:
  HUGO_VERSION: 0.164.0
```

当前代码所在文件：`.github/workflows/deploy.yml`

```yaml
- name: Setup Hugo
  uses: peaceiris/actions-hugo@v3
  with:
    hugo-version: ${{ env.HUGO_VERSION }}
    extended: true
```

主题最低版本所在文件：`themes/stack/theme.toml`

```toml
min_version = "0.157.0"
```

当前 CI 和本地验证都以 Hugo 0.164.0 Extended 为基线。以后升级时只修改 `.github/workflows/deploy.yml` 的 `HUGO_VERSION`，但应先用目标版本完成本地兼容性审计。

---

## 当前源码审计与未完成项

### 已完成迁移的接口

当前重新扫描了以下范围：

```text
config/_default/
layouts/
themes/stack/layouts/
.github/workflows/deploy.yml
```

历史迁移涉及的 `languageCode`、`languageName`、`languageDirection`、`.Language.Lang`、`.Site.LanguageCode`、`.Language.LanguageCode`、`.Site.Data` 和旧 `.Sites` 调用已经不再出现在这些源码中。

### 当前仍存在的 `.IsNode`

当前未完成代码所在文件：`layouts/_partials/footer/custom.html: 70 line`

```go-html-template
{{ if and .Site.Params.comments.waline.serverURL (or .IsHome .IsNode (eq .Params.comments false)) }}
```

这个条件控制 Waline 浏览量脚本是否在首页、列表节点或主动关闭评论的页面加载。源码中只有这一处 `.IsNode` 调用，但模板会为多个页面执行，所以一次完整 INFO 构建会重复报告同一调用。

Hugo 0.164.0 当前给出的迁移提示是使用 `.Page.IsBranch` 或 `not .Page.IsPage`。二者语义并不完全相同：修改前必须分别验证 home、section、taxonomy、term、普通文章和自定义页面，确认哪些页面应该批量加载 Waline 浏览量脚本。这里只记录问题，不在日志整理任务中修改模板。

### 严格构建与 INFO 审计的区别

严格构建命令，执行目录：`D:/Hugo/LKWLblog`

```bash
hugo --gc --minify --panicOnWarning --destination "<temporary-output-dir>"
```

INFO 审计命令，执行目录：`D:/Hugo/LKWLblog`

```bash
hugo --gc --minify --logLevel info --destination "<temporary-output-dir>"
```

`--panicOnWarning` 只会把 warning 提升为失败；`.IsNode` 的弃用信息当前属于 INFO。因而“严格构建退出码为 0”与“没有弃用接口”不是同一结论。

本次日志审计前的当前构建结果为：

**当前构建输出（不是配置文件）：**

```text
Hugo:             0.164.0 Extended
Pages:            81
Static files:     151
Processed images: 50
Aliases:          33
Build exit code:  0
```

同一次 INFO 构建在源码只有一个 `.IsNode` 调用点的情况下输出了 51 条 `.Page.IsNode` 弃用信息。这是该 partial 被多次渲染的结果，不代表源码中存在 51 处不同问题。

只有严格构建通过、INFO 日志不再包含待处理弃用接口，并且页面行为验证通过，才能表述为完成一次全面兼容性迁移。

---

## 后续升级与验证流程

以后升级 Hugo 时按以下顺序处理：

1. 在 `D:/Hugo/LKWLblog` 确认工作区已有修改，避免覆盖用户内容。
2. 使用目标 Hugo 版本执行普通构建和 `--logLevel info` 构建。
3. 按警告逐项定位 `config/_default/`、根 `layouts/` 和 `themes/stack/layouts/`。
4. 先修改当前真正生效的根覆盖文件，再同步仍需保留的主题后备文件。
5. 执行 `hugo --gc --minify --panicOnWarning`，不得通过隐藏警告或降低版本制造通过。
6. 对比 Hugo 版本、Pages、Static files、Processed images、Aliases、弃用信息数量和退出码。
7. 检查生成 HTML 的 `lang`/`dir`、RSS `<language>`、统计页、语言界面、PhotoSwipe 和外部资源。
8. 使用桌面和窄屏浏览器确认页面无资源错误或布局回归。
9. 本地验证完成后，再同步 `.github/workflows/deploy.yml` 的 `HUGO_VERSION`。
10. Git 提交、推送和部署仍作为独立外部操作确认。

页面或图片数量会随内容增长而变化，不能直接拿历史的 29 页与当前 81 页判断回归。有效比较必须使用同一内容基线；真正需要调查的是同一基线下页面突然减少、alias 消失、资源处理失败、输出语言错误或退出码改变。

> 兼容性维护的目标不是暂时让黄色警告消失，而是让当前配置、实际生效模板、本地主题后备和 CI 使用同一套可复现的 Hugo 接口。
