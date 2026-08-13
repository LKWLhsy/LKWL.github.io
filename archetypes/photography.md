---
title: "{{ replace .File.ContentBaseName `-` ` ` | title }}"
description: ""
date: {{ .Date }}
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

在这里填写本期摄影说明。

{{</* photo-gallery */>}}

如果需要把照片分成多组，并在组间插入文字，可以指定文件：

{{</* photo-gallery files="photos/01.jpg,photos/02.jpg" */>}}

组间写普通 Markdown 文字后，再调用下一组：

{{</* photo-gallery files="photos/03.jpg,photos/04.jpg" */>}}
