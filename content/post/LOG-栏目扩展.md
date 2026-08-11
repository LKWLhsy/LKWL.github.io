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
---

## 更新目标

本次更新为博客增加两个长期栏目：

1. **摄影**：按期发布摄影作品，每一期拥有独立正文、封面、照片和图注，并复用 Stack 自带的 PhotoSwipe。
2. **书与影**：优先展示“个人 Top 10 电影”等长期榜单，再记录当年的观影与阅读。

本轮没有启用 English，没有引入新的 JavaScript、npm、Go 或 Hugo Module 依赖，也没有修改 GitHub Actions。

---

## 一、菜单结构

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

## 二、摄影栏目的内容结构

摄影栏目使用 Hugo Page Bundle。栏目入口是 branch bundle，每一期是 leaf bundle：

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

示例期：

```text
content/photography/issue-000/index.md
```

第 00 期不复制图片，而是读取现有全局资源：

```yaml
demoImage: "img/default-cover.jpg"
```

这样不会在 Git 仓库中重复保存当前约 8 MB 的默认图片。

---

## 三、创建正式摄影期刊

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

## 四、相机照片压缩

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

### 可选 ImageMagick 命令

删除 EXIF 和 GPS：

```bash
magick input.jpg \
  -auto-orient \
  -resize "3000x3000>" \
  -colorspace sRGB \
  -strip \
  -quality 82 \
  output.jpg
```

如需保留 EXIF，则去掉 `-strip`。本项目没有安装或依赖 ImageMagick，该命令仅作为可选的本地预处理方法。

推荐体积：

```text
网页母版：1–3 MB
卡片封面：100–300 KB
灯箱大图：400 KB–1.5 MB
```

Git LFS 不适用于 GitHub Pages，因此原片应保存在本地硬盘、备份盘或对象存储中。[GitHub LFS 说明](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-git-large-file-storage)。

---

## 五、摄影图片处理代码

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

Hugo 会在构建时缓存处理结果。[Hugo 图片处理](https://gohugo.io/content-management/image-processing/)。

---

## 六、摄影模板与样式

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

## 七、书与影栏目

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

## 八、书与影数据

数据文件：

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

海报保存为：

```text
assets/img/books-and-film/movies/interstellar.jpg
```

详情信息和海报来自派拉蒙官方页面：

- [影片详情](https://www.paramountpictures.com/movies/interstellar)
- [官方海报资源](https://public-website-assets.paramountpictures.com/paramount2025/s3fs-public/styles/poster_medium/public/intersteller_en_dvd_800x1200.jpg?itok=YxrRaJN2)

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

## 九、书与影模板与样式

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

## 十、构建与发布

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

本次执行阶段没有提交、推送或部署。

---

## 十一、验证结果

本次执行完成了严格构建、生成文件检查和真实浏览器检查：

- 使用 Hugo 0.164.0 Extended 执行 `hugo --gc --minify --panicOnWarning`，退出码为 0，警告数为 0。
- 共生成 43 个页面、15 个别名页面，并处理 11 张图片。
- 摄影列表页、摄影第 00 期和书与影页均成功生成。
- 摄影列表封面宽度为 720 像素，正文预览图宽度为 1200 像素；二者均为 Hugo 从默认图片生成的 WebP，而不是直接输出原始大图。
- 《星际穿越》本地海报被处理为宽度 600 像素的 WebP，生成文件约 126 KB。
- 在 1440 × 1000 桌面视口和 390 × 844 手机视口中检查三个页面，均返回 HTTP 200，图片均完整加载，且没有横向溢出。
- 摄影详情页的 PhotoSwipe 灯箱可正常打开。
- 书与影页按预期先显示“个人 Top 10 电影”，并在下方显示 2026 年空状态。
- 浏览器检查未发现本站资源请求失败。
- `git diff --check` 通过，且本次没有修改 `public/` 或 `resources/`。
- 本次没有执行 Git 提交、推送或 GitHub Pages 部署。

> 相机原片用于保存作品，网页母版用于发布作品，响应式衍生图用于向访客展示作品。三者不应混为同一个文件。

---

## 十二、书与影索引与海报网格调整

### 调整原因

最初的榜单网格固定为单列：

```scss
.ranking-grid {
    grid-template-columns: 1fr;
}
```

每个条目又使用横向“海报 + 详情”布局，因此只有《星际穿越》一个条目时，卡片会占满整个内容宽度，看起来更像文章头图而不是影片海报。

### 分类索引

修改文件：

```text
layouts/page/books-and-film.html
```

新增两个并列标签：

```html
<nav class="books-and-film-tabs" role="tablist" aria-label="书与影分类">
    <button type="button" role="tab" data-books-tab="top10">Top 10</button>
    <button type="button" role="tab" data-books-tab="2026">2026</button>
</nav>
```

脚本会同步以下状态：

- `aria-selected`：供辅助技术识别当前标签。
- `tabindex`：只让当前标签进入正常键盘顺序。
- `hidden`：隐藏未选择的内容面板。
- URL hash：记录 `#top10` 或 `#2026`。
- 左右方向键：在两个标签之间切换。

脚本没有第三方依赖，也不会访问外部服务。

### 五列海报网格

修改样式：

```text
assets/scss/partials/custom-components/_books-and-film.scss
```

响应式列数：

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

海报区域统一使用：

```scss
aspect-ratio: 2 / 3;
object-fit: cover;
```

Hugo 为每张源海报生成：

```go-html-template
{{ $preview := .Fill "520x780 webp q82" }}
```

页面实际加载本地 WebP，而不是直接加载外部海报。

### 《蜘蛛侠：崭新之日》演示条目

修改数据：

```text
data/books-and-film/top10-movies.toml
```

新增本地海报：

```text
assets/img/books-and-film/movies/spider-man-brand-new-day.jpg
```

条目使用 Sony 官方影片页提供的资料，导演为 Destin Daniel Cretton，年份为 2026。由于本站尚未形成实际观影评价，条目明确填写：

```toml
status = "待上映 / 评分待定"
note   = "用于展示榜单海报网格的待上映示例，名次并非最终评价。"
```

来源：

- [Sony 官方影片页](https://www.sonypictures.com/movies/spidermanbrandnewday)
- [Sony 官方海报](https://www.sonypictures.com/sites/default/files/styles/max_860x460/public/title-key-art/spidermanbrandnewday_onesheet_1400x2100.jpg?itok=Nh6VAAh-)

下载后的 JPEG 为 69,023 字节，SHA-256：

```text
AC7C3710284AFEED0424F3795151184F7CA756115D0722BB5E591D12692C9E59
```

---

## 十三、摄影页布局与卡片比例

### 取消右侧栏

摄影列表和详情模板原先都定义了 `right-sidebar`。本次从以下文件删除该定义：

```text
layouts/photography/list.html
layouts/photography/single.html
```

摄影列表现在使用完整主内容宽度；详情页保持单栏文章和 PhotoSwipe 图集。

### 与书与影统一外壳

摄影标题和期刊网格现在共同放入：

```html
<article class="photography-shell card">
```

标题区使用底部分隔线，期刊卡片采用统一边框、圆角、阴影、内边距和等高内容区域。

响应式列数：

```text
小于 768 px：1 列
768–1199 px：2 列
1200 px 及以上：3 列
```

### 自定义摄影卡片形状

配置位置：

```text
content/photography/_index.zh.md
```

默认配置：

```yaml
cardAspect: "landscape"
```

可用值：

```text
landscape  3:2
wide       16:9
square     1:1
portrait   2:3
```

模板会根据设置选择匹配的 Hugo 图片处理尺寸：

```go-html-template
landscape → 720x480 WebP
wide      → 720x405 WebP
square    → 720x720 WebP
portrait  → 600x900 WebP
```

如果填写了不支持的值，模板会回退到 `landscape`。比例在栏目级统一设置，避免同一行的卡片高低不齐。

---

## 十四、本轮验证

本轮使用 Hugo 0.164.0 Extended 严格构建，并使用本机 Chrome 分别在 1440 × 1000 和 390 × 844 视口进行真实页面检查。

构建结果：

```text
Pages:             43
Processed images:  12
Aliases:           15
Exit code:          0
Warnings:           0
```

书与影检查：

- 宽屏为五列网格，窄屏为两列网格。
- Top 10 当前包含两张紧凑海报卡片。
- 两张海报均由 Hugo 生成宽度 520 像素的本地 WebP。
- Top 10 与 2026 可以点击切换。
- 2026 面板可以显示空状态。
- `#2026` 刷新后仍会恢复 2026 面板。
- `aria-selected` 与隐藏面板状态同步。

摄影检查：

- 宽屏为三列，中等屏幕规则为两列，窄屏为一列。
- 默认 `landscape` 卡片的浏览器实测宽高比为 1.50，即 3:2。
- 列表页和详情页均没有可见右侧栏。
- 列表封面由 Hugo 生成本地 WebP。
- 详情图完整加载，PhotoSwipe 灯箱能够打开。

通用检查：

- 三个被检查页面在两种视口下均返回 HTTP 200。
- 没有横向溢出。
- 没有本站资源请求失败。
- 没有安装新的浏览器、JavaScript 包或其他依赖。
- 本轮没有修改 `public/`、`resources/`、GitHub Actions 或远程仓库状态。
