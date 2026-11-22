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
- 交互增强只允许小型内联原生 JS，不得阻塞渲染，且必须服从动效开关（见第 4 节）。现有几处：
  - `extend_head.html`：动效门闩（首帧前给 `<html>` 加 `.js-motion`，需要安静时加 `.calm`，
    并提供 `window.bodyMotion`）；首页还有启动页门闩（`.splash-open`）。
  - `nightsky.html`（baseof 里紧跟 `.sky`）：画布星空库 `window.NightSky` + 全站背景星空。
  - `extend_footer.html`：顶栏动效开关、鼠标星轨、点击绽放、文章页阅读彗星、表格滚动包装。
  - `starsea-script.html`：启动页（草、光点升空）与首页星海（时间之河、旅人）。

## 3. Go Template 语法检查

修改 `.html` 模板时注意作用域（`.` 与根上下文 `$` 的区别），改完后必须确认
`hugo --gc --minify` exit 0、零 ERROR。

## 4. 设计语言红线（「星野手帖」）

- 只有夜空一种配色（深靛 `--paper: #0a0e1c`），没有浅色主题、没有主题切换。
  色板写在 `01-tokens.css` 的 `:root, :root[data-theme="dark"]`——必须带上后一个选择器，
  否则 PaperMod 自己的 `[data-theme="dark"]` 灰色变量会以更高优先级盖过来。
- 晨光金 `--accent` 是主强调色，只用于极小面积；唯一的红色是首页印章（`--seal`），
  不许再把红色扩散到别处。
- 星空：`.sky` 里的 CSS 星点和三颗 CSS 流星只是无 JS 兜底；有 JS 时由 `nightsky.html` 的两块画布接管
  （静止星场 + 淡银河带滚动视差；闪烁星 + 随机流星，平均约 2 秒一颗，偶尔成串、偶尔一颗金色火流星）。
  文章页的流星亮度降到 0.8，免得抢正文。
- 鼠标星轨要淡而稀：一根很细的光线 + 稀疏星尘（约每 30px 一粒）。点击任意处触发「星空绽放」
  （闪光、两圈光环、六道光芒、向外绽开的四角星，再落下几粒余烬），触屏也有。
- 首页先是氛围，其次才是博客：
  - 启动页 `_partials/splash.html` + `04-splash.css`：夜里的草坡，一个人背对我们仰望星空。
    每个标签页会话只出现一次（`sessionStorage` 键 `body:splash`），`/page/N/` 与无 JS 时不出现。
    点击 / 任意键 / 滚轮 / 滑动进入。近坡是抛物线（见 splash.html 顶部注释），改坡形要同步脚本里的 `hill()`。
  - 星海 `_partials/starsea.html` + `05-starsea.css`：每篇文章一颗星（最多 24 颗，更早的指向归档），
    沿时间之河从过去（左下）流向今天（右上）；间距按间隔天数取对数。河的形状只在
    `_partials/starsea-pos.html` 定义，星星位置与画布上的河都从这里取值。
- 动效开关：顶栏四角星按钮（原主题切换的位置），选择存在 `localStorage` 键 `body:motion`（on/off），
  没选过就跟随系统 `prefers-reduced-motion`。CSS 里安静态一律写 `html.calm …`；
  `@media (prefers-reduced-motion)` 只允许写成 `html:not(.js-motion) …` 作为无 JS 兜底。
  安静时：没有任何循环动画，流星定格成静止光痕，草不动，旅人淡入淡出代替飞行，
  启动页离场是纯淡出，星轨与绽放只原地淡出。
- 渐变只允许用于星云辉光与流星尾迹这类「发光体」，禁止用在按钮、卡片、文字上。
- 零卡片、零阴影，圆角 0–2px。
- 可读文字字号下限 11px（0.7rem）；移动端正文不低于 16px；可点区域 ≥44px。
- 正文对比度必须 ≥4.5:1。
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
（含代码块、表格、引用块）在 1440px / 390px 下的渲染效果（动效开关开 / 关各看一次），
确认无横向滚动后再交付。

## 6. Git 纪律

不要 `git commit`、`git checkout`、切分支、动 submodule。改完留在工作区，由人审查。
