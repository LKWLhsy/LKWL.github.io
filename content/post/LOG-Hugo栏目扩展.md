---
title: "Hugo 摄影与书与影栏目扩展记录"
description: "记录摄影 Page Bundle、响应式画廊、书与影榜单、图片处理与栏目维护方法。"
date: 2026-08-12T00:00:00+08:00
categories:
    - LOG
tags:
    - Hugo
    - 摄影
    - 书与影
    - 图片处理
comments: true
math: false
draft: false
---

本文记录摄影和“书与影”两个长期栏目从原型到当前实现的演进。部署与主题基础见 [LOG-Hugo](../log-hugo/)，弃用接口迁移见 [LOG-全面更新](../log-全面更新/)，日常配置方法见 [LOG-使用手册](../log-使用手册/)。

## 栏目目标

摄影按期发布作品，每一期拥有独立的 Page Bundle、封面、照片资源、图注和 PhotoSwipe 灯箱。“书与影”优先展示个人 Top 10 电影，再用独立的 `2026` 索引记录当年的观影和阅读，而不是让年份成为唯一分类。

本轮没有启用 English，没有新增 npm、Go 或 Hugo Module 依赖，也没有改变 GitHub Actions。中文菜单、图标、模板和数据文件都保留在站点仓库中，便于继续自定义。

## 菜单与图标

中文菜单在 `config/_default/menu.zh.toml` 中维护，摄影和书与影分别指向：

```toml
[[main]]
    identifier = "photography"
    name = "摄影"
    url = "/photography/"

[[main]]
    identifier = "books-and-film"
    name = "书与影"
    url = "/books-and-film/"
```

菜单图标放在本站覆盖层的 `assets/icons/`：

- `assets/icons/camera.svg`
- `assets/icons/book-movie.svg`

需要换图标时，保持 `identifier` 不变，只替换对应 SVG 或在菜单项中修改 `icon` 名称。菜单的显示文字、顺序和 URL 仍以 `menu.zh.toml` 为准。

## 摄影 Page Bundle

每一期摄影作品使用目录型 Page Bundle：

```text
content/photography/2026-07-22-彩虹/
├── index.md
└── photos/
    ├── DSC_9792.jpg
    ├── DSC_9800.jpg
    ├── DSC_9806.jpg
    └── DSC_9835.jpg
```

当前实际文章是“雨后彩虹”，封面和资源写在 `content/photography/2026-07-22-彩虹/index.md`：

```yaml
---
title: "20260722：雨后彩虹"
issue: 1
demoImage: "photos/DSC_9792.jpg"
cardAspect: "landscape"
resources:
    - src: "photos/DSC_9792.jpg"
      params:
          alt: "山与别墅"
          caption: "双层彩虹落在山下别墅的上方"
---
```

`demoImage` 是摄影列表卡片的封面，不会替代正文照片；正式照片放在同一 Page Bundle 的 `photos/` 目录。创建下一期可以复制这个结构，或使用：

```bash
hugo new content photography/issue-001/index.md --kind photography
```

`archetypes/photography.md` 提供 `issue`、`demoImage`、`location`、`resources` 等字段的初始模板。新文章创建后，应将真实照片复制到该文章自己的 `photos/` 目录，再修改 front matter。

## 摄影列表与卡片

摄影列表由 `layouts/photography/list.html` 渲染，详情页由摄影单页模板和根目录覆盖层共同完成。列表不使用经典 Stack 的右侧归档栏，页面保持单栏或双栏主体；卡片网格在不同宽度下自适应：

- 宽屏：三列。
- 中等宽度：两列。
- 窄屏：一列。

卡片比例通过 front matter 的 `cardAspect` 调整：

```yaml
cardAspect: "landscape"
```

可选值及其处理规格：

| 值 | 视觉比例 | 用途 |
| --- | --- | --- |
| `landscape` | 3:2 | 默认横幅照片 |
| `wide` | 16:9 | 风景或电影感画面 |
| `square` | 1:1 | 方形构图 |
| `portrait` | 2:3 | 人像或竖幅作品 |

非法值会回退到 `landscape`。列表会优先从当前 Page Bundle 找 `demoImage`，找不到时才尝试全局资源；正式文章的本地资源不会被误当作全局资源。

## 图片大小与资源流程

相机原片可以保留在本地归档，但不建议直接作为网页下载资源。推荐分成三层：

| 层级 | 目的 | 建议 |
| --- | --- | --- |
| 相机原片 | 长期保存和重新处理 | 外部硬盘或照片管理软件，不必全部提交仓库 |
| 网页母版 | Hugo 输入 | 长边约 2400–3600px，JPEG 质量约 82–88 |
| Hugo 衍生图 | 页面显示和灯箱 | 模板自动生成 WebP 缩略图与较大灯箱图 |

Lightroom 可以在导出时限制长边和质量；ImageMagick 的示例为：

```bash
magick DSC_7795.jpg -resize "3600x3600>" -strip -interlace Plane -quality 85 DSC_7795-web.jpg
```

提交前先看文件大小和实际显示尺寸。摄影 Page Bundle 的 JPG 仍可保留为源资源，但模板会通过 `.Fit` 生成 WebP，减少浏览器传输量。默认演示封面和正式摄影资源是两件事：`static/img/default-cover.jpg` 或 `assets/img/default-cover.jpg` 用于没有封面的普通文章，`photos/` 内的照片只属于当前摄影期刊。

## `photo-gallery` 短代码

`layouts/shortcodes/photo-gallery.html` 支持三种来源。

### 当前页面的全部照片

不传参数时，短代码读取当前 Page Bundle 的 `photos/*`：

```markdown
{{</* photo-gallery */>}}
```

### 全局演示资源

`global` 只适合主题演示图或站点级资源，资源由 `resources.Get` 查找：

```markdown
{{</* photo-gallery global="img/default-cover.jpg" */>}}
```

不要把文章自己的 `photos/` 路径误传给 `global`。文章本地照片应该使用 `files` 或默认的全部资源模式。

### 指定文件与分组

`files` 接受逗号分隔的文件名，短代码会先在当前 Page Bundle 中查找，也兼容省略 `photos/` 前缀：

```markdown
{{</* photo-gallery files="photos/DSC_9792.jpg,photos/DSC_9800.jpg" */>}}
```

如果需要两组照片，中间插入一段文字，直接写两个短代码：

```markdown
暴雨刚刚停下时，天空出现了第一道彩虹。

{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}

随着太阳西沉，彩虹逐渐与城市和桥梁重叠。

{{</* photo-gallery files="photos/03.jpg,photos/04.jpg" */>}}
```

这里的 `{{</* ... */>}}` 是日志中的转义展示形式；实际文章中应写成普通 Hugo 短代码（尖括号形式）。日志必须转义，否则构建日志文章时会提前执行示例短代码。

### 图注和替代文本

Page Bundle 的 `resources` 元数据提供每张照片的 `alt` 和 `caption`：

```yaml
resources:
    - src: "photos/01.jpg"
      params:
          alt: "雨后的山谷"
          caption: "彩虹从山谷上方延伸到城市边缘"
```

短代码读取这些参数，并将 `alt` 写入图片、将 `caption` 写入 `figcaption`。短代码参数 `alt` 或 `caption` 可以在特殊场景覆盖资源元数据。

### 资源查找的区别

```text
resources.Get                 全局资源：assets/ 或站点资源管道
.Page.Resources.GetMatch      当前页面 Page Bundle 的资源
.Page.Resources.Match         当前页面匹配的一组资源
```

Page Bundle 图片必须位于 `index.md` 同级或其子目录。Windows 用户不要在资源路径中使用 `~`；Hugo 不会把它展开成用户目录，结果会是资源找不到。使用项目相对路径或 Page Bundle 相对路径。

模板会为每张图生成 WebP 缩略图和灯箱图，并把宽高写进 HTML；PhotoSwipe 根据灯箱尺寸打开原图衍生版本。当前“雨后彩虹”已经修正为两个两图画廊，中间保留 Markdown 段落，而不是把四张照片挤成一个不可控的长行。

## 书与影页面

页面文件和模板分别是：

```text
content/books-and-film/index.zh.md
layouts/page/books-and-film.html
```

数据位于：

```text
data/books-and-film/top10-movies.toml
data/books-and-film/2026.toml
```

页面顶部提供两个并列索引：`Top 10` 和 `2026`。Top 10 展示长期个人榜单，`2026` 展示当年的电影和书籍。当前示例包含《星际穿越》和《蜘蛛侠：崭新之日》，每项数据包括片名、年份、导演、个人评分、海报路径和来源链接。

Top 10 海报网格在宽屏显示五列，中等宽度三列，窄屏两列；更窄的设备由 CSS 再收缩为适合阅读的布局。海报可以放在 `assets/img/` 并由 `resources.Get` 处理为 WebP，也可以使用可靠的外部来源链接；外部来源要保留 `target="_blank"` 和 `rel="noopener noreferrer"`。

添加电影时，复制 `top10-movies.toml` 中的一个项目，修改 `rank`、`title`、`cover` 和说明字段。添加年度记录时编辑 `2026.toml`，不要把年度项目混入 Top 10 数据。

## `data/` 目录的用途

根目录 `data/` 不是构建垃圾，也不是必须手动生成的缓存。Hugo 会把其中的 YAML、TOML、JSON 等文件加载为结构化数据，模板可通过 `hugo.Data` 读取。本项目使用它保存“书与影”的榜单和年度记录，让内容数据与页面布局分离：

- `content/`：需要生成文章或页面的正文。
- `data/`：供模板读取的结构化记录，不单独生成页面。
- `layouts/`：页面结构和渲染逻辑。

因此新增电影主要修改 `data/books-and-film/`，新增摄影文章则修改 `content/photography/`。

## 实际修正与验证

本栏目开发过程中修正了“雨后彩虹” Page Bundle 的资源路径、封面查找和照片计数，并加入 `files` 分组渲染。当前工作区中的摄影文章目录和栏目改动仍按 Git 状态保留；日志只记录能够从当前对话明确追溯的实现，不把未归因的其他修改伪装成已完成历史。

最近一次严格构建结果为：退出码 `0`，生成 41 个页面、4 个非页面文件、18 个处理后图片和 14 个别名；弃用警告和短代码警告均为 `0`。浏览器检查确认摄影页可以加载两组画廊，四张照片均有 WebP 资源，PhotoSwipe 可以打开，桌面和窄屏没有横向溢出。

章节标题使用语义化标题，不手工添加“一、二、三”等编号；主题自带目录负责章节编号。操作顺序仍可使用有序列表。
