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
