---
title: "LOG-迁移记录"
description: "记录 Hexo 文章、图片迁移到 Hugo 的实际过程、适配项和验证结果。"
date: 2026-08-12T21:41:42+08:00
categories:
    - LOG
tags:
    - Hexo
    - Hugo
    - 迁移
math: false
comments: true
draft: true
---
>本次 Hugo 的搭建与迁移所涉及的 **代码工作** 全部由 `gpt-5.6 Sol-medium` 进行
>
>毕竟我也看不懂 Hugo 的这些新编译语言 (
>
>不过全流程框架以及内容均按本人规划设计，可以放心阅读。

> WARNING: 内含大量 ai 生成内容，尽管已经经过大量人工审核修改，但本人编程水平太低难免遗留错误，请批判的阅读。

---
## 迁移范围

本次从 `D:\hexo\LKWLhsyblog` 迁移 7 篇正式文章和 4 篇草稿到当前 Hugo 站点。Hexo 源目录全程只读，没有执行写入、重命名、删除、checkout 或 reset。

Hexo 中没有字面意义上的 `LOG.md`；文件 `source/_posts/2025-03-16-record.md` 的标题为 `LOG`，本次按约定迁移为 `LOG-Hexo`，并将 Hugo 标题设置为 `LOG-Hexo`。

### Hexo 只读审计清单

迁移前先枚举 `source/_posts/`、`source/_drafts/` 和文章引用的资源目录，不运行 Hexo 生成、清理或部署命令。7 篇正式文章为：

| Hexo 源文件 | Hugo Page Bundle |
| --- | --- |
| `source/_posts/2025-03-16-record.md` | `content/post/LOG-Hexo/index.md` |
| `source/_posts/2025-03-20-黑洞裸奇点与宇宙监督假设.md` | `content/post/黑洞裸奇点与宇宙监督假设/index.md` |
| `source/_posts/2025-11-08-一些问题与解答.md` | `content/post/一些问题与解答/index.md` |
| `source/_posts/2025-11-27-2025-11-25-从电磁辐射到引力波-2-1.md` | `content/post/从电磁辐射到引力波-2-1/index.md` |
| `source/_posts/2025-11-28-2025-11-28-从电磁波到引力波-0-1.md` | `content/post/从电磁波到引力波-0-1/index.md` |
| `source/_posts/2026-02-09-2026-02-09-从电磁波到引力波-0-2.md` | `content/post/从电磁波到引力波-0-2/index.md` |
| `source/_posts/2026-05-19-从0开始的EMRI.md` | `content/post/从0开始的EMRI/index.md` |

4 篇草稿为：

| Hexo 源文件 | Hugo Page Bundle |
| --- | --- |
| `source/_drafts/2019-11-01-Hexo-Theme-Snail.md` | `content/post/hexo-theme-snail/index.md` |
| `source/_drafts/2025-11-25-从电磁辐射到引力波-1.md` | `content/post/从电磁辐射到引力波-0/index.md` |
| `source/_drafts/2026-02-09-2026-02-09-从电磁波到引力波-0-2.md` | `content/post/从电磁波到引力波-0-2-draft/index.md` |
| `source/_drafts/2026-03-17-KerrGeodesicOrbit.md` | `content/post/the-theory-of-kerr-timelike-geodesic-motion/index.md` |

正式文章和草稿中存在相似标题时使用不同 Bundle 名称，避免两个 `index.md` 生成相同 URL。这里记录的是迁移时清单；后续用户删除或继续编辑某篇草稿，不改变迁移工程已经处理过 4 篇草稿的事实。

## Hugo 目标结构

文章使用 Hugo Page Bundle，文章名作为 Bundle 目录名，内容入口保留为 Hugo 要求的 `index.md`：

- `LOG-Hexo`
- `黑洞裸奇点与宇宙监督假设`
- `一些问题与解答`
- `从电磁辐射到引力波-2-1`
- `从电磁波到引力波-0-1`
- `从电磁波到引力波-0-2`
- `从0开始的EMRI`
- `hexo-theme-snail`
- `从电磁辐射到引力波-0`
- `从电磁波到引力波-0-2-draft`
- `the-theory-of-kerr-timelike-geodesic-motion`

文章头图和正文图片均放在对应 Bundle 内。共迁移 18 个图片文件，未迁移 Hexo 摄影页、页面头图和未被文章使用的 Snail 主题素材。

## 内容适配

仅进行了 Hugo 适配，不改写正文、公式、代码、引用和科学内容：

- `subtitle` 映射为 `description`。
- `tag` 统一为 `tags`；EMRI 文章的单数 `tag` 已修正。
- `header-img` 映射为 Bundle 内的 `image`。
- `catalog`/`tocnum` 映射为 `toc`。
- 含 TeX 的页面设置 `math: true`。
- `<!-- more -->`、`<!--more -->` 和 `<!-- more-->` 统一为 `<!--more-->`。
- EMRI 正文中的 Hexo 图片路径改为 Bundle 内的 `waveform1.png`、`waveform3.png` 和 `waveform5.png`。
- 正式文章和同名草稿使用不同 slug，避免 Hugo 路由冲突。

### Front matter 字段转换

逐篇保留原 `title`、`date`、分类和标签，不用文件修改时间代替文章日期。主要字段对应为：

| Hexo / Snail 字段 | Hugo 字段 | 处理原则 |
| --- | --- | --- |
| `title` | `title` | 原文保留 |
| `subtitle` | `description` | 用于卡片摘要和 meta description |
| `date` | `date` | 保留原时区和发布日期 |
| `categories` | `categories` | 标量或数组统一为 YAML 列表 |
| `tag` / `tags` | `tags` | 统一为 `tags` 列表 |
| `header-img` | `image` | 图片复制进当前 Bundle 后写相对路径 |
| `catalog` / `tocnum` | `toc` | 只转换目录开关，不给标题手工编号 |
| Hexo `_drafts` | `draft: true` | Hugo 正式构建继续排除草稿 |

一个迁移后的典型 front matter 为：

```yaml
---
title: "文章标题"
description: "原 subtitle"
date: 2026-02-09T00:00:00+08:00
categories:
    - 物理
tags:
    - 引力波
image: "header.jpg"
toc: true
math: true
draft: false
---
```

主题会给目录自动编号，因此正文标题只保留 `## 背景` 这样的语义文字，不为迁移而添加“一、二、三”。已有正文中的编号属于原文章内容时则不擅自重写。

### 图片和资源路径转换

Hexo 文章可能从 `source/images/`、与文章同名的资源目录或主题约定路径引用图片。Hugo Page Bundle 改为“内容与资源同目录”：

```text
content/post/example/
├── index.md
├── header.jpg
└── images/
    └── figure-01.png
```

正文使用相对于 `index.md` 的路径：

```markdown
![图注](images/figure-01.png)
```

模板通过 `.Page.Resources` 识别 Bundle 资源，Hugo 才能生成页面级图片、Open Graph 头图和衍生尺寸。迁移时逐张复制而不是重新压缩，迁移前后 SHA-256 一致；网页优化属于后续独立工程，不能混入“内容无损迁移”的验收。

### Hexo 特有语法与 Markdown 适配

迁移审计逐篇搜索：

- Hexo tag plugin，例如 `{% ... %}`；
- Snail 主题专用 front matter；
- `<!-- more -->` 的不同空格写法；
- 主题根路径图片和绝对站点路径；
- fenced code block 的语言标识和围栏配对；
- TeX 分隔符、`align`、`\tag` 和 `\notag`；
- 原始 HTML 与可能被 Goldmark 过滤的标签。

摘要标记统一为 Hugo 能识别的 `<!--more-->`。普通 fenced code block 原样保留语言名称，不把程序代码当公式处理。数学文章增加 `math: true`，再由 Goldmark passthrough 保护 TeX；迁移阶段不改写公式推导、符号或编号语义。

## 保护与验证

迁移前创建了目标 Hugo 工作区备份：

`D:\Hugo\_migration_backups\20260812-212712`

备份包含 Hugo 当前工作树快照、HEAD、差异补丁，以及 Hexo `source` 目录 68 个文件的 SHA-256 基线。迁移后再次核对，Hexo `source` 仍为 68 个文件且哈希一致。

静态验证结果：

- 正式文章：7 篇。
- 草稿：4 篇，均保持 `draft: true`。
- 图片：18 个，源文件与目标文件 SHA-256 全部一致。
- 正文：仅包含批准的摘要标记和 EMRI 图片路径适配。
- slug：无重复。
- 代码围栏：未发现不成对围栏。
- `git diff --check`：退出码 0。

## 迁移前的数学渲染主题源码

以下“迁移前”以 Luna 动手前的工作树备份 `D:\Hugo\_migration_backups\20260812-212712\working-tree` 为准。当时站点没有 MathJax，也没有本地安装数学排版依赖；Stack 主题使用 KaTeX 0.16.9，并由访问者的浏览器从 jsDelivr 下载。

主题文件 `themes/stack/data/external.toml` 原配置为：

```toml
KaTeX = [
    { src = "https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css", integrity = "sha384-n8MVd4RsNIU0tAv4ct0nTaAbDJwPJzDEaqSD1odI+WdtXRGWt2kTvGFasHpSy3SV", type = "style" },
    { src = "https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js", integrity = "sha384-XjKyOOlGwcjNTAIQHIpgOno0Hl1YQqzUOEleOLALmuqehneUG+vnGctmUb0ZY0l8", type = "script", defer = true },
    { src = "https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js", integrity = "sha384-+VBxd3r6XgURycqtZ117nYw44OOcIax56Z4dCRWbxyPt0Koah1uHoK0o4+/RRE05", type = "script", defer = true },
]
```

主题文件 `themes/stack/layouts/_partials/article/components/math.html` 原实现为：

```go-html-template
{{- partial "helper/external" (dict "Context" . "Namespace" "KaTeX") -}}
<script>
    window.addEventListener("DOMContentLoaded", () => {
        const elementsToRender = [".main-article", ".widget--toc"];

        elementsToRender.forEach(selector => {
            const element = document.querySelector(selector);
            if (element) {
                renderMathInElement(element, {
                    delimiters: [
                        { left: "$$", right: "$$", display: true },
                        { left: "$", right: "$", display: false },
                        { left: "\\(", right: "\\)", display: false },
                        { left: "\\[", right: "\\]", display: true }
                    ],
                    ignoredClasses: ["gist"]
                });
            }
        });
    });
</script>
```

这套实现由 KaTeX 的 `auto-render` 扫描页面文字并排版公式。KaTeX 和 MathJax 都是浏览器端数学排版器，不是 Hugo 的 Markdown 编译器；二者之间不是在 Bash 中切换编译器。

## 迁移后的主题与覆盖层修改

### 数学加载模板：KaTeX 改为本地 MathJax

没有继续修改主题内原 KaTeX 模板，而是在站点覆盖层新建 `layouts/_partials/article/components/math.html`。Hugo 会优先采用根目录同路径模板，因此该文件替代主题实现：

```go-html-template
{{- $mathJaxScript := "vendor/mathjax/4.1.3/tex-mml-chtml.js" | relURL -}}
{{- $mathJaxFontBase := "vendor/mathjax/4.1.3/fonts/" | relURL -}}
<script>
    window.MathJax = {
        tex: {
            inlineMath: [["\\(", "\\)"], ["$", "$"]],
            displayMath: [["\\[", "\\]"], ["$$", "$$"]],
            tags: "ams",
            packages: {"[+]": ["ams"]}
        },
        options: {
            skipHtmlTags: ["script", "noscript", "style", "textarea", "pre", "code"],
            enableSpeech: false,
            enableBraille: false,
            menuOptions: {
                settings: {
                    enrich: false
                }
            }
        },
        output: {
            scale: 0.9,
            displayOverflow: "overflow",
            fontPath: {{ printf "%s%%%%FONT%%%%" $mathJaxFontBase | jsonify | safeJS }}
        },
        startup: {
            typeset: true
        }
    };
</script>
<script defer src="{{ $mathJaxScript }}"></script>
```

具体变化：

- `relURL` 根据站点 `baseURL` 生成主脚本和字体地址，兼容站点根路径及 `/LKWL.github.io/` 子路径。
- `tags: "ams"` 和 AMS 包保留 `align`、`\tag`、`\notag` 等编号语义。
- `scale: 0.9` 将公式字号调整到接近正文。
- 跳过代码块，并关闭未使用的自动语音、盲文和语义增强，避免最小本地运行集请求未部署的 SRE 资源。
- 删除原有 CDN 加载和基于固定时间的公式宽度轮询。

主题文件 `themes/stack/data/external.toml` 中原 KaTeX 数组被删除，最终不再配置 KaTeX 或 MathJax CDN；Cactus 和 PhotoSwipe 配置保持不变。MathJax 现在由上面的站点覆盖模板直接从 `static/vendor/` 加载。

### Hugo passthrough 输出保护

新增 `layouts/_markup/render-passthrough.html`：

```go-html-template
{{- if eq .Type "block" -}}
<div class="math-display-scroll" tabindex="0" role="region" aria-label="可横向滚动的数学公式">
    <div class="math-display-content">\[{{ .Inner }}\]</div>
</div>
{{- else -}}
<span class="math-inline">\({{ .Inner }}\)</span>
{{- end -}}
```

修改原因是默认 passthrough 输出会让公式中的 `<`、`>`、`&` 直接进入 HTML。模板表达式 `{{ .Inner }}` 由 Hugo 自动执行 HTML 转义，生成的 DOM 文本仍向 MathJax 提供原始 TeX 字符；块公式同时得到独立的可聚焦滚动外层。既有 `config/_default/markup.toml` 在 Luna 动手前已经包含 passthrough 配置，本次没有修改该配置。

### 公式滚动与滑块样式

在 `assets/scss/custom.scss` 增加导入：

```scss
@import "partials/custom-components/math";
```

并新增 `assets/scss/partials/custom-components/_math.scss`。关键实现如下：

```scss
.main-article,
.widget--toc {
    .math-display-scroll {
        max-width: 100%;
        overflow-x: auto;
        overflow-y: hidden;
        padding: 0 0 0.8rem;
        scrollbar-color: transparent transparent;
        scrollbar-width: thin;
        touch-action: pan-x;
        -webkit-overflow-scrolling: touch;
        outline: none;

        &:hover,
        &:focus-visible {
            scrollbar-color: rgba(0, 0, 0, 0.28) transparent;
        }

        &::-webkit-scrollbar {
            height: 6px;
        }

        &::-webkit-scrollbar-track,
        &::-webkit-scrollbar-corner {
            background: transparent;
        }

        &::-webkit-scrollbar-thumb {
            background-color: transparent;
            border-radius: 999px;
        }

        &:hover::-webkit-scrollbar-thumb,
        &:focus-visible::-webkit-scrollbar-thumb {
            background-color: rgba(0, 0, 0, 0.28);
        }

        &::-webkit-scrollbar-button {
            display: none;
            width: 0;
            height: 0;
        }
    }

    .math-display-content {
        width: max-content;
        min-width: 100%;

        > mjx-container[display="true"] {
            display: block !important;
            width: max-content !important;
            min-width: 0 !important;
            max-width: none !important;
            margin: 1em auto;
            overflow: visible !important;
        }
    }
}

[data-scheme="dark"] {
    .main-article,
    .widget--toc {
        .math-display-scroll:hover,
        .math-display-scroll:focus-visible {
            scrollbar-color: rgba(255, 255, 255, 0.3) transparent;
        }

        .math-display-scroll:hover::-webkit-scrollbar-thumb,
        .math-display-scroll:focus-visible::-webkit-scrollbar-thumb {
            background-color: rgba(255, 255, 255, 0.3);
        }
    }
}
```

外层 `.math-display-scroll` 是唯一滚动层；MathJax 容器只负责排版。短公式通过内层 `min-width: 100%` 居中，长公式从完整左端开始横滑。滑块静默时透明，悬停或键盘聚焦时才显示，底部 `0.8rem` 留白防止覆盖公式。

### Open Graph 与 Twitter 头图

直接修改两个 Stack 主题文件：

- `themes/stack/layouts/_partials/head/opengraph/provider/base.html`
- `themes/stack/layouts/_partials/head/opengraph/provider/twitter.html`

修改前：

```go-html-template
{{ absURL $image.permalink }}
```

修改后：

```go-html-template
{{ absURL $image.Permalink }}
```

Hugo 资源属性区分大小写。小写字段返回空值，导致 `og:image` 和 `twitter:image` 退化为站点根地址；改为 `.Permalink` 后会输出对应 Page Bundle 头图的绝对地址。

## 新增依赖的下载与部署

### 是否执行安装

没有执行以下操作：

- 没有运行 `npm install`、`pnpm install`、`npm install -g` 或其他项目/全局安装命令。
- 没有创建或修改 `package.json`、`package-lock.json`、`pnpm-lock.yaml`。
- 没有安装或切换 Hugo、Go、Sass、Node.js，也没有修改 PATH 或系统配置。

实际操作是从 npm registry 下载两个固定版本的发布档，再把浏览器运行所需文件复制进 Hugo 的静态目录。可复现的下载与解包方式为：

```powershell
npm pack mathjax@4.1.3
npm pack @mathjax/mathjax-newcm-font@4.1.3

New-Item -ItemType Directory -Force mathjax,newcm
tar -xf mathjax-4.1.3.tgz -C mathjax
tar -xf mathjax-mathjax-newcm-font-4.1.3.tgz -C newcm
```

解包只在迁移验证临时目录进行，不是项目运行时依赖。筛选后的文件部署到：

```text
static/vendor/mathjax/4.1.3/
├── tex-mml-chtml.js
├── LICENSE-MathJax
├── THIRD-PARTY-NOTICES.md
└── fonts/
    └── mathjax-newcm/
        ├── chtml.js
        ├── package.json
        └── chtml/
            ├── dynamic/   # 40 个动态字体定义
            └── woff2/     # 105 个 Woff2 字体
```

最终共 150 个文件，约 3.30 MB。`mathjax@4.1.3` 提供 CHTML 主脚本，`@mathjax/mathjax-newcm-font@4.1.3` 提供 NewCM 字体。Hugo 构建时会把 `static/` 内容原样复制到发布目录；运行时不需要 npm，也不会访问 MathJax CDN。

## 重新验证结果

实施前另建备份 `D:\Hugo\_migration_backups\20260813-114648-latex-rebuild`，保存计划内旧文件、Git 状态、差异、HEAD 和 SHA-256。Hexo 源基线在实施前后均为 68 个文件，文件数、大小和 SHA-256 差异为 0。

使用 Hugo Extended 0.164.0 严格构建到独立验证目录，未写入 `public/`：

- 正式构建：92 页，成功。
- `--buildDrafts` 构建：102 页，成功。
- EMRI 恢复为 259 个 MathJax 容器、115 个块公式、218 个编号节点。
- “从电磁波到引力波-0.1”为 105/41/39；“从电磁波到引力波-0.2”为 7/2/2。
- 所有数学文章的原始公式分隔符残留和 `mjx-merror` 均为 0。
- 390px、768px、1440px 三种宽度均无页面水平溢出；本地脚本、动态定义和字体请求全部返回 200。
- 根路径和 `/LKWL.github.io/` 子路径均通过验证。

本次没有覆盖 `public/`，也没有提交、推送或部署。

### 验收层级不能混用

迁移完成需要区分三个层级：

1. **文件迁移完成**：目标 Bundle、front matter、正文和图片已经按清单落盘，Hexo 源仍只读，哈希与数量审计通过。
2. **Hugo 构建成功**：正式内容和 `--buildDrafts` 都能由 Hugo Extended 0.164.0 生成，模板、短代码、资源管道和本地 MathJax 文件无构建错误。
3. **浏览器显示验证通过**：实际打开生成页面，在桌面和 390px 窄屏检查目录、代码块、图片、公式编号、横向公式滚动、头图和链接；浏览器请求的本地脚本与字体返回 200，页面本身没有横向溢出。

只有第一层不能证明 Hugo 能解析内容；只有第二层不能证明公式和布局正确；只有本地浏览器显示正确也不能证明 Hexo 源未被意外改写。因此交付报告必须分别列出三类证据，不能笼统写成“迁移成功”。

### CI 与发布行为

MathJax 文件位于 `static/vendor/mathjax/4.1.3/`，已经是仓库中的静态输入。GitHub Actions 构建时 Hugo 会原样复制这些文件，不执行 npm 安装，也不从 CDN 下载 MathJax。因此 CI 只需要现有 Hugo Extended 环境；如果将来更换 MathJax 版本，必须同时更新主脚本、NewCM 字体、许可证、第三方声明和模板中的版本路径，并重新验证根路径与项目子路径。

本轮验证输出到独立临时目录，没有修改 `public/` 或 `resources/`。推送和 Pages 部署属于后续外部状态操作，只有在重新确认 Git 变更范围后才执行。
