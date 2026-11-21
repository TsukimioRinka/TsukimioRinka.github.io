---
title: "用 Hugo 和 GitHub Pages 搭这个站"
date: 2026-11-21T10:00:00+08:00
draft: false
tags: ["hugo", "github-pages", "ci"]
categories: ["折腾"]
summary: "为什么选 Hugo，Cloudflare Pages 之外的免费托管，以及 Actions 部署 Pages 时踩到的几个坑。"
---

静态博客的方案翻来覆去就那几套。这次定下来的是 **Hugo + PaperMod + GitHub Actions + GitHub Pages**，理由和过程记一下。

## 为什么是 Hugo

| 方案 | 构建一千篇的耗时 | 依赖 |
|:--|:--|:--|
| Jekyll | 分钟级 | Ruby 工具链 |
| Hexo | 秒级到十几秒 | Node |
| Hugo | 亚秒级 | 单个二进制 |

Hugo 是 Go 写的单文件程序，`hugo` 一个可执行文件就能跑，CI 里也不用装 Node 或 Ruby。文章多了以后构建时间的差距会很明显——GitHub Pages 的 Jekyll 构建超过 10 分钟会直接超时。

## 为什么主题用 PaperMod

它只做一件事：把 Markdown 排成能读的单栏文章页，带深浅色切换、目录、代码复制、搜索。没有花哨的动效，改起来是纯模板，不需要懂某个前端框架。

## 一个必须注意的版本坑

PaperMod 的最新 release 是 **v8.0（2024 年 11 月）**，它声明的 Hugo 最低版本是 0.158，但实际上装不上——`layouts/partials/templates/opengraph.html` 里调用了 `partial "partials/templates/_funcs/get-page-images"`，而 Hugo 新版本改了模板目录布局（`layouts/partials` → `layouts/_partials`），这个前缀会直接报 `partial not found`。

Hugo 0.146 之后目录结构变了，PaperMod 的 master 分支跟进了，v8.0 的 tag 没跟。所以 submodule 要跟 master，而不是打 tag：

```bash
git submodule add -b master https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

代价是可复现性——master 随时会动。折中方案是记录当时的确切 commit，需要升级时再 `git submodule update --remote`。

## Actions 部署 Pages 的两个坑

**第一，`CNAME` 文件没有用。** 网上大量教程说「在仓库根目录或 public 里放一个 CNAME 文件写域名」，那是 `Deploy from a branch` 模式的做法。一旦切换到 `GitHub Actions` 作为发布源，CNAME 文件会被忽略，自定义域名只认仓库 Settings 里存的那份。

**第二，`baseURL` 别写死。** 如果 `hugo.yaml` 里把 `baseURL` 写死成最终域名，那么在绑定域名之前、或者拿 `github.io` 预览时，所有绝对链接都会指向一个还不存在的地址。用 `actions/configure-pages` 输出的 `base_url` 覆盖：

```yaml
- uses: actions/configure-pages@v5
  id: pages
- run: hugo --gc --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
```

这样不管当前是 `*.github.io` 还是已经绑了自定义域名，构建出来的链接都是对的。

## 工作流

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
```

`id-token: write` 是 `deploy-pages` 要求的（OIDC 验证），漏了会报权限错误。`concurrency` 不加的话，连续两次 push 可能让两次部署互相覆盖。

## 写文章

```bash
hugo new posts/标题.md     # 从 archetypes/default.md 生成骨架
hugo server -D             # 本地预览，含草稿
```

推 `main` 就自动构建。

---

剩下的等踩到再补。
