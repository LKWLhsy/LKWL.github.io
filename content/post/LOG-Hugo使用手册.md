---
title: "Hugo-Stacker 个性化主题使用手册"
description: "面向日常维护的 Hugo Stack 配置、写作、摄影、书与影和发布指南。"
date: 2026-08-12T18:00:00+08:00
categories:
    - LOG
tags:
    - Hugo
    - 使用手册
    - Stack
    - 摄影
    - 书与影
comments: true
math: false
draft: false
aliases:
    - /post/log-使用手册/
---
>本次 Hugo 的搭建与迁移所涉及的 **代码工作** 全部由 `gpt-5.6 Sol-medium` 进行
>
>毕竟我也看不懂 Hugo 的这些新编译语言 (
>
>不过全流程框架以及内容均按本人规划设计，可以放心阅读。

> WARNING: 内含大量 ai 生成内容，尽管已经经过大量人工审核修改，但本人编程水平太低难免遗留错误，请批判的阅读。

---

这是一份日常维护手册，目的是协助自己修改站点。部署和主题演进见 [Hugo 部署与主题个性化记录](../log-hugo/)，弃用接口迁移见 [Hugo 弃用接口更新记录](../log-hugo全面更新/)，摄影与书与影的实现记录见 [Hugo 摄影与书与影栏目扩展记录](../log-hugo栏目扩展/)。

## 项目目录

| 目录 | 用途 | 常见修改 |
| --- | --- | --- |
| `themes/stack/` | 本地 Stack 基础主题源码 | 修改主题原始实现 |
| `layouts/` | 本站模板覆盖层 | 修改页面结构、Footer、卡片和栏目 |
| `assets/` | Sass、SVG 和资源管道输入 | 修改样式、图标和需要处理的图片 |
| `i18n/` | 界面翻译 | 修改按钮、菜单和提示文字 |
| `static/` | 原样复制到发布目录的文件 | 放置无需处理的静态资源 |
| `config/` | 站点和语言配置 | 修改 URL、菜单开关和主题参数 |
| `content/` | 文章、页面和 Page Bundle | 写普通文章和摄影作品 |
| `data/` | 模板读取的结构化数据 | 修改书与影榜单 |
| `public/` | Hugo 构建输出 | 不手动编辑，也不提交本轮日志改动 |

主题本地化后，代码修改位置可以这样判断：

- 修改主题基础行为：编辑 `themes/stack/`。
- 修改本站覆盖层：编辑 `layouts/`、`assets/`、`i18n/` 或 `static/`。
- 修改站点数据：编辑 `config/` 或 `data/`。
- 修改文章内容：编辑 `content/`。

同路径的根目录模板优先于 `themes/stack/`。如果根目录已经存在覆盖文件，优先修改覆盖文件；只有当本站没有覆盖时才修改主题源码。

### 配置合并与覆盖优先级

本站使用配置目录而不是单个 `hugo.toml`。Hugo 会合并 `config/_default/` 下的配置文件；文件名只是为了分工，最终都会进入同一个 Site configuration：

```text
config/_default/config.toml       站点、URL、输出和 taxonomy
config/_default/languages.toml    语言定义和启停
config/_default/params.toml       所有语言共享的主题结构参数
config/_default/params.zh.toml    中文文字和中文专属参数
config/_default/menu.zh.toml      中文菜单
config/_default/markup.toml       Goldmark 和代码高亮
```

模板读取 `.Site.Params.footer.enabled` 时，Hugo 已经把全局 `params.toml` 与当前语言参数合并。结构开关放在全局文件，中文句子放在 `params.zh.toml`，英文配置放在 `params.en.toml` ，独立存在，互不影响。

配置并不是唯一的优先级。模板选择从具体到通用大致为：

```text
根目录的具体模板
  -> 根目录的通用模板
  -> themes/stack 中的具体模板
  -> themes/stack 中的通用模板
```

例如摄影列表相关配置优先选择 `layouts/photography/list.html`，不会进入 `layouts/list.html`，因为这属于特色模板；普通 section 才会使用根目录 `layouts/list.html`。排查“修改没有生效”时，先找实际模板入口。

## 站点和语言配置

项目站点的 URL 在 `config/_default/config.toml` 中：

```toml
baseurl = "https://lkwlhsy.github.io/LKWL.github.io/"
theme = "stack"
```

这是项目站点 URL，发布资源需要包含仓库子路径。将来若改成账户根站点，必须同时确认仓库名、Pages 设置和 `baseurl`，不能只改其中一项。

语言在 `config/_default/languages.toml` 中维护。当前中文启用、英文保留但暂停：

```toml
[languages.en]
    disabled = true
```

要重新启用 English，删除或改为 `false`，同时检查英文内容、菜单和翻译是否完整。暂时关闭 English 不会删除英文文件，只是不生成英文页面。

Hugo 0.164.0 使用新版字段标签：

```text
languageCode      -> locale
languageName      -> label
languageDirection -> direction
```

## Markdown、Goldmark 与 MathJax

Hugo 使用 Goldmark 将 Markdown 转换为 HTML，MathJax 再在浏览器中扫描公式文本并生成 CHTML 排版。两者职责不同：Goldmark 是 Markdown 处理器，MathJax 是浏览器端数学排版器；启用 Goldmark passthrough 不等于已经加载 MathJax。

数学文章需要在 front matter 中启用：

```yaml
math: true
```

Stack 会在 `.Params.math` 或全站 `article.math` 为真时加载 `layouts/_partials/article/components/math.html`。当前站点按文章具体配置启用，默认文章不会加载 MathJax 主脚本和字体。

### `markup.toml` 的作用

Markdown 配置位于 `config/_default/markup.toml`：

```toml
[goldmark.renderer]
    unsafe = true

[goldmark.extensions.passthrough]
    enable = true
    [goldmark.extensions.passthrough.delimiters]
        block  = [['$$', '$$'], ['\[', '\]']]
        inline = [['$', '$'], ['\(', '\)']]
```

`[goldmark.renderer] unsafe = true` 允许 Markdown 正文中的原始 HTML，例如 `<details>`、`<iframe>` 或自定义 `<div>`，直接进入最终页面。它与 MathJax 没有直接关系；设为 `false` 时，Goldmark 会省略正文中的原始 HTML。本站内容由自己维护，因此保留 `true`，但不要在不可信投稿或外部同步内容中直接允许任意 HTML。

`[goldmark.extensions.passthrough]` 用于识别并保护 TeX。Goldmark 遇到登记的分隔符后，不再把其中的 `_`、`*`、`<`、`>`、`&` 等字符解释为 Markdown 强调、HTML 标签或实体，而是把整段内容交给 `layouts/_markup/render-passthrough.html`。

当前分隔符含义：

| 配置 | 用途 | 建议 |
| --- | --- | --- |
| `$$...$$` | 块公式 | 新文章优先使用 |
| `\[...\]` | 块公式兼容写法 | 可读取迁移内容，但不作为首选 |
| `$...$` | 行内公式 | 新文章优先使用 |
| `\(...\)` | 行内公式兼容写法 | 可读取迁移内容，但不作为首选 |

当前 `markup.toml` 已同时登记 `$...$` 与 `\(...\)` 为 Goldmark passthrough 行内分隔符。新文章统一使用 `$...$`，旧内容中的 `\(...\)` 仍可兼容。

完整处理流程是：

```text
Markdown 文章
    -> Goldmark 识别 passthrough 分隔符并保留 TeX
    -> Hugo 调用 render-passthrough.html
    -> 模板安全转义 <、>、& 并生成公式容器
    -> 浏览器建立 HTML DOM
    -> 本地 MathJax 4.1.3 生成 CHTML 公式与编号
```

### 公式写法

行内公式：

```markdown
轨道偏心率记为 $e$，半长轴记为 $a$。
```

普通块公式：

```markdown
$$
E^2 = p_t^2 + m^2
$$
```

需要 AMS 自动编号的多行公式：

```markdown
$$
\begin{align}
E &= mc^2, \\
p^2 &= E^2-m^2, \\
L_z &= r^2 \dot{\phi}.
\end{align}
$$
```

不需要编号的行在行末写 `\notag`：

```markdown
$$
\begin{align}
f(r) &= r^2 + a^2 \notag \\
     &= \Delta + 2Mr.
\end{align}
$$
```

需要指定编号时使用 `\tag{...}`：

```markdown
$$
E = mc^2 \tag{A.1}
$$
```

不要同时给同一行使用自动编号、`\tag` 和 `\notag`。公式特别长时不需要在文章中添加 HTML 或滚动代码；passthrough render hook 会自动建立横向滚动容器。短公式保持居中，只有内容实际超过正文宽度时才能横向移动。

## Footer 整体区域

Footer 的显示开关和结构在 `config/_default/params.toml`：

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

中文文字在 `config/_default/params.zh.toml` 的 `[footer]` 中：

```toml
copyrightText     = "© {year} {title}"
runtimeIcon       = "🚀"
runtimeText       = "已运行 {days} 天 {hours} 时 {minutes} 分"
customText        = " "
poweredByText     = "主题 <b><a href=\"https://github.com/CaiJimmy/hugo-theme-stack\" target=\"_blank\" rel=\"noopener\">Stack</a></b>由 <a href=\"https://jimmycai.com\" target=\"_blank\" rel=\"noopener\">Jimmy</a> 设计"
privacyText       = "隐私政策"
```

可用占位符：

- `{year}`：当前年份。
- `{since}`：配置的起始年份。
- `{title}`：站点标题。
- `{days}`、`{hours}`、`{minutes}`：根据 `launchDate` 计算的运行时间。

Footer 的整个布局覆盖在 `layouts/_partials/footer/footer.html`，而不是只改某一行文字。需要调整版权、运行时间、自定义文字、Powered by、隐私政策的顺序或包裹元素时，修改该文件；需要调整中文默认文字时，修改 `params.zh.toml`。外部隐私政策可以直接填完整 URL，相对页面则使用 `privacy/`。

模板调用链为：

```text
themes/stack/layouts/baseof.html
  -> partialCached "footer/footer.html"
  -> layouts/_partials/footer/footer.html（根目录覆盖）
  -> config/_default/params.toml + params.zh.toml
```

根模板使用 `partialCached` 不代表修改后必须手工清除缓存；`hugo server` 会在模板或配置变化后重新构建。若本地浏览器仍显示旧内容，先确认终端已经完成 rebuild，再强制刷新浏览器。

整个区域的开关逻辑应保持在模板中，例如：

```go-html-template
{{- with .Site.Params.footer -}}
    {{- if .enabled -}}
        {{- if .showCopyright }}...{{ end -}}
        {{- if .showRuntime }}...{{ end -}}
        {{- if .showCustomText }}...{{ end -}}
        {{- if .showPoweredBy -}}
            ...
            {{- if and .showPrivacyLink .privacyText }}...{{ end -}}
        {{- end -}}
    {{- end -}}
{{- end -}}
```

关闭方法：将单项 `show...` 改为 `false` 会隐藏对应内容，但 `showPrivacyLink` 位于 `showPoweredBy` 的外层条件之内，属于例外；`showPoweredBy = false` 会同时隐藏主题署名和隐私链接。将 `enabled = false` 会关闭整个自定义 Footer。排列顺序由模板中的块顺序决定，文字由语言参数决定。常见错误是直接删模板中的 HTML，导致以后无法仅靠配置恢复；另一个错误是把站内 `privacyLink` 写成 `/privacy/`，在 GitHub Pages 项目站点中可能指向账户根目录，应写 `privacy/` 并让 `relLangURL` 补全子路径。

## 文章尾部信息框

文章详情的尾部组件是 `layouts/_partials/article/components/footer.html`，开关在 `config/_default/params.toml`：

```toml
[article.footerBox]
    enabled           = true
    showPersonalLabel = true
    icon              = "messages"
```

个性化标签在 `config/_default/params.zh.toml`：

```toml
[article.footerBox]
    personalLabel = "感谢阅读"
```

同一组件还会显示文章标签和许可信息。许可配置位于：

```toml
[article.license]
    enabled = true
    default = "采用 CC BY-NC-SA 4.0 许可协议"
```

文章 front matter 可以用 `license: false` 关闭单篇许可，也可以用 `tags` 设置该文章的标签。需要改变整个信息框的颜色、内边距或布局时，编辑 `assets/scss/custom.scss`；不要在每篇文章里复制一套 HTML。

实际调用链为：

```text
single.html
  -> article/article.html
  -> article/components/footer.html（根目录覆盖）
       -> article/components/tags.html
       -> helper/icon.html
  -> assets/scss/partials/custom-components/_article-footer.scss
```

组件只在普通 `post` 文章且全局 `footerBox.enabled` 为真时建立外框。许可显示使用当前模板中的真实条件：

```go-html-template
{{ if and (.Site.Params.article.license.enabled) (not (eq .Params.license false)) }}
```

因此：

- 全局启用、文章未写 `license`：显示默认许可；
- 全局启用、文章写 `license: false`：仅该篇隐藏；
- 全局关闭、文章未写 `license`：隐藏许可；
- 全局关闭、文章写 `license: true`：仍然隐藏许可；
- `showPersonalLabel = false` 只隐藏个性化标签，不影响 tags 和 license。

样式文件通过 `assets/scss/custom.scss` 导入：

```scss
@import "partials/custom-components/article-footer";
```

需要改暗色、窄屏或 tag pill 时修改该 partial；不要同时在 `custom.scss` 和 partial 中定义同一选择器。

## 菜单、图标和样式

中文菜单在 `config/_default/menu.zh.toml`，英文菜单在 `config/_default/menu.en.toml`。添加菜单项时至少设置唯一的 `identifier`、显示文字和 URL。图标通常放在 `assets/icons/`，例如：

```toml
[[main]]
    identifier = "photography"
    name = "摄影"
    url = "/photography/"
    weight = 4
    [main.params]
        icon = "camera"
```

颜色、卡片比例、Footer 框和响应式断点优先写在 `assets/scss/custom.scss`。主题升级或本地主题替换时，根目录覆盖层会继续优先使用，但仍应记录自定义选择，避免误删。

菜单图标的加载链为：

```text
menu.zh.toml 中 [main.params] icon = "camera"
  -> .Site.Menus.main
  -> .Params.Icon
  -> partial "helper/icon"
  -> assets/icons/camera.svg
```

`identifier` 必须唯一，`weight` 越小越靠前。本站是 GitHub Pages 项目站点，站内菜单优先写 `url = "photography/"` 这类相对地址；若模板会调用 `relLangURL`，它会自动补上 `/LKWL.github.io/`。添加图标时保持 SVG 使用 `currentColor`，这样浅色和暗色模式可继承菜单文字颜色。

恢复或删除菜单项时，同时检查 TOML 条目和对应页面。只删菜单不会删除内容，只删内容会留下指向 404 的菜单。SCSS 的新增模块集中由 `assets/scss/custom.scss` 导入；Hugo Extended 才能编译主题 Sass，普通 Hugo 构建会失败。

## 页面和左右侧栏宽度

Stack 的基础栅格位于 `themes/stack/assets/scss/grid.scss`，但日常调整不要直接编辑该文件。应在项目的 `assets/scss/custom.scss` 末尾覆盖，这样主题升级时更容易保留本站设置。

页面结构依次为：

```text
左栏（站点信息、头像、菜单） | 中栏（文章或列表） | 右栏（TOC 和 widgets）
```

`themes/stack/layouts/baseof.html` 根据 config/default/ 配置文件中的 widgets 配置选择布局：存在启用的 widgets 时使用 `.container.extended`，否则使用 `.container.compact`。当前配置包含首页 widgets 和文章 TOC，因此通常使用 `extended`。

### 当前 `extended` 宽度

| 浏览器宽度 | 容器最大宽度 | 左栏最大比例 | 右栏最大比例 | 说明 |
| --- | ---: | ---: | ---: | --- |
| `< 768px` | 视口宽度 | 100% | 隐藏 | 各区域纵向排列 |
| `768–1023px` | 1024px | 25% | 30%，但隐藏 | 右栏到 1024px 才显示 |
| `1024–1279px` | 1280px | 20% | 30% | 三栏布局 |
| `>= 1280px` | 1536px | 15% | 25% | 宽屏三栏布局 |

中栏没有固定百分比，它使用 `flex-grow: 1` 占据剩余空间。实际正文宽度等于容器宽度扣除左右栏、栏间距、容器内边距后的剩余值。`--section-separation` 当前为 40px；左右栏比例设置得越大，正文越窄。

没有 widgets 时使用 `compact`：768px、1024px、1280px 三个宽度阶段的容器最大宽度分别为 768px、1024px、1280px，左栏比例从 25% 变为 20%。

### 推荐修改位置

在 `assets/scss/custom.scss` 末尾添加覆盖。下面是一份完整模板，数值可以按需要修改：

```scss
/* Site layout width overrides */
@media (min-width: 1024px) and (max-width: 1279.98px) {
    .container.extended {
        max-width: 1280px;
        --left-sidebar-max-width: 20%;
        --right-sidebar-max-width: 30%;
    }
}

@media (min-width: 1280px) {
    .container.extended {
        max-width: 1536px;
        --left-sidebar-max-width: 15%;
        --right-sidebar-max-width: 25%;
    }

    .container.compact {
        max-width: 1280px;
        --left-sidebar-max-width: 20%;
    }
}
```

上面是当前主题值，复制后修改目标数字即可。常见调整如下。

扩大正文中栏：同时缩小左右栏，例如：

```scss
@media (min-width: 1280px) {
    .container.extended {
        --left-sidebar-max-width: 12%;
        --right-sidebar-max-width: 20%;
    }
}
```

扩大左栏但保持右栏不变：

```scss
@media (min-width: 1280px) {
    .container.extended {
        --left-sidebar-max-width: 20%;
    }
}
```

扩大右侧目录和 widgets：

```scss
@media (min-width: 1280px) {
    .container.extended {
        --right-sidebar-max-width: 30%;
    }
}
```

扩大整个页面在超宽显示器上的最大宽度：

```scss
@media (min-width: 1536px) {
    .container.extended {
        max-width: 1720px;
    }

    .container.compact {
        max-width: 1440px;
    }
}
```

`max-width` 大于当前浏览器可用宽度时不会强行撑出页面，只会在更宽的显示器上生效。修改后至少检查 390px、768px、1024px、1280px 和 1440px；重点确认正文没有过窄、右侧目录没有挤压中栏、页面根节点没有水平滚动。

## 默认卡片封面与经典首页

普通文章的默认卡片封面由 `config/_default/params.toml` 控制：

```toml
[cardImage]
    enabled = true
    default = "img/default-cover.jpg"
```

默认文件应位于主题资源或本站资源能找到的位置。文章自身有 `image`、`images` 或 Page Bundle 资源时，会优先使用文章图片；没有图片才使用默认封面。对应的查找逻辑在 `layouts/_partials/helper/card-image.html`。

### 默认封面的调用链

根目录 `layouts/list.html` 把主题原来的：

```go-html-template
{{ partial "helper/image" (dict "Context" . "Type" "articleList") }}
```

改为：

```go-html-template
{{ partial "helper/card-image" (dict "Context" . "Type" "articleList") }}
```

`card-image.html` 先尝试文章自己的 feature image；没有时才从 `[cardImage].default` 取得全局资源。关闭 `cardImage.enabled` 会恢复“没有文章图片就不显示封面”的行为。若默认图加载失败，依次检查配置路径、文件是否位于 `assets/` 或主题可见资源目录、以及生成 HTML 是否包含项目站点子路径。图片体积大只会影响速度，不会令一个正确路径自动变成 404。

### 首页双列网格、Stack 右侧栏与普通列表

这三件事由不同代码控制，不能把 `homepage.grid` 当成“右侧栏开关”。starter 的 `layouts/home.html` 原本已经包含两部分：

```go-html-template
{{ define "main" }}
    <section class="article-list {{ if .Site.Params.homepage.grid }}-grid{{ end }}">
        ...
    </section>
{{ end }}

{{ define "right-sidebar" }}
    {{ partial "sidebar/right.html" (dict "Context" . "Scope" "homepage") }}
{{ end }}
```

当 `grid = true` 时，`_homepage-grid.scss` 只把中栏 `.article-list.-grid` 中的文章卡片排成两列。`right-sidebar` 插槽始终存在，但 starter 默认把 `widgets.homepage` 注释掉，所以 `sidebar/right.html` 没有组件可输出，看起来就只有左栏加中间双列卡片。

本站要恢复的是“原生 Stack 单列文章流 + 有内容的右侧栏”，因此必须同时做两项配置：

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
```

完整调用链为：

```text
layouts/home.html
  -> define "right-sidebar"
  -> themes/stack/layouts/_partials/sidebar/right.html
  -> Scope = "homepage"
  -> Site.Params.widgets.homepage
  -> search / archives / categories / tag-cloud widgets
```

当前根目录不存在 `layouts/_partials/sidebar/right.html` 覆盖文件，因此实际使用上述主题 partial。

`themes/stack/layouts/baseof.html` 负责接收 `right-sidebar` block；检测到 widget 配置后使用 `.container.extended`，为三栏留出宽度。关闭右侧栏时: 注释或清空 `widgets.homepage`；恢复中栏双列卡片时，把 `grid` 改回 `true`。两者可以独立组合。

`layouts/list.html` 不控制首页，它覆盖普通 section、taxonomy 和 term 列表。本站在其中的主要改动是把 `helper/image` 换成 `helper/card-image` 以支持默认封面；它仍保留 Stack 的 `right-sidebar` 定义。摄影页使用更具体的 `layouts/photography/list.html`，书与影使用 `layouts/page/books-and-film.html`，所以不会进入通用列表模板，也不会自动继承普通列表右栏。

常见错误是只设置 `grid = false` 后期待归档组件出现；这只能关闭双列文章卡片，不能凭空创建 widgets。反过来只启用 widgets 而保留 `grid = true`，会得到“中栏双列卡片 + 右侧栏”的更拥挤布局。

## 创建普通文章

普通文章可以手动创建，也可以使用 Hugo：

```bash
hugo new content post/my-post/index.md
```

目录型 Page Bundle 适合需要图片或附件的文章：

```text
content/post/my-post/
├── index.md
└── images/
    └── example.jpg
```

文章 front matter 至少设置标题、日期和是否草稿。文章自己的资源必须放在 `index.md` 同级或子目录，模板才能通过 `.Page.Resources` 找到。

## 创建摄影期刊

使用摄影 archetype：

```bash
hugo new content photography/issue-001/index.md --kind photography
```

然后建立：

```text
content/photography/issue-001/
├── index.md
└── photos/
    ├── 01.jpg
    └── 02.jpg
```

列表封面在 `demoImage`：

```yaml
demoImage: "photos/01.jpg"
cardAspect: "landscape"
```

可用 `cardAspect`：

- `landscape`：默认横向比例。
- `wide`：宽幅 16:9。
- `square`：正方形。
- `portrait`：竖幅 2:3。

摄影列表宽屏三列、中等宽度两列、窄屏一列；“书与影”海报网格使用更密集的响应式列数。摄影页不显示经典 Stack 右侧栏。

## 添加照片、封面和图注

相机原片建议先保存在照片管理软件或外部硬盘，再导出网页母版。网页母版可以长边 2400–3600px、JPEG 质量 82–88；模板会进一步生成 WebP 缩略图和灯箱图。

Lightroom 导出时设置长边和质量；ImageMagick 示例：

```bash
magick DSC_7795.jpg -resize "3600x3600>" -strip -interlace Plane -quality 85 DSC_7795-web.jpg
```

每张照片可以在 front matter 中写 `alt` 和 `caption`：

```yaml
resources:
    - src: "photos/01.jpg"
      params:
          alt: "雨后的山谷"
          caption: "彩虹从山谷上方延伸到城市边缘"
```

`alt` 用于无障碍和图片加载失败时的替代文字，`caption` 显示在图片下方。正式文章照片用 Page Bundle 资源；默认封面是全站统一的，两者不要混用。

## 使用 `photo-gallery`

短代码文件是 `layouts/shortcodes/photo-gallery.html`。

读取当前文章全部照片：

```markdown
{{</* photo-gallery */>}}
```

读取全局演示图：

```markdown
{{</* photo-gallery global="img/default-cover.jpg" */>}}
```

读取指定的 Page Bundle 文件：

```markdown
{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}
```

两组照片中间插入文字：

```markdown
第一组照片记录雨停后的山谷。

{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}

第二组照片记录城市灯光逐渐亮起的时刻。

{{</* photo-gallery files="photos/03.jpg,photos/04.jpg" */>}}
```

日志中的 `{{</* ... */>}}` 是转义形式；实际文章中请写普通的 Hugo 短代码（尖括号形式），不要把可执行写法直接放进日志。否则 Hugo 会把示例当成真正的组件执行，可能产生缺图警告或空画廊。

资源查找的区别：

```text
resources.Get                 全局资源或 assets 管道
.Page.Resources.GetMatch      当前文章 Page Bundle 的单个资源
.Page.Resources.Match         当前文章 Page Bundle 的资源集合
```

不能在资源路径中使用 `~`。Hugo 不会把它展开为 Windows 用户目录；请使用项目相对路径、`photos/xxx.jpg` 或 `assets/xxx.jpg`。

## 添加 Top 10 电影和年度记录

书与影页面使用：

```text
content/books-and-film/index.zh.md
layouts/page/books-and-film.html
data/books-and-film/top10-movies.toml
data/books-and-film/2026.toml
```

Top 10 数据示例：

```toml
[[items]]
rank = 1
title = "星际穿越"
year = 2014
cover = "img/books-and-film/movies/interstellar.jpg"
url = "https://www.paramountpictures.com/movies/interstellar"
personalRating = 10.0
director = "Christopher Nolan"
```

添加第二部电影时复制一个 `[[items]]` 项，调整 `rank`、片名、海报和链接。当前示例还包括《蜘蛛侠：崭新之日》。Top 10 与 `2026` 是并列索引；年度记录写入 `2026.toml`，不要把年度项目混进长期榜单。

海报可以放进 `assets/img/books-and-film/`，再由 `resources.Get` 处理成 WebP。外部来源链接应使用可靠页面，并保留新窗口和安全 rel 属性。页面布局在宽屏显示更多列，窄屏自动收缩，不需要为每部电影手工写 CSS。

## `data/` 目录

`data/` 保存 Hugo 模板读取的 YAML、TOML 或 JSON 结构化数据，不会像 `content/` 一样直接生成文章页面。本项目用它保存书与影的榜单和年度记录：

```text
data/books-and-film/top10-movies.toml
data/books-and-film/2026.toml
```

模板通过 `hugo.Data` 读取。新增电影修改数据文件；新增摄影作品修改 `content/photography/`；不要把结构化数据误放入 `static/` 或 `public/`。

## 隐私政策页面

隐私政策正文在：

```text
content/privacy/index.zh.md
```

它会生成 `/privacy/`。Footer 中设置：

```toml
[footer]
    showPrivacyLink = true
    privacyLink = "privacy/"
```

如果希望跳转到外部站点，填写完整的 `https://...` URL；如果使用本站页面，填写相对路径，让 Hugo 自动处理语言前缀和项目站点 base URL。

## 本地预览、构建和发布

启动本地预览：

```bash
hugo server -D
```

`-D` 会显示草稿，浏览器默认访问 Hugo 提示的本地地址。正式构建使用严格模式：

```bash
hugo --gc --minify --panicOnWarning
```

提交前先查看范围：

```bash
git status
git diff --check
```

确认只包含预期文件后提交和发布：

```bash
git add -A
git commit -m "update blog"
git push origin main
```

Hugo 没有 Hexo `hexo d` 那样的单独部署命令。本项目的“发布”是把源文件推送到 `main`，GitHub Actions 自动执行 build 和 deploy。不要把 `public/` 当作手工上传目录，也不要为了清缓存每次删除源文件；需要重新生成时直接运行 Hugo，缓存由 Hugo 和 Actions 管理。

## 常见错误排查

| 现象 | 常见原因 | 处理 |
| --- | --- | --- |
| 摄影封面空白 | `demoImage` 路径写错，或图片不在当前 Page Bundle | 确认 `demoImage: "photos/xxx.jpg"`，并检查 `photos/` |
| 文章照片找不到 | 图片放在其他文章目录或项目外 | 将图片放到当前 `index.md` 同级或子目录 |
| `global` 画廊为空 | 把 Page Bundle 路径当成全局资源 | 用 `files`，或省略参数读取当前页面全部照片 |
| 日志构建出现短代码警告 | 示例短代码没有转义 | 在日志中使用 `{{</* ... */>}}` |
| 使用 `~` 后资源不存在 | Hugo 不展开 Windows 用户目录 | 改成项目相对路径 |
| 图片体积很大 | 直接提交相机原片 | 导出网页母版，让 Hugo 生成 WebP |
| 外部字体或 Waline 加载慢 | 外部请求、网络或服务端响应延迟 | 先确认本地 HTML 和本地图片正常，再检查外部服务 |
| 本地成功、CI 失败 | Hugo 版本或 Extended 状态不一致 | 对齐本地和 `.github/workflows/deploy.yml` 的版本 |

章节标题采用语义化标题，不手工添加数字；主题自带目录负责显示章节编号。操作步骤中的有序列表仍然表示执行顺序。
