# GitHub 个人主页 README 改版设计

日期：2026-10-05
仓库：tsix2019/tsix2019

## 目标

把 2023 年的旧主页翻新成「清爽卡片风」，中文为主，修掉已经失效的组件，补上近期项目，并自动适配 GitHub 深色/浅色主题。

## 现状问题

- B站、酷安粉丝数徽章失效（api.swo.moe 对这两个平台返回 `failed: true`）
- metrics.lecoq.io 卡片返回 500，主页显示裂图
- 内容停在 2023 年，没有 safeIP 等近期项目
- 「framel」拼写错误；邮箱链接指向 GitHub 主页而不是 `mailto:`
- Gitee、T00111100 不再需要展示

## 可靠性原则

- 静态资源（技术栈图标）放进仓库，零外部依赖
- 徽章用 shields.io 静态徽章，不再显示粉丝数
- 统计类卡片用当前验证可用的公共服务；挂了只影响单张卡片
- 不使用已失效的服务：metrics.lecoq.io（500）、github-profile-trophy（402）、github-readme-activity-graph（402）

## 页面结构（从上到下）

### 1. 头部（居中）

- 标题：`Hi, I'm Tsix 👋`
- 打字动画（readme-typing-svg，字体 Noto Sans SC，颜色 `#2F81F7`，深浅色下都看得清）：
  前端开发工程师 / 主力 JavaScript，偶尔写 Go / 周末钓鱼佬 🎣
- 徽章行（shields.io，`for-the-badge` 样式）：
  - 哔哩哔哩 → https://space.bilibili.com/536466393
  - 酷安（沿用旧 README 里的白色 logo）→ https://www.coolapk.com/u/3406488
  - 掘金 → https://juejin.cn/user/870468941779726
  - Email → mailto:tsix2019@gmail.com
  - 访问量（komarev.com，同一样式）。标签用 `Views`：komarev 按拉丁字符算宽度，中文「访问量」会被截断

### 2. 关于我

- 📍 安徽 · 芜湖
- 💼 前端开发，主力 JavaScript，偶尔写 Go
- 🔭 最近在做 safeIP、dicar2
- ❤️ 🚴 骑行 · 🎣 钓鱼 · 🍵 喝茶 · 📈 炒股

### 3. 技术栈

一排图标（48px）：Vue · Nuxt · React · Node.js · Electron · Go · Gin · Java · Kotlin

- 图标来自 tandpfun/skill-icons（MIT），下载到 `icons/`
- 有深浅两版的图标用 `<picture>` 按 `prefers-color-scheme` 切换
- Gin 没有现成图标：自制 SVG，用 skill-icons 同款圆角底色（深 `#242938` / 浅 `#F4F2ED`），把原 `Gin.png` 以 data URI 嵌进去，风格统一
- 删除旧 PNG（Electron、Nodejs、React、Vue、Webpack、jQuery、Gin），git 历史可找回

### 4. 精选项目

github-readme-stats pin 卡片两张并排：safeIP、dicar2，点击跳转到仓库。

- 不用 `<picture>`：`<picture>` 套在 `<a>` 里时，GitHub 会给 `<img>` 再包一层指向图片的链接，嵌套链接被浏览器拆开，`<img>` 掉出 `<picture>`，结果深色版失效、点击打开的是图片
- 改用一张深浅色通用的透明卡片：`bg_color=00000000`、标题/图标 `2F81F7`、正文 `768390`、边框 `76839066`（半透明，浅色下是浅灰、深色下是深灰）

### 5. GitHub 数据

- 个人概览卡（github-profile-summary-cards `profile-details`，含贡献曲线）
- 统计卡 + 语言占比卡并排（github-readme-stats）
- 写代码时间分布卡（github-profile-summary-cards `productive-time`，`utcOffset=8`）
- 原方案中的「连续贡献天数」卡取消：过去一年只有 8 天有贡献，会显示 0 天

这些卡片不套链接，可以用 `<picture>` 提供深浅两套主题：
github-readme-stats 用 `default` / `github_dark`，profile-summary-cards 用 `github` / `github_dark`。

### 6. 贪吃蛇（页面最底部，紧跟 GitHub 数据）

- 新增 `.github/workflows/snake.yml`：`Platane/snk/svg-only@v3` 生成 `github-snake.svg` 和 `github-snake-dark.svg`（`palette=github-dark`），用 `crazy-max/ghaction-github-pages@v5` 推到 `output` 分支
- 触发方式：每天 UTC 0 点（北京时间 8 点）、push 到 main、手动触发
- 只用仓库自带的 `GITHUB_TOKEN`（job 级 `contents: write`），不需要配置 PAT
- README 引用 `https://raw.githubusercontent.com/tsix2019/tsix2019/output/github-snake(-dark).svg`
- 第一次 workflow 跑完之前图片是 404；贡献少，蛇可吃的格子不多

## 改动文件

- `README.md`：重写
- `icons/`：换成 SVG 图标
- `.github/workflows/snake.yml`：新增

## 验证

- 用 GitHub Markdown API（`POST /markdown`，`mode=markdown`，和 README 文件的渲染方式一致；`gfm` 模式会把换行变成 `<br>`）渲染 README，在浏览器面板里分别检查浅色和深色效果
- 本地预览没有 GitHub 的 `<themed-picture>` 脚本，深色检查时用一小段脚本模拟它的切换逻辑
- 逐个请求 README 中的外部图片 URL，确认都返回 200 且是 SVG
- 本地只做 commit，push 前先征求同意

## 修订：改为全英文（2026-10-05）

参考 anuraghazra、DenverCoder1、rahuldkjain 等热门主页的写法：一两句自我介绍 + 少量要点，图标和卡片已经展示的信息不再用文字重复。

- README 和 workflow 注释全部改为英文；徽章标签改为 Bilibili / Coolapk / Juejin / Email
- 打字动画改用 Fira Code：Front-end Developer / Mostly JavaScript, some Go / Based in Wuhu, China
- 去掉「关于我」标题和重复的要点，只保留两条：在做的项目、业余爱好
- 小节标题去掉 emoji：Tech Stack / Featured Projects / GitHub Stats
- 项目卡片里的描述来自各仓库自己的 GitHub 描述，目前是中英双语，README 无法覆盖
