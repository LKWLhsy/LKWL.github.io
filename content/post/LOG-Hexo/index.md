---
title: 'LOG-Hexo'
date: 2025-03-16T17:26:51+08:00
slug: 'log-hexo'
tags:
  - 'Log'
image: 'donghu_piers.jpg'
toc: true
math: true
comments: true
draft: false
---
记录迁移博客与更换snail主题，以及进行个性化建设的日志.

<!--more-->

## 2025/11/7 迁移
将 blog 从原电脑 小新14pro 迁移到 thinkpad x1 carbon.
1. 将 hexo 博客文件复制过去
2. 在新电脑下载 git 与 nod.js（一路next即可，保持默认设置）
3. 在blog根目录打开 git bash 并执行：
```bash
$ git config --global user.name "Github用户名"
$ git config --global user.email "GitHub邮箱"
$ ssh-keygen -t rsa -C "Github邮箱"
```
随后即可在 C:\Users\hsy\.ssh\id.rsa.pub 文档中找到 'ssh-rsa  xxxxxx' ，全部复制下来，到 GitHub 的 SSH key 中添加公共密钥。

4. 最后进入git bash，执行：
```bash
$ npm install hexo-cli -g
$ npm install hexo-deployer-git --save
```
注意，对于目前使用的 snail 模板，以上操作默认安装了 hexo-4.7 版本，目前 Node.js 最新为 24版 ，需要下载 18.20.8 版本且将 hexo-util 卸载换为 hexo-util-2.6.1版:
```cmd
$ cd /d D:\hexo\LKWLhsyblog
$ rmdir /s /q node_modules
$ del package-lock.json
$ npm install hexo-util@2.6.1 --save
```
这样注意运行：
```bash
$ npm list --depth=0
```
检查一下负责公式的包是否为 hexo-renderer-kramed 与 hexo-renderer-mathjax（且不依赖原来默认的包），注意这样也要把 D:\hexo\LKWLhsyblog\node_modules\hexo-renderer-mathjax\mathjax.HTML 文件最后一行再改为 snail 文档中给的：
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.6/MathJax.js?config=TeX-MML-AM_CHTML"></script>
```
ps: 也可能不用将模组全部删除，可能只需要一开始就下载 Node.js 18.20.8 且把 hexo-util 换成 2.6.1版 的就可以了，不过我没试过。

## 2025/11/7 snail 主题设置
### 预览设置

```yml
#找到themes/snail/layout/index.ejs，有代码(注意区分有top与main两个部分的文章编辑)：
<div class="post-content-preview">
  <%- truncate(strip_html(post.content), {length: 200, omission: '...'}) %>...
</div>
#自动截取前200字符，超出直接用...省略了，不好看，改为：
<%- strip_html(post.excerpt || truncate(strip_html(post.content), {length: 200, omission: '...'}) ) %>
#将Preview与正文部分分开显示：进入 \themes\snail\layout\post.ejs，将：
<%- page.content %>
#改为
<%- page.more || page.content %>
```
### 浏览器标签页图标修改
```yml
#找到\themes\snail\layout\_partial\head.ejs开头的代码修改成自己的图片
<link rel="shortcut icon" href="<%= config.root %>img/me.jpg" />
```

### 解决无法正常 hexo d 
将 deploy 改成：
```yml
deploy:
  type: git
  repo: git@github.com:LKWLhsy/LKWLhsy.github.io.git  #不出现https://，不然无法链接发布到我的GitHub
  branch: main #符合自己的GitHub设置
```

## 第一次建设博客的记录
#### 2025/03/16
+ 搭建完成个人博客：LKWLhsy.github.io 。 
+ 下载主题：hingle、fluid、melody，目前使用 hingle，未来可能会尝试 Cards。  

#### 2025/03/17
+ 完善博客的网页图标以及个人头像
+ 接入 mathJax 拓展（安装了 hexo-filter-mathjax ），hingle 主题获得编译 \(LaTeX\) 公式的功能。
+ 解决的\(Latex\)公式换行问题;
+ 卸载了 hexo-renderer-prismjs ，安装了 prismjs 。 

#### 2025/03/18
+ 调整了文章预览页面最大显示行数，添加摘要部分，使其更加舒服。

#### 2025/03/19
+ 添加被Google、Microsoft搜索引擎检索功能；等待审核
  
#### 2025/03/20
+ Microsoft Bing 编制索引完成。
+ 添加 CC BY-NC-SA 4.0 授权协议

#### 2025/03/30
+ 添加了 Utterances 评论功能

#### 代办项
+ 完善文章格式
+ \(Latex\)公式手机浏览超出页面问题 

#### Note
1. 关于 \(Latex\) 公式问题（基于 GPT-4o 解决）  
网上诸多解决方法均没有用，于是求助与GPT进行解决。  
在 _config.yml 中添加代码解锁利用 mathjax 进行 Latex 公式编译功能
```yml
math:
  enable: true
  mathjax:
    enable: true
    src: "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"
    options:
      tex:
        inlineMath: [["$", "$"], ["\\(", "\\)"]]
        displayMath: [["$$", "$$"], ["\\[", "\\]"]]
        tags: "ams"  # 让 \tag{} 生效
        packages: { "[+]": ["AMSmath", "AMSsymbols"] }  
```

随后在切到主题目录下的路径：\themes\Hingle-main\layout\_partial 文件中的 head.ejs 文件末尾中添加相应代码调用（问 GPT 即可，css 格式我不知道怎么用 hexo 正常编译）。  

在此基础上 hexo 还存在对于公式太长原有 \(Latex\) 语法下行间公式使用 \\ 无法换行的问题，很可能是 Markdown 解析器把 \ 吞掉了；可以通过多添加一个 \ 进行转译：  
例如：
```markdown
$$
 \begin{aligned}
  V(x)=A + B x \\\\
  +C x^{2} + D x^{3}
 \end{aligned}
$$
```

代码块颜色问题暂时解决不了（其实代码显示上存在很多问题。。。
Hingle主题虽然好看简洁，但对于新手而言过于白板，作者也并没有对该主题的使用与修改做进阶的介绍，使新手在进行个性化时异常困难。并且初步使用感觉该主题对于数学排版的兼容性有些低，更加适合码农。


2. Hingle主题添加评论
   在 hingle/layout/post.ejs 文件中的 <%- page.content %> 之后添加代码启动评论功能。
```html
<div id="utterances-comments">
  <script src="https://utteranc.es/client.js"
          repo="你的GitHub用户名/你的GitHub评论仓库"
          issue-term="pathname"
          label="comment"
          theme="github-light"
          crossorigin="anonymous"
          async>
  </script>
</div>
```

## 参考文献
[1] [知乎](https://zhuanlan.zhihu.com/p/686890167)



