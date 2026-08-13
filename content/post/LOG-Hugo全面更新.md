---
title: "Hugo 弃用接口迁移记录"
description: "记录 Hugo 0.164.0 Extended 的弃用接口迁移、构建环境统一与严格验证。"
date: 2026-08-11T22:06:40+08:00
categories:
    - LOG
tags:
    - Hugo
    - Stack
    - GitHub Actions
    - 主题维护
math: false
comments: true
draft: false
---

本文只记录兼容性迁移，不重复日常主题配置。站点结构和部署背景见 [LOG-Hugo](../log-hugo/)，日常操作见 [LOG-使用手册](../log-使用手册/)。

## 迁移目标

Hugo 0.164.0 Extended 可以完成站点构建，但旧模板仍调用已标记为 deprecated 的语言、多站点和数据接口。弃用警告不影响当前输出，却会在未来版本移除旧接口时变成构建错误。本次迁移保持页面结构、内容和功能不变，只替换 API，并让本地与 GitHub Actions 使用同一个 Hugo 版本。

迁移时遵循三条原则：不隐藏警告、不降低 Hugo 版本、不用修改页面输出的方式绕过警告。严格构建使用：

```bash
hugo --gc --minify --panicOnWarning
```

## 构建基线

- Hugo：`0.164.0 Extended`
- GitHub Actions：`peaceiris/actions-hugo@v3`，`extended: true`
- Go：`1.25.5`
- Node.js：`24.12.0`
- Dart Sass：`1.97.1`

迁移前的基线构建可以生成页面，但会打印语言字段、站点集合、站点数据和语言对象相关警告。迁移完成后，严格构建退出码为 `0`，弃用警告和短代码警告均为 `0`。页面数量、资源数量以构建输出为准，不用手工修改统计数字。

## 语言配置迁移

Hugo 新版配置字段对应关系如下：

| 旧字段 | 新字段 |
| --- | --- |
| `languageCode` | `locale` |
| `languageName` | `label` |
| `languageDirection` | `direction` |

当前 `config/_default/languages.toml` 使用 `zh` 作为默认语言，`en` 保留配置但暂时 `disabled = true`。迁移后的结构示例：

```toml
[zh]
    locale = "zh-CN"
    label = "简体中文"
    direction = "ltr"

[en]
    locale = "en-US"
    label = "English"
    direction = "ltr"
    disabled = true
```

如果以后重新开放英文，只需移除 `disabled = true`，并补齐对应的英文内容和菜单；不要把新字段改回旧字段。

## 根目录模板迁移

根目录模板属于本站覆盖层，优先级高于 `themes/stack/`。本次涉及的关键文件如下。

### 统计页

`layouts/_default/stats.html` 中的站点集合从旧的 `.Sites` 改为全局 `hugo.Sites`：

```markdown
{{</* range hugo.Sites */>}}
```

这里的写法仅用于日志展示，实际模板中使用 Hugo 模板语法；日志中的示例必须使用 `{{</* ... */>}}` 转义，避免发布日志时被当作短代码执行。

### 文章详情、左侧栏和 Footer

以下文件将语言对象的旧字段换成新版字段：

- `layouts/_partials/article/components/details.html`
- `layouts/_partials/sidebar/left.html`
- `layouts/_partials/footer/custom.html`

对应替换为：

```text
.Language.LanguageName  ->  .Language.Label
.Language.Lang          ->  .Language.Name
```

这些替换只改变 API 访问方式，不改变界面文字、菜单顺序或 English 开关。

## 本地 Stack 主题迁移

本地化主题也必须同步迁移，否则根目录没有覆盖到的主题模板仍会发出警告。关键文件如下。

### `themes/stack/layouts/baseof.html`

```text
.Site.LanguageCode              ->  .Site.Language.Locale
.Language.LanguageDirection     ->  .Language.Direction
```

### `themes/stack/layouts/rss.xml`

```text
site.Language.LanguageCode  ->  site.Language.Locale
```

### 外部资源和 PhotoSwipe

主题模板中的全局数据访问从站点对象迁移到 Hugo 全局数据：

```text
.Site.Data.external...          ->  hugo.Data.external...
.Context.Site.Data.external...  ->  hugo.Data.external...
```

实际检查和修改的文件包括：

- `themes/stack/layouts/_partials/article/components/photoswipe.html`
- `themes/stack/layouts/_partials/helper/external.html`
- 主题中的语言后备模板（details、sidebar 等）

根目录仍然优先覆盖本站的文章详情、Footer、侧栏和首页；主题目录中的修改用于补齐没有被本站覆盖的调用点。

## GitHub Actions 版本统一

工作流 `.github/workflows/deploy.yml` 的 `HUGO_VERSION` 已统一为 `0.164.0`，并保留 Extended 构建。版本统一的目的有两个：

- 本地构建和 CI 使用同一套模板 API，避免“本地成功、CI 失败”。
- 主题 Sass、资源处理和 WebP 输出使用同一版本行为，减少部署后差异。

工作流仍使用 GitHub Actions 作为 Pages Source，推送 `main` 后由 `build` 成功才进入 `deploy`。本轮日志整理不修改工作流。

## 严格验证流程

排查弃用警告时按以下顺序进行：

1. 记录 Hugo 版本和 Extended 标识。
2. 运行普通构建，按警告中的文件和行号分组。
3. 先改配置字段，再改根目录覆盖模板，最后检查本地主题后备模板。
4. 搜索旧 API 的全部调用点，避免只修复第一处。
5. 使用 `--panicOnWarning` 重新构建；任何警告都视为未完成。
6. 对比页面、静态资源、图片衍生文件和关键链接。

升级 Hugo 时，先在本地复制工作区运行严格构建，再同步 GitHub Actions 版本。若出现新警告，先定位 API 迁移规则，不要用关闭警告或降低版本解决。

## 与其他日志的边界

- 部署、主题本地化和 Footer 等站点功能见 [LOG-Hugo](../log-hugo/)。
- 摄影、书与影和图片处理见 [LOG-栏目扩展](../log-栏目扩展/)。
- 配置、写作、预览和发布命令见 [LOG-使用手册](../log-使用手册/)。

本文件的章节标题采用语义化标题，不手工添加“一、二、三”等编号；文章目录编号由 Stack 主题负责显示。
