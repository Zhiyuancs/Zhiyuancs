title: Hexo-Hueman Insight-Search配置
author: Zhiyuan
tags:
  - Hexo
categories: []
date: 2019-09-09 15:23:00
---
博客中的搜索功能一直无法使用，使用
`npm install -S hexo-generator-json-content`
安装插件后，仍然无法使用。

通过阅读**hexo-generator-json-content**的源码配置文件后得知，还需要对hexo的config文件做相应配置：

```javascript
jsonContent:
  meta: true
  dafts: false
  file: content.json
  keywords: undefined
  dateFormat: undefined
  pages:
    title: true
    slug: true
    date: true
    updated: true
    comments: true
    path: true
    link: true
    permalink: true
    excerpt: true
    keywords: false
    text: true
    raw: false
    content: false
    author: true
  posts:
    title: true
    slug: true
    date: true
    updated: true
    comments: true
    path: true
    link: true
    permalink: true
    excerpt: true
    keywords: false
    text: true
    raw: false
    content: false
    author: true
    categories: true
    tags: true
```

