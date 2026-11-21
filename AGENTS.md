# Hugo 博客美化规范

本仓库是 Hugo 0.170.1 extended + PaperMod（git submodule，跟踪 master）的静态博客。
任何 AI / 人工修改都必须遵守以下约束。

## 1. 严禁直接修改 `themes/` 目录下的任何文件

必须使用 Hugo 的 Lookup Order 覆盖机制：

- 修改模板：先将 `themes/PaperMod/layouts/...` 复制到项目根目录对应的 `layouts/...` 再编辑。
  注意 Hugo 0.146+ 的模板目录是 `layouts/_partials` 和 `layouts/_shortcodes`，
  不是老教程里的 `layouts/partials`，放错位置不会报错，只是不生效。
- 修改样式：只改 `assets/css/extended/*.css`（PaperMod 按文件名排序自动合并），
  配色 / 字体只改 `01-tokens.css`，且必须同步修改文件末尾的
  `--theme` / `--entry` / `--primary` 等变量桥接，否则主题内置组件会停在旧色板。

## 2. 保持零前端框架依赖

- 只使用原生 HTML、现代 CSS（Grid / Flex / CSS Variables）。
- 严禁引入 React / Vue / Webpack / Vite / Tailwind / PostCSS / npm 构建链。
- 零外部运行时资源：不许 Google Fonts / jsDelivr / unpkg / 任何 CDN / 外链图片。
  字体只用系统字体栈（`--serif` / `--sans` / `--mono`），图标用内联 SVG。

## 3. Go Template 语法检查

修改 `.html` 模板时注意作用域（`.` 与根上下文 `$` 的区别），改完后必须确认
`hugo --gc --minify` exit 0、零 ERROR。

## 4. 设计语言红线（「星野手帖」）

- 深色 = 夜空（深靛 `--paper: #0a0e1c`），浅色 = 黎明（银灰蓝 `#edf0f6`），不许回到米白暖纸。
- 晨光金 `--accent` 是主强调色，只用于极小面积；唯一的红色是首页印章（`--seal`），
  不许再把红色扩散到别处。
- 星野层在 `03-sky.css` + `baseof.html` 的 `.sky`：纯 CSS、零图片、零 JS。
  只在深色模式可见（浅色是白昼）；`prefers-reduced-motion` 下流星隐藏、星星静止。
- 渐变只允许用于星云辉光与流星尾迹这类「发光体」，禁止用在按钮、卡片、文字上。
- 零卡片、零阴影，圆角 0–2px。
- 深色选择器写 `:root[data-theme="dark"]`（PaperMod 的 JS 在 `<html data-theme>` 上切换）。
- 可读文字字号下限 11px（0.7rem）；移动端正文不低于 16px；可点区域 ≥44px。
- 深浅两模式正文对比度都必须 ≥4.5:1。
- `prefers-reduced-motion` 下所有动效必须停用或转为静态。
- RSS 是明确移除的（`disableKinds: ["RSS"]`），不要加回来。
- 路由固定：`/ /posts/ /archives/ /tags/ /categories/ /search/ /about/`，不加新路由。
- 不要写死 `baseURL`（CI 用 actions/configure-pages 注入）。

## 5. 视觉自检闭环

每次样式改动完成后：

```bash
~/.local/bin/hugo --gc --minify --destination /tmp/out   # exit 0，零 ERROR
grep -rE "https?://(fonts|cdn|unpkg|jsdelivr)" /tmp/out  # 必须无输出
grep -rhoE "font-size: 0\.[0-6][0-9]*rem" assets/css/extended/*.css  # 必须无输出
```

并用 Playwright MCP 访问 `http://localhost:1313`，检查首页、文章页
（含代码块、表格、引用块）在 1440px / 390px、浅色 / 深色下的渲染效果，
确认无横向滚动后再交付。

## 6. Git 纪律

不要 `git commit`、`git checkout`、切分支、动 submodule。改完留在工作区，由人审查。
