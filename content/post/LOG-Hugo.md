---
title: "Hugo 部署与主题个性化记录"
description: "记录 Hexo 到 Hugo 的迁移、Stack 主题本地化、GitHub Pages 部署和站点个性化配置。"
date: 2026-08-11T13:12:11+08:00
categories:
    - LOG
tags:
    - Hugo
    - GitHub
    - Stack
    - 主题定制
math: false
comments: true
draft: false
---

由于计划从 Hexo 迁移到 Hugo ，因此先把 Hugo+Stack 框架建好。本文记录了 Hugo+stack 部署和主题个性化阶段的过程；Hugo 弃用接口迁移单独记录在 [Hugo 弃用接口更新记录](../log-全面更新/)，书与影栏目记录在 [Hugo 摄影与书与影栏目扩展记录](../log-栏目扩展/)，日常操作指南记录在 [Hugo-Stacker 个性化主题使用手册](../log-使用手册/)。
>本次 Hugo 的搭建与迁移所涉及的 **代码工作** 全部由 `gpt-5.6 Sol-medium` 进行
>
>毕竟我也看不懂 Hugo 的这些新编译语言 (
>
>不过全流程框架以及内容均按本人规划设计，可以放心阅读。

本文遵循以下原则：
- 自定义功能优先放在博客根目录，因此能够用 Hugo 的特性去覆盖主题同路径文件。
- 代码记录采用“修改文件 + 关键片段 + 行为变化”，不复制未修改的完整主题文件。
- 本文标题不手工编号，目录编号由 Stack 主题负责。
> WARNING: 内含大量 ai 生成内容，尽管已经经过大量人工审核修改，但本人编程水平太低难免遗留错误，请批判的阅读。
---

## Hexo 到 Hugo 的迁移背景

Hugo 的源文件由模板、配置、内容、资源和数据组成，构建后再发布静态文件。目前将 Hugo 博客所属仓库命名为名 `LKWL.github.io`，因此 Hugo 博客暂时使用：

```toml
baseurl = "https://lkwlhsy.github.io/LKWL.github.io/"
```

将来如果正式将 Hexo 迁移到 Hugo ，则将重新命名 GitHub Pages 仓库和 `baseurl`，目前暂不在迁移阶段提前改成账号根路径 `"https://lkwlhsy.github.io/"` 。

---
## Stack 主题本地化

### 修改目的

最初通过 Hugo Modules 从互联网上游加载 Stack (即 GitHub)。这样的远程模块便于随着主题作者的更新而更新升级，但构建依赖上游仓库和 Go 模块缓存。为了让主题源码可审阅、可修改，并避免上游下线导致博客无法构建（防止作者删库跑路或者年久失修），将锁定的 Stack v4.0.0-beta.15 完整复制到：

```text
LKWLhsy/themes/stack/
```

本地副本保留主题的 `LICENSE`、`README.md`、`theme.toml`、模板、SCSS、TypeScript、i18n 和主题数据。原来采用的 `module.toml` ：
```toml
[[imports]]
    path = "github.com/CaiJimmy/hugo-theme-stack/v4"
```
来调用 Stack 原生主题（如前文所说，基于 hugo module 功能），本地化后主题调用配置变为在 `config.toml`中设置：
```toml
theme = "stack"
```

同时移除了博客根目录不再需要的远程模块导入（即 `config/module.toml`）、`go.mod` 和 `go.sum`。主题自身目录中的 `go.mod` 是主题源码元数据的一部分，不等同于博客根目录的远程依赖。

### 修改后的维护方式

基础主题功能可以编辑：

```text
themes/stack/
```

本站专属覆盖优先编辑博客根目录下的同名文件：
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

GitHub Pages 来源保持 `GitHub Actions` ，这可以将本地文件自动化推送到 Github。工作流执行以下关键步骤：

1. 检出仓库并安装 Go、Node.js、Dart Sass 和 Hugo Extended。
2. 使用 Hugo 0.164.0 构建站点。
3. 通过 `configure-pages` 注入 Pages 的部署地址。(即传入`baseURL`)
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

需要同目录图片时使用 `Page Bundle`：

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

### 推送整个博客
```bash
 git add -A && git commit -m "update blog" && git push
```
将会把所有文件和数据推送。

---
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
    customText         = "自定义填入"
    poweredByText      = "主题 <b><a href=\"https://github.com/CaiJimmy/hugo-theme-stack\" target=\"_blank\" rel=\"noopener\">Stack</a></b>由 <a href=\"https://jimmycai.com\" target=\"_blank\" rel=\"noopener\">Jimmy</a> 设计"
    privacyText        = "隐私政策"
```

模板覆盖文件是：

```text
layouts/_partials/footer/footer.html
```

该模板文件会读取 `.Site.Params.footer`模块，用 `replace` 替换 `{year}`、`{since}` 和 `{title}`，用 `safeHTML` 保留配置中写好的受控链接；运行时间由 `launchDate` 及其下属包含 `runtimeLink`、`runtimeURL`、`runtimeIcon` 等指令控制；`customText`、`poweredByText`、`privacyText`同理。

要隐藏对应内容，可以关闭相应开关。例如：

```toml
showRuntime = false
showCustomText = false
showPrivacyLink = false
```

其中 `showPrivacyLink` 只在 `showPoweredBy = true` 时独立控制隐私链接；当前模板把隐私链接放在 `showPoweredBy` 的外层条件中，因此 `showPoweredBy = false` 会同时隐藏主题署名和隐私链接。

| `showPoweredBy` | `showPrivacyLink` | 结果 |
| --- | --- | --- |
| `true` | `true` | 显示主题署名和隐私链接 |
| `true` | `false` | 只显示主题署名 |
| `false` | `true` | 两者均不显示 |
| `false` | `false` | 两者均不显示 |

要改变显示顺序，编辑 `layouts/_partials/footer/footer.html` 中各个 `<section>` 的顺序；不要只调整 TOML 字段顺序。

---
## 暂停 English
由于目前以中文内容为首，再考虑英文维护会大大增加建设时间成本，因此暂时关闭英文页面。

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

恢复时删除 `disabled = true`。英文源文件仍保存在 `content/`，因此禁用不会丢失内容。

---
## 默认卡片封面

默认图片位于：

```text
assets/img/default-cover.jpg
```

全局开关的路径位于：
```text
config/_default/params.toml
```

```toml
[cardImage]
    enabled = true
    default = "img/default-cover.jpg"
```

项目覆盖模板为：

```text
layouts/_partials/helper/card-image.html
```

模板先尝试文章、分类或页面自己的图片；没有找到时才读取 `cardImage.default`。因此默认封面用于列表卡片，不会强制插入文章正文顶部。

---
## 恢复经典的 Stack 主题首页侧栏
本博客使用的是基于 stack 的 美化版主题：[hugo-stack-starter](https://github.com/liu-houliang/hugo-stack-starter) ，该主题已在 Stack 基础上做过一次首页改造：主内容区显示两列文章卡片，同时把原生右侧组件配置注释掉。本站后来执行了两项配套修改：关闭 starter 的双列文章网格，并启用已经预留的 Stack 右侧栏。

### starter 的原始状态

首页覆盖模板从一开始就位于：

```text
layouts/home.html
```

它既定义主内容，也填充基础布局中的 `right-sidebar` 插槽：

```go-html-template
{{ define "right-sidebar" }}
    {{ partial "sidebar/right.html" (dict "Context" . "Scope" "homepage") }}
{{ end }}
```

starter 默认配置为：

```toml
[homepage]
    grid = true

[widgets]
    # homepage = [
    #     { type = "search" },
    #     { type = "archives", params = { limit = 5 } },
    #     { type = "categories", params = { limit = 10 } },
    #     { type = "tag-cloud", params = { limit = 10 } },
    # ]
    page = [{ type = "toc" }]
```

`grid = true` 会让 `home.html` 输出带 `-grid` 的列表：

```html
<section class="article-list -grid">
```

`assets/scss/partials/custom-components/_homepage-grid.scss` 中的：

```scss
.article-list.-grid {
    @media (min-width: 1024px) {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 30px;
    }
}
```
随后在桌面宽度把它改成两列。所以 starter 的“双栏”是主内容区里的两列文章卡片。

另一方面，`home.html` 虽然定义了 `right-sidebar`，但 `themes/stack/layouts/_partials/sidebar/right.html` 会按 `Scope = "homepage"` 读取 `widgets.homepage`：

```go-html-template
{{- $scope := default "homepage" .Scope -}}
{{- $context := .Context -}}
{{- with (index .Context.Site.Params.widgets $scope) -}}
    <aside class="sidebar right-sidebar sticky">
        {{ range $widget := . }}
            {{ if templates.Exists (printf "_partials/widget/%s.html" .type) }}
                <!-- Ensure that the `params` is not nil -->
                {{- $params := default dict .params -}}

                {{ partial (printf "widget/%s" .type) (dict "Context" $context "Params" $params) }}
            {{ else }}
                {{ warnf "Widget %s not found" .type }}
            {{ end }}
        {{ end }}
    </aside>
{{ end }}
```

starter 将 `widgets.homepage` 注释，因此 `with` 接收不到组件数组，模板不会生成右侧 `<aside>`。原始状态实际是“右侧栏接口存在，但没有右侧栏内容”，因此最后效果是只存在 单/双 栏卡片展示。

### 执行的配置修改

修改位置：

```text
config/_default/params.toml
```

先关闭 starter 的两列文章卡片：

```toml
[homepage]
    grid = false
```

`home.html` 不再追加 `-grid`，`_homepage-grid.scss` 的两列规则也不再命中，文章恢复为 Stack 默认的纵向列表。

然后启用首页组件：

```toml
[widgets]
    homepage = [
        { type = "search" },
        { type = "archives", params = { limit = 5 } },
        { type = "categories", params = { limit = 10 } },
        { type = "tag-cloud", params = { limit = 10 } },
    ]
    page = [{ type = "toc" }]
```

数组顺序就是组件的显示顺序。删除某一项会隐藏对应组件；修改 `limit` 只改变该组件最多列出的项目数；删除整个 `homepage` 数组会再次隐藏首页右侧栏。

### 右侧栏的完整调用链

```text
config/_default/params.toml
    widgets.homepage
        ↓
layouts/home.html
    define "right-sidebar"
        ↓
themes/stack/layouts/baseof.html
    block "right-sidebar"
        ↓
themes/stack/layouts/_partials/sidebar/right.html
    index Site.Params.widgets "homepage"
        ↓
widget/search、archives、categories、tag-cloud
```

`baseof.html` 只提供页面槽位和总体宽度：

```go-html-template
<div class="container main-container flex on-phone--column
    {{ if $hasWidget }}extended{{ else }}compact{{ end }}">
    {{ block "left-sidebar" . }}
        {{ partial "sidebar/left.html" . }}
    {{ end }}
    {{ block "right-sidebar" . }}{{ end }}
    <main class="main full-width">
        {{ block "main" . }}{{ end }}
    </main>
</div>
```

`extended` 与 `compact` 决定容器宽度，不创建组件。右侧栏真正出现需要两个条件：当前页面模板填充 `right-sidebar` 插槽，且对应 scope 的 widgets 数组非空。

### `layouts/list.html` 的实际作用

根目录的 `layouts/list.html` 不控制首页。主要模板选择关系为：

| 页面 | 模板 |
| --- | --- |
| 首页 `/` | `layouts/home.html` |
| 普通 section、分类、标签等列表 | `layouts/list.html` |
| 摄影栏目 | `layouts/photography/list.html` |
| 书与影 | `layouts/page/books-and-film.html` |

`layouts/list.html` 从 `themes/stack/layouts/list.html` 复制而来，所作的修改是将列表头图解析从：

```go-html-template
{{- $image := partial "helper/image" (dict "Image" .Params.image "Resources" .Resources "Context" .) -}}
```

改为：

```go-html-template
{{- $image := partial "helper/card-image" (dict "Image" .Params.image "Resources" .Resources "Context" .) -}}
```

原因在于 Stacker 采取的是 `\layouts\_partials\helper\card-image.html` 会先调用 stack 主题原生 `helper/image`；页面没有图片时，再读取全局 `cardImage.default`：
```toml
# ~\config\_default\params.toml
[cardImage]
    enabled = true
    default = "img/default-cover.jpg"
```
所以新增 `list.html` 的主要目的，是让普通列表页也获得默认封面回退，而不是改变首页。

该覆盖文件保留了 Stack 原有的 `right-sidebar` 定义。启用 `widgets.homepage` 后，普通 section、分类和标签页也显示相同组件。摄影栏目命中更具体的 `layouts/photography/list.html` 没有定义 `right-sidebar`，因此摄影页仍然没有右侧栏。

### 三种首页组合

starter 的“两列文章、无右栏”：

```toml
[homepage]
    grid = true

[widgets]
    page = [{ type = "toc" }]
```

单列文章、无右栏：

```toml
[homepage]
    grid = false

[widgets]
    page = [{ type = "toc" }]
```

当前的“单列文章、原生 Stack 右栏”：

```toml
[homepage]
    grid = false

[widgets]
    homepage = [
        { type = "search" },
        { type = "archives", params = { limit = 5 } },
        { type = "categories", params = { limit = 10 } },
        { type = "tag-cloud", params = { limit = 10 } },
    ]
    page = [{ type = "toc" }]
```

---
## 文章尾部个性化信息框

这块信息框将文章标签、许可协议和一行个性化文字放入同一个卡片，同时保留 Stack 原有的“最后更新时间”。它只应用于 `post` 博文底部，不会自动出现在隐私政策、关于、摄影列表等独立页面。

### 配置结构与字段作用

全局结构配置位于：

```text
config/_default/params.toml
```

```toml
[article.footerBox]
    enabled           = true
    showPersonalLabel = true
    icon              = "messages"
```

字段作用：

| 字段 | 作用 |
| --- | --- |
| `enabled` | 为 `post` 启用统一信息框；关闭后恢复无框布局 |
| `showPersonalLabel` | 只控制最后一行个性化文字 |
| `icon` | 个性化文字前调用的 Stack SVG 图标名 |

中文内容位于：

```text
config/_default/params.zh.toml
```

```toml
[article]
    headingAnchor = true
    math          = false
    readingTime   = true

    [article.footerBox]
        personalLabel = "感谢阅读"

    [article.license]
        enabled = true
        default = "采用 CC BY-NC-SA 4.0 许可协议"
```

`params.toml` 决定结构开关，`params.zh.toml` 决定中文显示文字。以后只改签名时，不需要碰模板：

```toml
[article.footerBox]
    personalLabel = "感谢阅读，愿每一次记录都有回声。"
```

隐藏不同模块：

```toml
[article.footerBox]
    enabled           = false # 整个信息框恢复为主题原布局
    showPersonalLabel = false # 保留标签和许可，只隐藏签名

[article.license]
    enabled = false           # 全站隐藏许可协议
```

单篇文章可以覆盖全局许可配置：

```yaml
license: false
```

`comments: false` 只关闭评论，不参与文章信息框判断。

### Footer 功能实现

新增根目录覆盖文件：

```text
layouts/_partials/article/components/footer.html
```

核心逻辑如下：

```go-html-template
{{- $footerBox := .Site.Params.article.footerBox -}}
{{- $showFooterBox := and $footerBox.enabled (eq .Section "post") -}}

<footer class="article-meta article-footer">
    {{- if $showFooterBox -}}
    <div class="article-footer-info">
        {{ partial "article/components/tags" . }}

        {{ if and (.Site.Params.article.license.enabled)
            (not (eq .Params.license false)) }}
        <section class="article-copyright inline-meta">
            {{ partial "helper/icon" "copyright" }}
            <span>
                {{ default .Site.Params.article.license.default
                    .Params.license | markdownify }}
            </span>
        </section>
        {{ end }}

        {{ if and $footerBox.showPersonalLabel $footerBox.personalLabel }}
        <section class="article-personal-label inline-meta">
            {{ partial "helper/icon"
                (default "messages" $footerBox.icon) }}
            <span>{{ $footerBox.personalLabel | markdownify }}</span>
        </section>
        {{ end }}
    </div>
    {{- else -}}
        {{ partial "article/components/tags" . }}
        {{/* 关闭 footerBox 时仍保留主题原有许可内容 */}}
    {{- end -}}

    {{- if ne .Lastmod .Date -}}
    <section class="article-lastmod inline-meta">
        {{ partial "helper/icon" "clock" }}
        <span>
            {{ T "article.lastUpdatedOn" }}
            {{ .Lastmod | time.Format .Site.Params.dateFormat.lastUpdated }}
        </span>
    </section>
    {{- end -}}
</footer>
```

判断顺序为：

1. `enabled` 必须为 `true`。
2. 当前页面 `.Section` 必须是 `post`。
3. 标签根据文章 `tags` 输出。
4. 许可同时受全局 `article.license.enabled` 和单篇 `license: false` 控制。
5. 个性化文字同时受显示开关和文字是否为空控制。
6. `Lastmod` 与 `Date` 不同时，最后更新时间放在信息框外，保留主题语义。

### 标签功能修改

新增覆盖文件：

```text
layouts/_partials/article/components/tags.html
```

```go-html-template
{{ if .Params.Tags }}
    <section class="article-tags">
        {{ partial "helper/icon" "tag" }}
        <div class="article-tags-links">
            {{ range (.GetTerms "tags") }}
                <a href="{{ .RelPermalink }}">{{ .LinkTitle }}</a>
            {{ end }}
        </div>
    </section>
{{ end }}
```

`.GetTerms "tags"` 返回 Hugo 已生成的 taxonomy term 页面，而不是简单打印字符串。因此每个标签都获得正确的 `RelPermalink`，并能自动适配当前项目站点的 `baseURL`。

### 信息框样式的实现

新增样式文件：

```text
assets/scss/partials/custom-components/_article-footer.scss
```

主要结构：

```scss
.article-page .main-article .article-footer {
    .article-footer-info {
        margin-top: 2rem;
        padding: 1.6rem 2rem;
        display: flex;
        flex-direction: column;
        gap: 1.2rem;
        background: var(--card-background-selected);
        border: 1px solid var(--card-separator-color);
        border-radius: 8px;

        .article-tags,
        .article-copyright,
        .article-personal-label {
            margin: 0;
            display: flex;
            align-items: center;
            gap: 10px;
            color: var(--card-text-color-secondary);
            font-size: 1.4rem;
            background: transparent;
            border: 0;
        }

        .article-tags-links {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
        }
    }
}
```

实际文件还包括：
- 16px 图标尺寸和 `flex-shrink: 0`，防止窄屏挤压图标。
- 标签按钮的背景、边框、悬停颜色和轻微位移。
- 暗色模式边框与阴影。
- 768px 以下缩小内边距、字体和标签按钮。

最后在：

```text
assets/scss/custom.scss
```

加入：

```scss
@import "partials/custom-components/article-footer";
```

这样 SCSS 继续由 Hugo Pipes 编译，不需要直接修改 `themes/stack` 的主题样式。

---
## LOG 分类与隐私政策

### 建立 LOG 分类入口

Hugo 的分类归档会根据文章 front matter 自动生成，但仅靠文章中的 `categories` 只能得到一个没有说明文字的 taxonomy term 页面。为了让 `LOG` 分类拥有固定标题和说明，在内容目录中建立：

```text
content/categories/LOG/_index.zh.md
```

文件内容为：

```yaml
---
title: "LOG"
description: "记录本站建设、主题定制、维护与发布过程。"
---
```

这里的 `_index.zh.md` 是一个 branch bundle 的入口文件：

- `categories` 对应站点在 `config/_default/config.toml` 中启用的分类 taxonomy；
- `LOG` 是 term 名称，也决定默认生成的 URL 片段；
- `.zh` 表示它属于中文站点；
- `_index.md` 负责描述列表页，而不是创建一篇普通文章。

需要归入该分类的博文在 front matter 中写：

```yaml
categories:
    - LOG
```

`tags` 与 `categories` 的职责不同：分类用于较稳定的内容分组，标签用于描述文章涉及的技术点。一篇工程日志可以只有一个 `LOG` 分类，同时拥有 `Hugo`、`Stack`、`GitHub Pages` 等多个标签。

### 建立隐私政策页面

隐私政策是一个独立页面，而不是文章列表中的普通博文：

```text
content/privacy/index.zh.md
```

它的 front matter 关闭评论和文章许可框，保留目录：

```yaml
---
title: "隐私政策"
description: "说明本站在访问、评论和使用外部资源时可能涉及的数据处理。"
date: 2026-08-11
lastmod: 2026-08-11
comments: false
license: false
toc: true
---
```

正文按照本站实际功能说明 GitHub Pages 托管、Waline 评论与浏览量、浏览器本地存储、外部字体和脚本、外部链接以及联系方式。以后增加新内容后将同步修改正文与 `lastmod`。

### 从 Footer 控制链接

全局配置位于 `config/_default/params.toml`：

```toml
[footer]
showPrivacyLink = true
privacyLink = "privacy/"
```

中文链接文字位于 `config/_default/params.zh.toml`：

```toml
[footer]
privacyText = "隐私政策"
```

`layouts/_partials/footer/footer.html` 使用 `urls.Parse` 解析 URL：存在 scheme 的地址不经过 `relLangURL`，以 `#` 开头的片段链接也保持原样，普通相对路径才交给 `relLangURL` 处理：

```go-html-template
{{- $privacyLink := default "privacy/" $footer.privacyLink -}}
        {{- $privacyURL := urls.Parse $privacyLink -}}
        {{- if and (not $privacyURL.Scheme) (not (strings.HasPrefix $privacyLink "#")) -}}
            {{- $privacyLink = $privacyLink | relLangURL -}}
        {{- end -}}
        <section class="powerby">
            {{- with $footer.poweredByText -}}
                {{ . | safeHTML }}
            {{- end -}}
            {{- if and $footer.showPrivacyLink $footer.privacyText -}}
                {{- with $footer.poweredByText }} | {{ end -}}
                <a href="{{ $privacyLink }}">{{ $footer.privacyText }}</a>
            {{- end -}}
        </section>
```

以上隐私链接片段位于外层 `{{ if $footer.showPoweredBy }}` 条件中，因此 `showPoweredBy = false` 时，即使 `showPrivacyLink = true`，隐私链接也不会输出。

这段处理同时解决多语言前缀和 GitHub Pages 项目子路径：当前站点 `baseurl` 为 `https://lkwlhsy.github.io/LKWL.github.io/`，因此 `privacy/` 会生成 `/LKWL.github.io/privacy/`，而不是错误地指向账户根路径 `/privacy/`。

配置中 `privacyLink` 的三种写法的区别如下：

| 配置值 | 处理结果 | 适用情况 |
| --- | --- | --- |
| `"privacy/"` | 经过 `relLangURL`，自动补语言和项目路径 | 本站页面，推荐 |
| `"/privacy/"` | 前导斜杠表达主机根路径，项目站点中容易绕过仓库子路径 | 只有根域部署时使用 |
| `"https://example.com/privacy/"` | 识别为外部地址，原样使用 | 跳转到另一个站点 |

因此本站内部链接优先使用不带前导斜杠的相对路径。若未来仓库改为账户根站点或自定义域名，Hugo 会根据新的 `baseURL` 重新计算链接，无需修改模板。

---
## 博客文件状态查询方法

博客定制过程中经常同时存在内容、模板、样式和图片改动。日志只能记录已经从源码或 Git 历史确认的变化，不能把工作区中所有未提交文件都自动归入最近一次工程。

本次日志整理能够明确追溯的内容包括主题本地化、Footer、默认封面、首页侧栏、文章尾部信息框、隐私政策，以及摄影和书与影栏目。其他未提交内容若无法从当前对话和提交历史确认来源，只保留原状并标记为 “当前未提交、未归因”。

提交前先查看文件级范围：

```bash
git status --short
git diff --stat
```

再分别检查已跟踪文件和新文件：

```bash
git diff --name-only
git ls-files --others --exclude-standard
```

最后用 `git diff -- <path>` 检查准备提交的模板、配置或日志。不要因为计划使用 `git add -A` 就跳过范围确认；`git add -A` 会同时暂存 "删除、新增和修改，包括与博客定制无关的用户改动"。

---
## 维护检查
>主要是给 ai 执行任务时候的要确定的基本准则
### 修改前定位真正生效的文件

Stack 的同名文件可能同时存在于站点根目录和 `themes/stack/`。修改前按以下顺序定位：

1. 从当前页面类型确定入口，例如首页为 `layouts/home.html`，普通列表为 `layouts/list.html`，摄影列表为 `layouts/photography/list.html`。
2. 在根目录查找同名模板和 partial；根目录存在时，它会覆盖主题文件。
3. 沿 `partial`、`block` 和参数访问继续追踪调用链，确认配置值真正被谁读取。
4. 对照 `themes/stack/` 中同路径文件，区分本站修改与主题原生行为。
5. 修改配置前同时检查 `params.toml`、`params.zh.toml` 和主题默认配置，避免把语言级文字写进全局结构配置。

例如首页侧栏不能只看 `grid`：必须同时检查 `home.html` 是否声明 `right-sidebar`、`sidebar/right.html` 读取哪个 scope，以及 `widgets.homepage` 是否实际启用。

### 本地预览与严格构建

编辑内容时使用草稿预览：

```bash
hugo server -D
```

发布前使用与 GitHub Actions 相同的 Hugo Extended 版本执行：

```bash
hugo --gc --minify --panicOnWarning
```

- `--gc` 清理本次资源管道不再引用的缓存条目；
- `--minify` 生成与正式部署相近的压缩资源；
- `--panicOnWarning` 让普通 warning 直接使构建失败。

为了不改变仓库中的 `public/`，审计时应输出到仓库外的临时目录：

```bash
hugo --gc --minify --panicOnWarning --destination "D:/Hugo/tmp-build"
```

构建成功只表示模板和资源管道完成，不等于浏览器布局、外部链接或交互功能已经验证。

### 浏览器验收

至少检查：

1. 首页文章列表和右侧搜索、归档、分类、标签云；
2. 普通 section、taxonomy、term 列表是否仍使用通用 `list.html`；
3. 摄影列表是否进入更具体的 `photography/list.html`，并保持无右侧栏；
4. 普通文章的标签、许可、个性化标签和最后更新时间；
5. Footer 的运行时间、隐私政策和项目子路径；
6. 默认封面、文章自有封面、PhotoSwipe 图集与本地海报；
7. 桌面宽度和约 `390px` 窄屏下是否出现整页横向滚动。

外部字体和 Waline 可能延迟或失败，不能用它们是否加载完成替代站点主体检查。必要时在浏览器网络面板中区分本站资源 404 与第三方请求失败。

### 发布前确认

完成验证后再次执行：

```bash
git diff --check
git status --short
```

`git diff --check` 用于发现行尾空白和冲突标记。确认 `public/`、`resources/`、临时构建目录及相机原片没有被加入版本控制，然后才执行提交与推送：

```bash
git add -A
git commit -m "update blog"
git push origin main
```

推送触发 GitHub Actions；仍需等待 `build` 和 `deploy` 两个作业成功，再访问正式地址检查浏览器缓存和静态资源。只有本地构建不会更新线上网站。

---
弃用接口的具体迁移记录见 [LOG-Hugo 全面更新](../log-hugo全面更新/)，日常操作见 [LOG-Hugo 使用手册](../log-hugo使用手册/)，摄影与书影栏目见 [LOG-Hugo 栏目扩展](../log-hugo栏目扩展/)。旧地址由相应文章的 `aliases` 继续兼容。
