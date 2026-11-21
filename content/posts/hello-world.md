---
title: "第一篇"
date: 2026-11-21T00:00:00+08:00
draft: false
tags: ["meta"]
categories: ["Meta"]
summary: "站点跑起来了，先说清楚这里打算放什么。"
---

这个站挂在 GitHub Pages 上，域名是 `body.is-a.dev`。

## 写一篇新文章

```bash
hugo new posts/我的文章.md
```

然后在 `content/posts/` 下编辑，推上去就会自动构建。

## 本地预览

```bash
hugo server -D
```

打开 <http://localhost:1313>，改文件会实时刷新。
