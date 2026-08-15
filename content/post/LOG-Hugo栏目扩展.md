---
title: "Hugo 摄影与书与影栏目扩展记录"
description: "记录摄影期刊、响应式图集、书与影个人榜单、图片压缩与后续维护方法。"
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
aliases:
    - /post/log-栏目扩展/
---

>本次 Hugo 的搭建与迁移所涉及的 **代码工作** 全部由 `gpt-5.6 Sol-medium` 进行
>
>毕竟我也看不懂 Hugo 的这些新编译语言 (
>
>不过全流程框架以及内容均按本人规划设计，可以放心阅读。

> WARNING: 内含大量 ai 生成内容，尽管已经经过大量人工审核修改，但本人编程水平太低难免遗留错误，请批判的阅读。

---
## 更新目标

本次更新为博客增加两个长期栏目：

1. **摄影**：按期发布摄影作品，每一期拥有独立正文、封面、照片和图注，并使用 Stack 自带的 PhotoSwipe。
2. **书与影**：优先展示“个人 Top 10 电影”等长期榜单，再记录当年的观影与阅读。

---

## 菜单结构

修改文件：

```text
config/_default/menu.zh.toml
```

新的中文菜单顺序为：

```text
首页
归档
摄影
书与影
搜索
关于
```

摄影和书与影对应：

```toml
[[main]]
    identifier = "photography"
    name       = "摄影"
    url        = "/photography/"
    weight     = 3
    [main.params]
        icon = "camera"

[[main]]
    identifier = "books-and-film"
    name       = "书与影"
    url        = "/books-and-film/"
    weight     = 4
    [main.params]
        icon = "book-movie"
```

新增图标：

```text
assets/icons/camera.svg
assets/icons/book-movie.svg
```

菜单使用站点内部路径，由 Hugo 根据 GitHub Pages 的项目子路径生成最终 URL。

---
## 摄影模板与样式

新增模板：

```text
layouts/photography/list.html
layouts/photography/single.html
```

列表页按照：

```text
issue 降序 → date 降序
```

显示每一期的：

- 期数
- 封面
- 标题
- 日期
- 简介
- 地点
- 照片数量

新增样式：

```text
assets/scss/partials/custom-components/_photography.scss
```

并在 `assets/scss/custom.scss` 中导入。

---
## 摄影栏目的内容结构

摄影栏目使用 Hugo Page Bundle。栏目入口是 branch bundle，每一期是 leaf bundle。以下目录树：

```text
content/
└── photography/
    ├── _index.zh.md
    ├── issue-000/
    │   └── index.md
    └── issue-001/
        ├── index.md
        ├── cover.jpg
        └── photos/
            ├── 01.jpg
            ├── 02.jpg
            └── 03.jpg
```

Hugo Page Bundle 会把正文和相关图片保存在同一目录，使资源不会与其他文章混淆。官方说明：[Page Bundles](https://gohugo.io/content-management/page-bundles/)。

摄影栏目入口：

```text
content/photography/_index.zh.md
```

---

## 创建正式摄影期刊

新增原型：

```text
archetypes/photography.md
```

创建新一期：

```bash
hugo new content photography/issue-001/index.md --kind photography
```

生成后的主要 Front Matter：

```yaml
---
title: "摄影第 01 期"
description: ""
date: 2026-08-12
issue: 1
location: ""
camera: ""
lens: ""
cover: "cover.jpg"
comments: true
toc: false
license: false
draft: true
resources:
    - src: "photos/*"
      params:
          alt: ""
          caption: ""
---
```

使用步骤：

1. 将网页母版封面保存为 `cover.jpg`。
2. 将本期照片放入 `photos/`。
3. 修改标题、期数、地点和器材信息。
4. 为照片填写 `alt` 和 `caption`。
5. 将 `draft` 改为 `false`。
6. 在正文加入图集短代码。

图集短代码：

```go-html-template
{{</* photo-gallery */>}}
```

---

## 相机照片压缩

相机直接输出的 JPEG 通常仍有 8–15 MB，不应该原样批量提交到博客仓库。正确流程是：

```text
RAW / 相机原始 JPG
        ↓ 离线保存
网页母版 JPEG
        ↓ 提交到 Hugo
Hugo 生成 WebP 缩略图和灯箱图
```

### Lightroom 导出建议

```text
格式：JPEG
色彩空间：sRGB
质量：82
长边：3000 px
禁止放大：开启
输出锐化：屏幕 / 标准
元数据：仅版权信息
```

DPI 不影响网页显示体积，真正重要的是像素尺寸和压缩质量。

>建议使用电脑自带的相片查看功能缩小jpg大小至 1Mb 以下，约定缩小到 60% 。

---

## 摄影图片处理代码

新增短代码：

```text
layouts/shortcodes/photo-gallery.html
```

关键处理：

```go-html-template
{{ $thumbnail := $image.Fit "1200x1200 webp q82" }}
{{ $lightbox := $image.Fit "2400x2400 webp q85" }}
```

缩略图负责文章内展示，灯箱图负责 PhotoSwipe 放大浏览。页面不会让缩略图位置直接下载相机原始文件。

图集输出包含：

- 图片宽高，减少页面布局跳动。
- `loading="lazy"` 懒加载。
- `alt` 无障碍说明。
- `figcaption` 图注。
- PhotoSwipe 所需的原始展示宽高。
- 横图和竖图的弹性布局参数。

Hugo 会在构建时缓存处理结果 ([Hugo 图片处理](https://gohugo.io/content-management/image-processing/))。

### 短代码的三种资源模式

`photo-gallery.html` 先读取两个可选参数，再按优先级选择资源：

```go-html-template
{{- $images := slice -}}
{{- $global := .Get "global" -}}
{{- $files := .Get "files" -}}

{{- if $global -}}
    {{- with resources.Get $global -}}
        {{- $images = $images | append . -}}
    {{- else -}}
        {{- warnf "Photography demo image %q was not found" $global -}}
    {{- end -}}
{{- else if $files -}}
    {{- range split $files "," -}}
        {{- $pattern := trim . " " -}}
        {{- $resource := $.Page.Resources.GetMatch $pattern -}}
        {{- if not $resource -}}
            {{- $resource = $.Page.Resources.GetMatch (printf "photos/%s" $pattern) -}}
        {{- end -}}
        {{- with $resource -}}
            {{- $images = $images | append . -}}
        {{- else -}}
            {{- warnf "Photography image %q was not found in the current Page Bundle" $pattern -}}
        {{- end -}}
    {{- end -}}
{{- else -}}
    {{- $images = .Page.Resources.Match "photos/*" -}}
{{- end -}}
```

三种模式的用途不同：

| 写法 | 资源来源 | 使用场景 |
| --- | --- | --- |
| `global="img/default-cover.jpg"` | `assets/`，由 `resources.Get` 读取 | 示例期复用全局默认图 |
| 不传参数 | 当前 Page Bundle 的 `photos/*` | 一次渲染本期所有照片 |
| `files="photos/01.jpg,photos/02.jpg"` | 当前 Page Bundle，按给定顺序逐个匹配 | 分组、控制顺序、在组间插入正文 |

对应的文章写法示例：

```go-html-template
{{</* photo-gallery global="img/default-cover.jpg" */>}}
{{</* photo-gallery */>}}
{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}
```

`global` 不是“从任意磁盘路径取图”。`resources.Get` 只能读取 Hugo asset pipeline 中的资源，路径相对于 `assets/`，不能写 `~`、Windows 绝对路径或文章目录路径。文章自己的照片则必须使用 `.Page.Resources.GetMatch` 或 `.Page.Resources.Match`，并放在与 `index.md` 同一个 Page Bundle 下，正如前面刨析的结构展示的那样。

### 资源元数据、图片处理和 PhotoSwipe 调用链

Page Bundle front matter 中的 `resources` 为每张图提供元数据：

```yaml
resources:
    - src: "photos/DSC_9792.jpg"
      params:
          alt: "山与别墅"
          caption: "双层彩虹落在山下别墅的上方"
```

短代码读取 `.Params.alt` 和 `.Params.caption`，也允许短代码参数覆盖。每张图片再生成两份 WebP：

```go-html-template
{{- $thumbnail := $image.Fit "1200x1200 webp q82" -}}
{{- $lightbox := $image.Fit "2400x2400 webp q85" -}}
```

输出的 `<a>` 保存灯箱图地址和 `data-pswp-width`、`data-pswp-height`；Stack 的 PhotoSwipe partial 扫描 `.image-link` 后建立灯箱。完整数据流是：

```text
Page Bundle 原图
  -> .Page.Resources
  -> photo-gallery shortcode
  -> Hugo Fit 生成缩略图和灯箱 WebP
  -> figure / image-link / data-pswp-*
  -> Stack PhotoSwipe 初始化
```

因此“四张照片并排”不是 Markdown 自动规定的，而是同一次短代码调用输出四个 `.photography-gallery-item`，再由图集 flex 样式把它们排在同一组中。需要分组时应调用两次短代码，中间直接写 Markdown：

```markdown
第一组照片之前的说明。

{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}

这里是一段普通 Markdown 正文，用来说明拍摄位置或光线变化。

{{</* photo-gallery files="photos/03.jpg,photos/04.jpg" */>}}
```

---

## 书与影模板与样式

新增模板：

```text
layouts/page/books-and-film.html
```

Front Matter 通过：

```yaml
layout: "books-and-film"
```

选择该模板，不影响其他普通页面。

页面包含：

- 排名徽章
- 本地响应式 WebP 海报
- 中文名与原名
- 年份、导演、片长
- 个人评分
- 个人短评
- 官方详情链接
- 海报来源链接
- 2026 空状态
- Waline 评论

新增样式：

```text
assets/scss/partials/custom-components/_books-and-film.scss
```

海报采用紧凑的竖向卡片：宽屏五列、中等屏幕三列、窄屏两列。移动端隐藏较长短评，并缩小元数据字号，保证两列海报仍可阅读。

---
## 书与影栏目

栏目入口：

```text
content/books-and-film/index.zh.md
```

页面地址：

```text
/books-and-film/
```

页面通过并列索引切换长期榜单和年度记录：

```text
[ Top 10 ] [ 2026 ]
```

默认显示 Top 10；点击 2026 后只显示年度记录。分类状态会写入 `#top10` 或 `#2026`，刷新页面后仍能恢复当前分类。切换代码直接保存在页面模板中，不加载第三方 JavaScript。

---

## 书与影数据

框架性的数据文件存放在根目录的 `data` 文件下：

```text
data/books-and-film/top10-movies.toml
data/books-and-film/2026.toml
```

个人 Top 10 电影条目格式：

```toml
[[items]]
    rank           = 1
    title          = "星际穿越"
    originalTitle  = "Interstellar"
    year           = 2014
    director       = "Christopher Nolan"
    duration       = "168 分钟"
    cover          = "img/books-and-film/movies/interstellar.jpg"
    url            = "https://www.paramountpictures.com/movies/interstellar"
    personalRating = 10.0
    note            = "关于时间、选择与归途的科幻旅程。"
```

海报保存在根目录 `assets` 文件下：

```text
assets/img/books-and-film/movies/interstellar.jpg
```

详情信息和海报来自派拉蒙官方页面： [影片详情](https://www.paramountpictures.com/movies/interstellar) , [官方海报资源](https://public-website-assets.paramountpictures.com/paramount2025/s3fs-public/styles/poster_medium/public/intersteller_en_dvd_800x1200.jpg?itok=YxrRaJN2)

页面使用本地海报副本，避免长期热链第三方服务器。海报仅作为影片记录封面，页面同时保留来源链接。

### 添加新的 Top 10 电影

1. 将海报保存到 `assets/img/books-and-film/movies/`。
2. 在 `top10-movies.toml` 中增加一个 `[[items]]`。
3. 填写唯一的 `rank`。
4. 填写标题、年份、导演、链接和个人短评。
5. 执行严格构建。

### 添加 2026 记录

电影：

```toml
[[movies]]
    title          = "电影名称"
    watchedDate    = "2026-08-12"
    cover          = "img/books-and-film/movies/example.jpg"
    personalRating = 8.5
    note            = "个人短评。"
    url             = "https://example.com/"
```

书籍：

```toml
[[books]]
    title          = "书名"
    author         = "作者"
    readDate       = "2026-08-12"
    cover          = "img/books-and-film/books/example.jpg"
    personalRating = 9.0
    note            = "阅读短评。"
    url             = "https://example.com/"
```

当前 2026 数据为空，因此页面显示明确的空状态。

---

## 构建与发布

发布前严格检查：

```bash
hugo --gc --minify --panicOnWarning
```

确认无误后：

```bash
hdeploy "新增摄影与书与影栏目"
```

或使用标准 Git 命令：

```bash
git add -A
git commit -m "Add photography and books-and-film sections"
git push
```

---

## 书与影索引与海报网格调整

### 问题定位与真实文件

书与影的真实网格选择器是 `.media-grid`，结构和样式分别位于：

```text
layouts/page/books-and-film.html
assets/scss/partials/custom-components/_books-and-film.scss
```

实际问题不是某个“单列配置”，而是早期页面只有一个条目时，缺少并列索引和紧凑的海报网格，单张卡片在主内容区域显得过大。调整后的工程拆成四层：内容入口决定页面模板，`data/` 保存条目，模板组织标签和卡片，SCSS 决定海报比例与响应式列数。

### 页面入口与模板选择

书与影入口文件是：

```text
content/books-and-film/index.zh.md
```

其中两个字段决定页面类型和布局：

```yaml
layout: "books-and-film"
type: "page"
```

Hugo 因此使用：

```text
layouts/page/books-and-film.html
```

而不是通用的 `layouts/single.html` 或 `layouts/list.html`。该模板只定义 `main`，没有定义 `right-sidebar`，因此书与影页面使用完整主内容区域，不加载首页或普通列表的右侧组件。

### 数据文件与模板读取

书与影的内容数据不写死在 HTML 中，而是分别保存在：

```text
data/books-and-film/top10-movies.toml
data/books-and-film/2026.toml
```

模板开头通过 `hugo.Data` 读取这两个文件：

```go-html-template
{{- $collection := index hugo.Data "books-and-film" -}}
{{- $ranking := index $collection "top10-movies" -}}
{{- $yearData := index $collection "2026" -}}
{{- $movies := default (slice) $yearData.movies -}}
{{- $books := default (slice) $yearData.books -}}
```

文件名与模板键一一对应：`top10-movies.toml` 对应 `top10-movies`，`2026.toml` 对应 `2026`。`default (slice)` 把缺失的 `movies` 或 `books` 转成空集合，避免模板在年度数据尚未填写时构建失败。

当前 `2026.toml` 只有标题、说明、类型和权重，没有 `movies` 或 `books`，所以模板进入空状态并显示“2026 年的观影与阅读记录尚未添加”。

### 栏目扩展的数据流

从菜单进入页面后，完整调用链是：

```text
config/_default/menu.zh.toml
  → content/books-and-film/index.zh.md
  → layout: books-and-film
  → layouts/page/books-and-film.html
  → hugo.Data["books-and-film"]
  → top10-movies.toml / 2026.toml
  → resources.Get 读取 assets 海报
  → Hugo Fill 生成 WebP
  → .media-grid 响应式卡片
  → 浏览器标签脚本切换面板
```

这条链也决定日常修改位置：页面标题和简介属于 `content/`，电影与书籍条目属于 `data/`，标签和卡片结构属于 `layouts/`，列数、比例和视觉样式属于 `assets/scss/`。

### 分类索引与状态同步

Top 10 和 2026 是模板 `layouts/page/books-and-film.html` 中明确声明的两个并列索引：

```html
<nav class="books-and-film-tabs" role="tablist" aria-label="书与影分类">
    <button type="button" role="tab" id="tab-top10" aria-controls="panel-top10" aria-selected="true" data-books-tab="top10">Top 10</button>
    <button type="button" role="tab" id="tab-2026" aria-controls="panel-2026" aria-selected="false" tabindex="-1" data-books-tab="2026">2026</button>
</nav>
```

每个按钮的 `data-books-tab` 必须与对应面板的 `data-books-panel` 相同。模板脚本收集所有标签和面板，再由 `activate()` 同步状态：

```javascript
const activate = (name, updateHash = false) => {
    if (!validTabs.has(name)) name = 'top10';

    tabs.forEach((tab) => {
        const active = tab.dataset.booksTab === name;
        tab.setAttribute('aria-selected', String(active));
        tab.tabIndex = active ? 0 : -1;
    });
    panels.forEach((panel) => {
        panel.hidden = panel.dataset.booksPanel !== name;
    });

    if (updateHash) history.replaceState(null, '', `#${name}`);
};
```

各状态的作用如下：

- `aria-selected` 告诉辅助技术哪个标签处于选中状态。
- `tabIndex` 只让当前标签进入正常键盘顺序。
- `hidden` 控制两个内容面板的实际显示与隐藏。
- URL hash 保存 `#top10` 或 `#2026`，刷新后仍能恢复选择。
- 非法或不存在的 hash 通过 `validTabs` 回退到 `top10`。
- 左右方向键循环选择相邻标签，并把键盘焦点移动到新标签。

脚本直接保存在页面模板中，不依赖第三方 JavaScript。以后增加第三个并列索引时，需要同时增加数据入口、按钮和面板；只要 `data-books-tab` 与 `data-books-panel` 对应，现有脚本会自动把新标签加入切换集合。

### 海报资源管道

Top 10 数据中的 `cover` 是相对于 `assets/` 的资源路径。例如：

```toml
cover = "img/books-and-film/movies/interstellar.jpg"
```

对应源文件为：

```text
assets/img/books-and-film/movies/interstellar.jpg
```

模板先读取资源，再生成固定比例的 WebP：

```go-html-template
{{- $item := . -}}
{{- $poster := resources.Get .cover -}}
{{ with $poster }}
    {{- $preview := .Fill "520x780 webp q82" -}}
    <img src="{{ $preview.RelPermalink }}"
         width="{{ $preview.Width }}"
         height="{{ $preview.Height }}"
         loading="lazy"
         alt="{{ $item.title }}海报">
{{ else }}
    <div class="media-poster-placeholder">{{ substr $item.title 0 1 }}</div>
{{ end }}
```

完整过程是：

```text
data.cover
  → resources.Get
  → assets/img/books-and-film/movies/
  → Fill "520x780 webp q82"
  → $preview.RelPermalink
  → 浏览器加载本地 WebP
```

海报放在 `assets/` 而不是 `static/`，因为 `resources.Get` 和 `Fill` 只能处理 Hugo 资源管道中的图片。找不到海报时模板不会输出损坏的 `<img>`，而是显示片名首字符占位块。`posterSource` 只生成“海报来源”链接，不参与海报下载或构建。

### 响应式海报网格

网格样式位于：

```text
assets/scss/partials/custom-components/_books-and-film.scss
```

该文件由 `assets/scss/custom.scss` 导入。当前真实列数为：

```scss
.media-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
}

@media (min-width: 768px) {
    .media-grid {
        grid-template-columns: repeat(3, minmax(0, 1fr));
    }
}

@media (min-width: 1200px) {
    .media-grid {
        grid-template-columns: repeat(5, minmax(0, 1fr));
    }
}
```

窄屏保留两列以保持海报列表的紧凑感，768px 起使用三列，1200px 起使用五列。`minmax(0, 1fr)` 允许卡片等分可用宽度，同时避免卡片内容把网格轨道强行撑宽。

海报容器与图片分别使用：

```scss
.media-poster {
    aspect-ratio: 2 / 3;
}

.media-poster img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}
```

Hugo 生成的 520×780 图片和 CSS 的 2:3 比例一致，因此各卡片海报等高；`.media-details` 再负责标题、年份、导演、片长、评分、状态和备注，不需要为每一部电影单独写布局。

### 示例条目与日常添加

Top 10 数据文件是：

```text
data/books-and-film/top10-movies.toml
```

《蜘蛛侠：崭新之日》的当前条目保留“待上映 / 评分待定”，避免把演示排名写成已经形成的个人评价：

```toml
[[items]]
    rank          = 2
    title         = "蜘蛛侠：崭新之日"
    originalTitle = "Spider-Man: Brand New Day"
    year          = 2026
    director      = "Destin Daniel Cretton"
    cover         = "img/books-and-film/movies/spider-man-brand-new-day.jpg"
    url           = "https://www.sonypictures.com/movies/spidermanbrandnewday"
    status        = "待上映 / 评分待定"
    note          = "用于展示榜单海报网格的待上映示例，名次并非最终评价。"
```

该条目的资料和海报均保留原工程记录中的官方来源：

- [Sony 官方影片页](https://www.sonypictures.com/movies/spidermanbrandnewday)
- [Sony 官方海报](https://www.sonypictures.com/sites/default/files/styles/max_860x460/public/title-key-art/spidermanbrandnewday_onesheet_1400x2100.jpg?itok=Nh6VAAh-)

下载后的 JPEG 为 69,023 字节，SHA-256：

```text
AC7C3710284AFEED0424F3795151184F7CA756115D0722BB5E591D12692C9E59
```

新增 Top 10 电影时只需要：

1. 把海报保存到 `assets/img/books-and-film/movies/`。
2. 在 `top10-movies.toml` 增加一个 `[[items]]`。
3. 设置唯一的 `rank`、片名、年份、海报路径和影片链接。
4. 按实际情况填写导演、片长、个人评分、状态、备注和海报来源。

新增 2026 年度记录时，在 `data/books-and-film/2026.toml` 中添加：

```toml
[[movies]]
title = "电影名称"

[[books]]
title = "书籍名称"
```

当前年度面板模板只读取条目的 `title`。如果以后希望年度条目也显示海报、评分或日期，需要先扩展 `layouts/page/books-and-film.html` 中的年度卡片结构，再增加对应数据字段；仅向 TOML 添加模板未读取的字段不会自动显示。

---

## 摄影页布局与卡片比例

### 模板选择与右侧栏

摄影栏目使用两个更具体的模板：

```text
layouts/photography/list.html
layouts/photography/single.html
```

Hugo 生成摄影栏目入口时优先选择 `layouts/photography/list.html`，不会进入通用 `layouts/list.html`。生成单期详情时使用 `layouts/photography/single.html`。

这两个模板都只定义了 `main`，没有定义 `right-sidebar`：

```go-html-template
{{ define "main" }}
    ...
{{ end }}
```

本地 Stack 基础模板 `themes/stack/layouts/baseof.html` 对右侧栏只提供一个空 block：

```go-html-template
{{- block "right-sidebar" . -}}{{ end }}
```

因此摄影页面不显示右侧栏是模板选择的结果，不是通过 CSS 把右栏隐藏。列表页可以使用完整主内容宽度；详情页继续渲染文章、评论、Footer 和 PhotoSwipe。

从工程演进看，这两个项目覆盖模板在该次栏目改造前都曾定义 `right-sidebar`；当时从以下文件删除了该 block：

```text
layouts/photography/list.html
layouts/photography/single.html
```

所以“取消右侧栏”不是主题全局配置，也没有修改通用 `layouts/list.html`：它是摄影专用模板的一次局部改动。当前源码中已看不到被删除的 block，只保留上述 `main` 定义；其他栏目仍按各自模板决定是否提供右侧栏。

### 与书与影统一页面外壳

摄影标题和期刊网格共同放在以下容器中：

```html
<article class="photography-shell card">
```

这段结构位于 `layouts/photography/list.html`，对应样式位于：

```text
assets/scss/partials/custom-components/_photography.scss
```

`.photography-shell` 复用 Stack 的 `card` 外观；`.photography-hero` 提供标题区和底部分隔线；`.photography-issue`、`.photography-issue-link` 与详情区再定义期刊卡片的边框、圆角、阴影、内边距和等高内容区域。因此“与书与影看齐”指两者都采用“栏目标题 + 规则网格 + 统一卡片”的页面层次，不代表共用 `.media-grid` 或同一套 SCSS。

### 栏目配置与列表数据流

摄影栏目入口配置位于：

```text
content/photography/_index.zh.md
```

当前配置为：

```yaml
title: "光与影"
description: "摄影，是光影的艺术，是时空的快照。"
cardAspect: "landscape"
comments: true
draft: false
```

列表模板读取栏目参数和所有正式摄影页面，完整数据流是：

```text
content/photography/_index.zh.md
  → .Params.cardAspect
  → layouts/photography/list.html
  → .RegularPages
  → 按 Params.issue 降序
  → Page Bundle 封面资源
  → Hugo Fill 生成 WebP
  → .photography-issues 响应式网格
```

排序代码为：

```go-html-template
{{- $issues := sort .RegularPages "Params.issue" "desc" -}}
```

因此每一期必须在 front matter 中提供可转换为整数的 `issue`。日期只负责显示，不决定列表顺序。

### 封面选择优先级

封面处理代码位于：

```text
layouts/photography/list.html
```

当前逻辑为：

```go-html-template
{{- $cover := false -}}
{{- with .Params.demoImage -}}
    {{- $cover = $page.Resources.GetMatch . -}}
    {{- if not $cover }}{{ $cover = resources.Get . }}{{ end -}}
{{- else -}}
    {{- with .Params.cover -}}
        {{- $cover = $page.Resources.GetMatch . -}}
    {{- end -}}
{{- end -}}
```

优先级如下：

1. 存在 `demoImage` 时，先在当前摄影 Page Bundle 中查找。
2. Bundle 中找不到 `demoImage` 时，再调用全局 `resources.Get`，因此可以引用 `assets/` 中的示例图。
3. 没有 `demoImage` 时才读取 `cover`，并且 `cover` 只从当前 Page Bundle 查找。
4. 两者都找不到时，卡片仍然输出标题和信息，但不会生成封面 `<img>`。

照片数量来自：

```go-html-template
{{- $photoCount := len (.Resources.Match "photos/*") -}}
{{- if and (eq $photoCount 0) .Params.demoImage }}{{ $photoCount = 1 }}{{ end -}}
```

正式摄影期统计 `photos/` 下的资源；只有 demo 条目在没有本地照片时按 1 张处理。

### 自定义摄影卡片比例

卡片比例由栏目入口统一配置：

```yaml
cardAspect: "landscape"
```

模板首先验证允许值，并建立图片处理参数表：

```go-html-template
{{- $aspect := default "landscape" .Params.cardAspect -}}
{{- $validAspects := slice "landscape" "wide" "square" "portrait" -}}
{{- if not (in $validAspects $aspect) }}{{ $aspect = "landscape" }}{{ end -}}
{{- $fillSpecs := dict
    "landscape" "720x480 webp q82"
    "wide" "720x405 webp q82"
    "square" "720x720 webp q82"
    "portrait" "600x900 webp q82"
-}}
```

四种配置对应：

| 值 | 卡片比例 | Hugo 输出 |
| --- | --- | --- |
| `landscape` | 3:2 | 720×480 WebP |
| `wide` | 16:9 | 720×405 WebP |
| `square` | 1:1 | 720×720 WebP |
| `portrait` | 2:3 | 600×900 WebP |

非法值会回退到 `landscape`。模板将比例写入 class：

```go-html-template
<section class="photography-issues photography-aspect-{{ $aspect }}">
```

SCSS 再用同一个 class 设置 `aspect-ratio`，确保 Hugo 生成尺寸和浏览器卡片形状一致。比例在栏目级统一，而不是为每一期单独设置，从而避免同一网格中的卡片高低不齐。

### 摄影响应式网格

摄影样式位于：

```text
assets/scss/partials/custom-components/_photography.scss
```

该文件同样由 `assets/scss/custom.scss` 导入。实际网格规则是：

```scss
.photography-issues {
    grid-template-columns: 1fr;
}

@media (min-width: 768px) {
    .photography-issues {
        grid-template-columns: repeat(2, minmax(0, 1fr));
    }
}

@media (min-width: 1200px) {
    .photography-issues {
        grid-template-columns: repeat(3, minmax(0, 1fr));
    }
}
```

因此摄影栏目默认一列，768px 起两列，1200px 起三列。它与书与影共享“栏目标题 + 卡片网格”的视觉思路，但使用独立模板、独立资源来源和独立 SCSS；修改 `.media-grid` 不会改变摄影列数，修改 `.photography-issues` 也不会影响书与影。

### 摄影详情与 PhotoSwipe

摄影详情模板是：

```text
layouts/photography/single.html
```

`layouts/photography/single.html` 中写的是 partial 名称；Hugo 会先查找项目根目录覆盖层，再回退到 `themes/stack/`。当前调用顺序和实际落盘文件为：

```text
partial "article/article.html"
  → themes/stack/layouts/_partials/article/article.html
  → 摄影正文和 photo-gallery 短代码
partial "comments/include"（仅 comments 不为 false）
  → themes/stack/layouts/_partials/comments/include.html
partialCached "footer/footer"
  → layouts/_partials/footer/footer.html（本站根目录覆盖）
partialCached "article/components/photoswipe"
  → themes/stack/layouts/_partials/article/components/photoswipe.html
```

列表页只负责期刊入口和封面；正文照片的分组、WebP 缩略图、灯箱图及 PhotoSwipe 数据仍由前文记录的 `photo-gallery` 短代码负责。

---

## 栏目扩展维护映射

书与影和摄影的日常修改位置如下：

| 修改目标 | 修改文件 | 作用层 |
| --- | --- | --- |
| 书与影标题、简介和评论开关 | `content/books-and-film/index.zh.md` | 页面配置 |
| Top 10 电影条目 | `data/books-and-film/top10-movies.toml` | 结构化数据 |
| 2026 年电影与书籍 | `data/books-and-film/2026.toml` | 结构化数据 |
| 分类标签、面板、数据读取和卡片 HTML | `layouts/page/books-and-film.html` | 页面结构与交互 |
| 海报列数、比例和卡片外观 | `assets/scss/partials/custom-components/_books-and-film.scss` | 页面样式 |
| 摄影标题、简介和统一卡片比例 | `content/photography/_index.zh.md` | 栏目配置 |
| 摄影排序、封面选择和列表卡片 | `layouts/photography/list.html` | 列表结构与资源处理 |
| 摄影文章、评论、Footer 和 PhotoSwipe | `layouts/photography/single.html` | 详情调用链 |
| 摄影列数、比例和卡片外观 | `assets/scss/partials/custom-components/_photography.scss` | 页面样式 |

维护时先按“内容或数据 → 模板 → 资源管道 → SCSS”的顺序定位问题：条目内容不对先检查 `content/` 或 `data/`，元素没有输出再检查 `layouts/`，图片没有生成检查资源路径与 Hugo Pipes，最后才检查 SCSS 的布局和显示规则。

---
