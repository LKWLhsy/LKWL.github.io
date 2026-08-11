---
title: "Hugo Stack 主题本地化与发布记录"
description: "记录 Stack v4 主题本地化、离线构建验证，以及通过 GitHub Actions 发布 Hugo 博客的流程。"
date: 2026-08-11T13:12:11+08:00
categories:
    - LOG
tags:
    - Hugo
    - GitHub
    - 主题定制
math: false
comments: true
draft: false
---

## 背景

本站正在从 Hexo 迁移到 Hugo。最初使用 Hugo Modules 从 GitHub 加载 Stack v4 主题，这种方式便于升级，但构建依赖上游仓库和 Go 模块缓存。

为了确保主题即使停止维护或上游仓库不可用，本站仍能继续构建，并且可以直接修改模板、样式和脚本，我将当前使用的主题完整保存在博客源码仓库中。

> 代码构建基于：gpt-5.6-sol 
>
> 毕竟 Hugo 稀奇古怪的代码我也改不明白(
---

## 首次部署

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

### 4. Hugo 博客发布指令

本站使用 GitHub Actions 构建并部署 GitHub Pages。Hugo 本身没有针对这种工作流的 `hugo d`；真正触发部署的命令是：

```bash
git push origin main
```

### 5. 新建文章

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

### 6. 本地预览

预览包括草稿在内的内容：

```bash
hugo server -D
```

浏览器访问：

```text
http://localhost:1313/
```

### 7. 正式构建

```bash
hugo --gc --minify
```

生成结果位于 `public/`，该目录由 Git 忽略，不需要手动提交。

### 8. 提交并发布一篇新文章

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

### 9. 接近 `hexo d` 的连续命令

确认只需要提交指定文章时，可以使用：

```bash
hugo --gc --minify && \
git add content/post/my-new-post.md && \
git commit -m "post: add my new post" && \
git push origin main
```

不建议把 `git add -A` 固定写入发布命令，因为它可能把尚未完成的配置、文章或其他文件一起提交。

### 10. 后续维护

主题源码现在由本站仓库保存。后续可以直接修改本地主题，但升级 Hugo 后仍应重新执行完整构建和页面检查。

Google Fonts、KaTeX、Waline 和表情资源目前仍可能来自外部服务；主题源码本地化不等于所有浏览器端资源已经离线化。

---

## 一期个性化(2026-08-11)：Footer 模块、默认卡片封面与原版 stack 首页

本次调整继续采用站点根目录覆盖本地 Stack 主题的方式，没有直接修改 `themes/stack/`。这样既能自行定制，又能清楚地区分本站修改与主题原始源码。

### 修改结果

- Footer 的版权、运行时间、自定义内容和主题署名不仅可以分别开关，其全部文字、图标和链接也可以在配置文件中编辑。
- 暂时停用 English 站点，但保留全部英文内容与配置，之后可以随时恢复。
- 没有设置专属图片的文章卡片和分类卡片会显示统一默认封面。
- 首页恢复为经典 Stack 单列布局，右侧显示搜索、归档、分类和标签云。
- 默认封面只用于列表与卡片；没有设置文章图片时，不会强制出现在文章正文页顶部。

### 1. Footer 整个区域改为可配置模块

最初版本只允许通过 `showCopyright`、`showRuntime`、`showCustomText` 和 `showPoweredBy` 控制各部分是否显示，只有 `customText` 可以直接修改。这仍不足以自由编辑整个 Footer，因此进一步将版权文字、运行时间图标、加载提示、运行时间格式、统计链接以及主题署名全部迁移到配置文件。

Footer 的结构分为四个模块：

1. `copyright`：版权信息。
2. `runtime`：动态运行时间和统计页链接。
3. `custom`：任意自定义 HTML。
4. `poweredBy`：Hugo、Stack 和作者署名，或者任何希望放在最后一行的内容。

整个 Footer、四个模块以及四个模块中的全部可见内容都可以单独编辑。

#### 1.1 修改全局结构配置

文件位置：

```text
config/_default/params.toml
```

加入或修改以下代码：

```toml
[footer]
    enabled          = true
    since            = 2026
    launchDate       = "2026-08-10"
    runtimeLink      = "stats/"
    showCopyright    = true
    showRuntime      = true
    showCustomText   = true
    showPoweredBy    = true
```

字段说明：

- `enabled`：控制整个 Footer。设为 `false` 后不输出 `<footer>` 元素。
- `since`：建站年份，可以在版权文字中通过 `{since}` 使用。
- `launchDate`：运行时间的起算日期，格式建议使用 `YYYY-MM-DD`。
- `runtimeLink`：点击运行时间徽章后进入的地址。相对地址会自动带上语言和项目站点路径，也可以填写完整外部网址或 `#锚点`。
- `showCopyright`：控制版权模块。
- `showRuntime`：控制运行时间模块。
- `showCustomText`：控制自由 HTML 模块。
- `showPoweredBy`：控制最后的署名模块。

如果只希望隐藏其中一行，把对应的 `show...` 改为 `false`。如果希望隐藏整个 Footer，只需修改：

```toml
[footer]
    enabled = false
```

#### 1.2 修改中文 Footer 的全部内容

文件位置：

```text
config/_default/params.zh.toml
```

当前完整中文配置为：

```toml
[footer]
    copyrightText      = "© {year} {title}"
    runtimeIcon        = "🚀"
    runtimeLoadingText = "正在计算运行时间..."
    runtimeText        = "已运行 {days} 天 {hours} 时 {minutes} 分"
    customText         = "这里可以自定义footer内容"
    poweredByText      = "使用 <a href=\"https://gohugo.io/\" target=\"_blank\" rel=\"noopener\">Hugo</a> 构建 | 主题 <b><a href=\"https://github.com/CaiJimmy/hugo-theme-stack\" target=\"_blank\" rel=\"noopener\">Stack</a></b> 由 <a href=\"https://jimmycai.com\" target=\"_blank\" rel=\"noopener\">Jimmy</a> 设计"
```

字段说明：

- `copyrightText`：第一行版权文字，可以使用 HTML 和版权占位符。
- `runtimeIcon`：运行时间徽章左侧图标，可以换成 Emoji 或简单 HTML。
- `runtimeLoadingText`：JavaScript 计算完成前短暂显示的文字。
- `runtimeText`：运行时间格式，使用运行时间占位符。
- `customText`：任意自定义内容，可以包含链接、换行和简单 HTML。
- `poweredByText`：最后一行的完整内容，不再由模板强制生成 Hugo、Stack 或 Jimmy 文字。

英文对应字段保存在：

```text
config/_default/params.en.toml
```

English 当前被禁用，但这些配置仍然保留，因此恢复 English 后不会缺少 Footer 内容。

#### 1.3 可用占位符

版权模块支持：

- `{year}`：构建时的当前年份。
- `{since}`：`params.toml` 中设置的建站年份。
- `{title}`：当前语言的网站标题。

例如：

```toml
copyrightText = "© {since} - {year} {title}. All rights reserved."
```

运行时间模块支持：

- `{days}`：已经运行的完整天数。
- `{hours}`：扣除完整天数后的小时数。
- `{minutes}`：扣除完整小时后的分钟数。

例如：

```toml
runtimeText = "本站已经运行 {days} 天 {hours} 小时 {minutes} 分钟"
```

占位符名称必须保持不变，但它们前后的文字、标点与顺序可以自由修改。

#### 1.4 HTML 与多行内容

`copyrightText`、`runtimeIcon`、`customText` 和 `poweredByText` 会作为可信 HTML 输出。这些内容只应来自本站受版本控制的配置，不应直接接收访客提交的文字。

单行内容需要转义 HTML 属性中的双引号：

```toml
customText = "联系我：<a href=\"mailto:example@example.com\">Email</a>"
```

内容较长时，可以使用 TOML 多行字符串，避免大量转义：

```toml
customText = '''
<strong>欢迎来到我的博客</strong><br>
这里记录引力波、科研学习与生活。<br>
<a href="/about/">关于我</a>
'''
```

`poweredByText` 也不必继续显示主题署名，可以完全替换为自己的内容：

```toml
poweredByText = '''
本站由黄时雨维护 · <a href="https://github.com/LKWLhsy">GitHub</a>
'''
```

#### 1.5 完整 Footer 模板代码

模板覆盖位置：

```text
layouts/_partials/footer/footer.html
```

当前完整有效代码如下：

```go-html-template
{{- $footer := .Site.Params.footer -}}
{{- if $footer.enabled -}}
<footer class="site-footer">
    {{- if $footer.showCopyright -}}
        {{- $copyrightText := default "© {year} {title}" $footer.copyrightText -}}
        {{- $copyrightText = replace $copyrightText "{year}" (now.Format "2006") -}}
        {{- $copyrightText = replace $copyrightText "{since}" (printf "%v" $footer.since) -}}
        {{- $copyrightText = replace $copyrightText "{title}" .Site.Title -}}
        <section class="copyright">
            {{ $copyrightText | safeHTML }}
        </section>
    {{- end -}}

    {{- if and $footer.showRuntime $footer.launchDate -}}
        {{- $runtimeLink := default "stats/" $footer.runtimeLink -}}
        {{- $runtimeURL := urls.Parse $runtimeLink -}}
        {{- if and (not $runtimeURL.Scheme) (not (strings.HasPrefix $runtimeLink "#")) -}}
            {{- $runtimeLink = $runtimeLink | relLangURL -}}
        {{- end -}}
        {{- $runtimeIcon := default "🚀" $footer.runtimeIcon -}}
        {{- $runtimeLoadingText := default (T "footer.loading") $footer.runtimeLoadingText -}}
        {{- $runtimeText := default (T "footer.runTime") $footer.runtimeText -}}
        <section class="powerby site-runtime-section">
            <a href="{{ $runtimeLink }}" class="stats-link">
                <span class="stats-icon">{{ $runtimeIcon | safeHTML }}</span>
                <span id="run-time-text">{{ $runtimeLoadingText }}</span>
            </a>
        </section>
        <script>
            (function() {
                const launchDate = new Date({{ $footer.launchDate | jsonify | safeJS }});
                const runTimeTemplate = {{ $runtimeText | jsonify | safeJS }};
                if (isNaN(launchDate.getTime())) {
                    return;
                }
                function updateRunTime() {
                    const now = new Date();
                    const diff = now - launchDate;
                    const days = Math.floor(diff / (1000 * 60 * 60 * 24));
                    const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
                    const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
                    const element = document.getElementById('run-time-text');
                    if (element && runTimeTemplate) {
                        element.textContent = runTimeTemplate
                            .replace('{days}', days)
                            .replace('{hours}', hours)
                            .replace('{minutes}', minutes);
                    }
                }
                document.addEventListener('DOMContentLoaded', function() {
                    updateRunTime();
                    setInterval(updateRunTime, 60000);
                });
            })();
        </script>
    {{- end -}}

    {{- if and $footer.showCustomText $footer.customText -}}
        <section class="powerby">
            {{ $footer.customText | safeHTML }}
        </section>
    {{- end -}}

    {{- if and $footer.showPoweredBy $footer.poweredByText -}}
        <section class="powerby">
            {{ $footer.poweredByText | safeHTML }}
        </section>
    {{- end -}}
</footer>
{{- end -}}
```

实现要点：

1. 第一行读取 `.Site.Params.footer`，后续不再重复书写完整配置路径。
2. 外层 `if` 控制整个 Footer 是否输出。
3. 版权文字在 Hugo 构建阶段完成 `{year}`、`{since}` 和 `{title}` 替换。
4. 相对的 `runtimeLink` 通过 `relLangURL` 自动适配项目站点子路径；完整外部网址保持原样。
5. `launchDate` 与 `runtimeText` 使用 JSON 编码后再写入 JavaScript，避免配置文字中的引号破坏脚本。
6. 运行时间每分钟刷新一次，并通过 `textContent` 写入页面。
7. 自定义内容和署名内容使用 `safeHTML`，允许在本地配置中加入链接与换行。

#### 1.6 修改模块顺序或增加模块

目前模块顺序由 `footer.html` 中四段 `<section>` 的先后顺序决定：版权、运行时间、自定义内容、署名。若要调整顺序，只需移动对应的完整条件块，不要只移动 `<section>` 而把变量计算或运行时间脚本留在原处。

例如，要增加一个独立的备案信息模块，先在 `params.toml` 中增加开关：

```toml
[footer]
    showRegistration = true
```

在 `params.zh.toml` 中增加内容：

```toml
[footer]
    registrationText = "备案信息或其他说明"
```

然后在 `footer.html` 的 `</footer>` 之前加入：

```go-html-template
{{- if and $footer.showRegistration $footer.registrationText -}}
    <section class="powerby">
        {{ $footer.registrationText | safeHTML }}
    </section>
{{- end -}}
```

如果新模块需要独立外观，再到以下文件添加样式：

```text
assets/scss/partials/custom-components/_footer.scss
```

普通文字与链接通常可以继续使用 `powerby` 类，无需新增 CSS。

### 2. 暂时停用 English

修改位置：

```text
config/_default/languages.toml
```

在 `[en]` 中增加：

```toml
disabled = true
```

英文内容、菜单和参数文件均未删除。需要恢复英文站点时，将其改为：

```toml
disabled = false
```

也可以直接删除这一行。

### 3. 设置卡片默认封面

将提供的照片 `DSC_7795.jpg` 保存为：

```text
assets/img/default-cover.jpg
```

默认卡片图片的配置位置为 `config/_default/params.toml`：

```toml
[cardImage]
    enabled = true
    default = "img/default-cover.jpg"
```

将 `enabled` 改为 `false` 可以完全停用默认封面。

新增辅助模板：

```text
layouts/_partials/helper/card-image.html
```

它首先读取文章或分类自身的 `image`；只有没有专属图片时，才读取 `cardImage.default`。因此，后续可以为任意页面包添加自己的封面：

```text
content/post/example/
├── index.md
└── cover.jpg
```

在 `index.md` 的 frontmatter 中加入：

```yaml
image: cover.jpg
```

专属图片会自动覆盖默认图片。

为了只在卡片和列表中启用回退图片，增加了以下根目录模板覆盖：

```text
layouts/_partials/article/components/header.html
layouts/_partials/article-list/default.html
layouts/_partials/article-list/compact.html
layouts/_partials/article-list/tile.html
layouts/list.html
```

这些文件来自当前本地 Stack 版本，仅将原来的 `helper/image` 调用替换为 `helper/card-image`。文章正文页继续使用原来的图片逻辑，不会因为启用默认卡片封面而自动增加头图。

升级 Stack 主题时，应将这些根目录覆盖文件与新版主题中的同路径文件进行比较。

### 4. 恢复经典 Stack 首页

修改位置：

```text
config/_default/params.toml
```

首页保持单列文章模式：

```toml
[homepage]
    grid = false
```

同时启用右侧栏：

```toml
[widgets]
    homepage = [
        { type = "search" },
        { type = "archives", params = { limit = 5 } },
        { type = "categories", params = { limit = 10 } },
        { type = "tag-cloud", params = { limit = 10 } },
    ]
```

可以调整数组顺序来改变右侧模块顺序，也可以删除某一项来隐藏对应模块。若希望重新使用两列网格文章布局，将 `grid` 改为 `true`。


---
## 二期个性化(2026-08-11)：文章信息框、LOG 分类与隐私政策

第三次个性化参考了 Stack 博客文章底部的信息框结构，但实现继续保存在项目根目录，不直接修改 `themes/stack/`。本次内容包括：只在本篇 LOG 中手工加入章节横线、把标签与许可信息收进统一卡片、增加一行全局可配置文字、创建 LOG 分类，以及建立本站自己的隐私政策页面。

### 1. 只在指定文章中加入章节横线

参考文章中的章节横线并不是主题自动生成的，而是 Markdown 水平分隔线。在需要分隔的两个顶级章节之间写入：

```markdown
上一章的最后一段。


### 2. 下一章标题
```

`---` 前后必须保留空行，避免被 Markdown 解释为其他语法。本轮只修改 `content/post/LOG.md`，没有建立全局标题样式，因此其他文章不会自动出现横线。

本篇文章的第一个 `##` 标题前不加横线；第二个及后续顶级章节前手工加入。若未来不再需要某条横线，删除对应的 `---` 即可。

### 3. 创建并使用 LOG 分类

将本文 frontmatter 从：

```yaml
categories:
    - Hugo
```

改为：

```yaml
categories:
    - LOG
```

同时创建分类元数据文件：

```text
content/categories/LOG/_index.zh.md
```

文件内容：

```yaml
---
title: "LOG"
description: "记录本站建设、主题定制、维护与发布过程。"
---
```

这里使用 `_index.zh.md`，因为分类项是 taxonomy term 列表页面，不是普通独立文章。构建后分类地址为：

```text
https://lkwlhsy.github.io/LKWL.github.io/categories/log/
```

### 4. 全局文章结尾信息框配置

结构配置写在：

```text
config/_default/params.toml
```

增加：

```toml
[article.footerBox]
    enabled           = true
    showPersonalLabel = true
    icon              = "messages"
```

字段说明：

- `enabled`：是否把文章标签、许可协议和个性化标签放入统一卡片框。
- `showPersonalLabel`：是否显示最后一行个性化文字。
- `icon`：个性化文字前使用的本地 Stack SVG 图标名称；当前使用 `messages`。

中文文字放在语言配置：

```text
config/_default/params.zh.toml
```

```toml
[article]
    headingAnchor = true
    math          = false
    readingTime   = true

    [article.footerBox]
        personalLabel = "个性化标签"

    [article.license]
        enabled = true
        default = "采用 CC BY-NC-SA 4.0 许可协议"
```

以后只需修改 `personalLabel`，就能改变所有博文信息框的最后一行。例如：

```toml
[article.footerBox]
    personalLabel = "感谢阅读，愿每一次记录都有回声。"
```

设置以下任一配置可以隐藏相应内容：

```toml
[article.footerBox]
    enabled           = false # 恢复主题原本的无框布局
    showPersonalLabel = false # 保留框，只隐藏个性化标签

[article.license]
    enabled = false           # 全站隐藏许可协议
```

单篇文章仍可通过 frontmatter 隐藏许可协议：

```yaml
license: false
```

信息框只应用于 `post` 栏目。隐私政策、关于页面等独立页面不会显示这块博客文章信息框。

### 5. 覆盖文章 Footer 模板

新增根目录覆盖文件：

```text
layouts/_partials/article/components/footer.html
```

完整代码：

```go-html-template
{{- $footerBox := .Site.Params.article.footerBox -}}
{{- $showFooterBox := and $footerBox.enabled (eq .Section "post") -}}
<footer class="article-meta article-footer">
    {{- if $showFooterBox -}}
    <div class="article-footer-info">
        {{ partial "article/components/tags" . }}

        {{ if and (.Site.Params.article.license.enabled) (not (eq .Params.license false)) }}
        <section class="article-copyright inline-meta">
            {{ partial "helper/icon" "copyright" }}
            <span>{{ default .Site.Params.article.license.default .Params.license | markdownify }}</span>
        </section>
        {{ end }}

        {{ if and $footerBox.showPersonalLabel $footerBox.personalLabel }}
        <section class="article-personal-label inline-meta">
            {{ partial "helper/icon" (default "messages" $footerBox.icon) }}
            <span>{{ $footerBox.personalLabel | markdownify }}</span>
        </section>
        {{ end }}
    </div>
    {{- else -}}
        {{ partial "article/components/tags" . }}

        {{ if and (.Site.Params.article.license.enabled) (not (eq .Params.license false)) }}
        <section class="article-copyright inline-meta">
            {{ partial "helper/icon" "copyright" }}
            <span>{{ default .Site.Params.article.license.default .Params.license | markdownify }}</span>
        </section>
        {{ end }}
    {{- end -}}

    {{- if ne .Lastmod .Date -}}
    <section class="article-lastmod inline-meta">
        {{ partial "helper/icon" "clock" }}
        <span>
            {{ T "article.lastUpdatedOn" }} {{ .Lastmod | time.Format .Site.Params.dateFormat.lastUpdated }}
        </span>
    </section>
    {{- end -}}
</footer>
```

模板先判断当前页面是否属于 `post`，再决定是否使用卡片。文章最后更新时间仍保留在卡片外，避免改变 Stack 原有语义。

### 6. 为标签增加图标和链接容器

新增：

```text
layouts/_partials/article/components/tags.html
```

完整代码：

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

主题原模板只输出一组 `<a>`；这里增加本地 `tag.svg` 图标和 `.article-tags-links` 容器，使图标、标签和换行能够稳定排列。

### 7. 信息框 SCSS

新增：

```text
assets/scss/partials/custom-components/_article-footer.scss
```

主要样式：

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
            padding: 0;
            display: flex;
            align-items: center;
            gap: 10px;
            color: var(--card-text-color-secondary);
            font-size: 1.4rem;
            line-height: 1.4;
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

实际文件还包含图标尺寸、标签悬停、暗色模式阴影和移动端间距。将它加入 `assets/scss/custom.scss`：

```scss
@import "partials/custom-components/article-footer";
```

这样样式继续通过 Hugo Pipes 编译，不需要修改本地主题副本。

### 8. 创建隐私政策页面

创建独立页面：

```text
content/privacy/index.zh.md
```

frontmatter：

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

正文根据本站当前实际配置编写，包含：

- GitHub Pages 托管与可能的基础访问日志。
- Waline 评论和浏览量；当前使用的是第三方演示服务，并非本站自托管。
- 用户主动提交的昵称、邮箱、网站和评论内容。
- Stack 使用 `localStorage` 保存明暗模式。
- 当前未启用 Cookie 同意横幅、Google Analytics 和广告。
- Google Fonts、unpkg、jsDelivr、KaTeX、PhotoSwipe、Mermaid 等外部资源可能收到网络请求所需的技术信息。
- 评论删除与政策问题的联系邮箱。
- 隐私政策是现状说明，不构成法律意见或特定司法辖区的合规保证。

隐私页面关闭评论和文章许可，也不会显示只属于 `post` 的信息框。构建后的地址为：

```text
https://lkwlhsy.github.io/LKWL.github.io/privacy/
```

当 Waline 服务地址、分析工具、广告、托管平台或外部资源发生变化时，应同步更新隐私政策。

### 9. 在站点 Footer 中加入隐私政策链接

在 `config/_default/params.toml` 的 `[footer]` 中加入：

```toml
showPrivacyLink = true
privacyLink     = "privacy/"
```

在中文语言配置的 `[footer]` 中加入：

```toml
privacyText = "隐私政策"
```

`layouts/_partials/footer/footer.html` 会判断 `privacyLink` 是站内相对地址还是外部地址。对于 `privacy/`，模板调用 `relLangURL`，自动生成包含项目仓库子路径的地址：

```go-html-template
{{- $privacyLink := default "privacy/" $footer.privacyLink -}}
{{- $privacyURL := urls.Parse $privacyLink -}}
{{- if and (not $privacyURL.Scheme) (not (strings.HasPrefix $privacyLink "#")) -}}
    {{- $privacyLink = $privacyLink | relLangURL -}}
{{- end -}}
```

因此不应把链接写成 `/privacy/`。以斜杠开头会忽略当前项目站点的 `/LKWL.github.io/` 子路径。

隐藏 Footer 隐私链接：

```toml
showPrivacyLink = false
```

若未来改成外部隐私政策，可以直接填写完整网址：

```toml
privacyLink = "https://example.com/privacy/"
```

### 10. 模板自带的引用框示例

截图中的浅色内容框是 Stack 对标准 Markdown `blockquote` 的默认样式，不需要增加 Shortcode、HTML 或模板覆盖。

在 Markdown 中，每一段前加入 `>`：

```markdown
> 这是一个引用框示例。它可以用来填写提示、补充说明、文章签名或需要特别强调的内容。
```

多行内容可以连续书写：

```markdown
> **补充说明**
>
> 第一段内容。
>
> 第二段内容，也可以加入 [链接](https://gohugo.io/) 和其他 Markdown 格式。
```

下面是本文实际渲染的板块示例：

> 记录不是终点，而是下一次修改的起点。这个引用框可以替换为文章签名、补充说明或任何希望在正文结尾强调的内容。
