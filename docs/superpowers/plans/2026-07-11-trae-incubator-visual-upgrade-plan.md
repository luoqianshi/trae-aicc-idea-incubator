# TRAE 灵感孵化舱 · 视觉与交互升级实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use `executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 按照已确认的设计文档，对 `index.html` 进行单文件改造，实现电影感暗色视觉、高级感 Hero、任务追踪器、对话气泡与选择卡片式游戏化引导。

**Architecture：** 继续沿用单文件 HTML + CSS + JS 架构；通过 CSS 变量统一视觉系统；用 IntersectionObserver 驱动滚动动画与任务追踪；用 localStorage 持久化选择卡片的完成状态；所有改动集中在 `index.html`，不引入构建工具。

**Tech Stack：** HTML5、CSS3（Custom Properties、Grid/Flex、Backdrop-filter、@keyframes）、原生 JS（Canvas、IntersectionObserver、localStorage、Clipboard API）、Google Fonts（Bricolage Grotesque）。

---

## 文件结构

- **修改：** `index.html` —— 唯一目标文件，包含全部样式、结构与脚本。
- **保留：** `assets/` 目录下所有图片与 logo 资源。
- **新增设计稿：** `docs/superpowers/specs/2026-07-11-trae-idea-incubator-visual-upgrade-design.md`（已存在，仅作为参考）。

---

## Task 1：引入 Google Fonts 并更新全局视觉系统

**Files：**
- Modify: `index.html` `<head>` 与 `:root` CSS 区域。

- [ ] **Step 1：在 `<head>` 末尾添加 Bricolage Grotesque 字体链接**

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,400;12..96,600;12..96,700;12..96,800&family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
```

- [ ] **Step 2：替换 `:root` 变量为新的视觉系统**

```css
:root {
  --bg-base:        #050505;
  --bg-card:        rgba(255, 255, 255, 0.035);
  --bg-card-hover:  rgba(255, 255, 255, 0.055);
  --bg-tag:         rgba(255, 255, 255, 0.06);
  --border:         rgba(255, 255, 255, 0.08);
  --border-strong:  rgba(255, 255, 255, 0.14);

  --accent:         #22c55e;
  --accent-deep:    #16a34a;
  --accent-glow:    rgba(34, 197, 94, 0.35);
  --accent-soft:    rgba(34, 197, 94, 0.12);
  --accent-highlight:#a3e635;

  --gold:           #fbbf24;
  --silver:         #a1a1aa;
  --bronze:         #b45309;

  --text-primary:   #ffffff;
  --text-secondary: #a1a1aa;
  --text-tertiary:  #d4d4d8;
  --text-muted:     #71717a;

  --container:      1280px;
  --radius-card:    20px;
  --radius-pill:    9999px;
  --radius-input:   10px;

  --font-display:   "Bricolage Grotesque", "Inter", "SF Pro Display", -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
  --font-sans:      "Inter", "SF Pro Display", -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", "Segoe UI", sans-serif;
  --font-mono:      "JetBrains Mono", "SF Mono", Menlo, Consolas, monospace;
}
```

- [ ] **Step 3：给 `body` 添加极光背景层**

在 `#particle-canvas` 之后、`<nav>` 之前插入一层静态渐变：

```html
<div class="aurora-bg" aria-hidden="true"></div>
```

并在 CSS 中添加：

```css
.aurora-bg {
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  background:
    radial-gradient(circle at 20% 20%, rgba(34, 197, 94, 0.18), transparent 45%),
    radial-gradient(circle at 80% 80%, rgba(34, 197, 94, 0.08), transparent 40%),
    radial-gradient(circle at 50% 120%, rgba(34, 197, 94, 0.10), transparent 50%);
  filter: blur(60px);
  opacity: 0.7;
}
```

- [ ] **Step 4：验证**

打开 `index.html`，检查：
- 网络面板已加载 Google Fonts。
- `:root` 变量值为新值。
- 页面背景为极黑并带有绿色光晕。

---

## Task 2：升级粒子画布

**Files：**
- Modify: `index.html` 中粒子 Canvas 脚本区域。

- [ ] **Step 1：替换粒子参数与绘制逻辑**

在现有粒子脚本中，将以下常量与绘制代码替换：

```js
const PARTICLE_COUNT = Math.min(60, Math.floor(window.innerWidth / 24));
const CONNECT_DIST = 160;
const MOUSE_DIST = 220;
```

并将循环内的 `ctx.fillStyle` 与连线颜色从固定 `rgba(34, 197, 94, ...)` 改为使用 `p.opacity`：

```js
ctx.beginPath();
ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2);
ctx.fillStyle = `rgba(34, 197, 94, ${p.opacity})`;
ctx.fill();
```

连线 alpha 改为：

```js
const alpha = (1 - d / CONNECT_DIST) * 0.12;
ctx.strokeStyle = `rgba(34, 197, 94, ${alpha})`;
ctx.lineWidth = 0.5;
```

鼠标排斥/吸引改为：

```js
if (dist < MOUSE_DIST) {
  const force = (MOUSE_DIST - dist) / MOUSE_DIST;
  p.x -= dx * force * 0.008;
  p.y -= dy * force * 0.008;
}
```

- [ ] **Step 2：验证**

- 粒子数量更少、更柔和。
- 鼠标靠近时粒子轻微避开。
- 连线更细更淡。

---

## Task 3：改造导航栏

**Files：**
- Modify: `index.html` 中 `.navbar` 相关 CSS 与 HTML。

- [ ] **Step 1：更新 `.navbar` 样式**

```css
.navbar {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 100;
  height: 68px;
  padding: 0 32px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(5, 5, 5, 0.55);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-bottom: 1px solid var(--border);
  transition: background 0.3s ease, border-color 0.3s ease;
}
.navbar.scrolled {
  background: rgba(5, 5, 5, 0.88);
  border-color: var(--border-strong);
}
```

- [ ] **Step 2：把 logo 图片切换为 `assets/trae_dark.png`**

```html
<div class="logo-mark"><img src="assets/trae_dark.png" alt="TRAE"></div>
```

并调整 `.logo-mark` 尺寸：

```css
.logo-mark {
  width: 30px; height: 30px;
  display: grid; place-items: center;
}
.logo-mark img { width: 100%; height: 100%; object-fit: contain; }
```

- [ ] **Step 3：验证**

- 导航栏有磨砂玻璃效果。
- 滚动后背景变深。
- logo 显示为新版深色图标。

---

## Task 4：重新设计 Hero

**Files：**
- Modify: `index.html` 中 `.hero` CSS 与 HTML。

- [ ] **Step 1：更新 `.hero` 容器与内容最大宽度**

```css
.hero {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  padding-top: 68px;
  text-align: center;
}
.hero-inner {
  max-width: 820px;
  margin: 0 auto;
}
```

- [ ] **Step 2：把 Hero HTML 改为 4 个 CTA**

```html
<header class="hero">
  <div class="container">
    <div class="hero-inner">
      <div class="pill-tag">
        <span class="dot"></span>
        <span>TRAE AI 创造力大赛 · 社会服务赛道</span>
      </div>
      <h1>
        一颗灵感种子，<br>
        如何在 <span class="accent">TRAE</span> 中<br>
        长成完整作品？
      </h1>
      <p>从灵光一闪到报名提交，四步孵化，让每一个想法都有机会绽放。</p>
      <div class="hero-cta">
        <button class="btn btn-primary btn-lg" onclick="document.getElementById('sow').scrollIntoView({behavior:'smooth'})">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polygon points="5 3 19 12 5 21 5 3"/></svg>
          开始孵化之旅
        </button>
        <a href="https://solo.trae.cn/" target="_blank" class="btn btn-secondary btn-lg">TRAE Work Go Go Go!</a>
        <a href="https://luoqianshi.github.io/TRAE-AI-Creativity-Competition-Idea-Hall/" target="_blank" class="btn btn-secondary btn-lg">Idea Hall</a>
        <a href="https://luoqianshi.github.io/trae-demo-wall/" target="_blank" class="btn btn-secondary btn-lg">Demo Wall</a>
      </div>
    </div>
  </div>
  <div class="scroll-hint">
    <span>SCROLL</span>
    <div class="line"></div>
  </div>
</header>
```

- [ ] **Step 3：添加 Hero 入场动画类**

在 CSS 中添加：

```css
.hero .pill-tag,
.hero h1,
.hero p,
.hero-cta .btn {
  opacity: 0;
  transform: translateY(20px);
  animation: heroIn 0.8s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}
.hero h1 { animation-delay: 0.1s; }
.hero p  { animation-delay: 0.2s; }
.hero-cta .btn:nth-child(1) { animation-delay: 0.3s; }
.hero-cta .btn:nth-child(2) { animation-delay: 0.38s; }
.hero-cta .btn:nth-child(3) { animation-delay: 0.46s; }
.hero-cta .btn:nth-child(4) { animation-delay: 0.54s; }

@keyframes heroIn {
  to { opacity: 1; transform: translateY(0); }
}
```

- [ ] **Step 4：验证**

- Hero 内容居中。
- 四个 CTA 按钮按主次排列，换行自然。
- 刷新页面时标题、描述、按钮依次入场。

---

## Task 5：构建任务追踪器

**Files：**
- Modify: `index.html` HTML 结构与 CSS、JS 滚动监听。

- [ ] **Step 1：在 `<main>` 外层包裹 `.page-layout`，并添加侧边追踪器**

把：

```html
<main>
```

改为：

```html
<div class="page-layout">
  <aside class="mission-tracker" aria-label="任务进度">
    <div class="tracker-title">Mission Log</div>
    <ul class="tracker-list">
      <li><a href="#sow" data-target="sow" class="active"><span class="tracker-num">01</span><span class="tracker-label">播种</span></a></li>
      <li><a href="#water" data-target="water"><span class="tracker-num">02</span><span class="tracker-label">浇灌</span></a></li>
      <li><a href="#prune" data-target="prune"><span class="tracker-num">03</span><span class="tracker-label">修剪</span></a></li>
      <li><a href="#bloom" data-target="bloom"><span class="tracker-num">04</span><span class="tracker-label">绽放</span></a></li>
    </ul>
  </aside>

  <main>
```

并在 `</main>` 后补 `</div>`。

- [ ] **Step 2：添加移动端顶部进度条**

在 `<div class="page-layout">` 之前插入：

```html
<div class="mobile-progress" aria-hidden="true">
  <div class="mobile-progress-bar" id="mobileProgressBar"></div>
</div>
```

- [ ] **Step 3：添加追踪器 CSS**

```css
.page-layout {
  position: relative;
  z-index: 1;
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 32px;
  max-width: calc(var(--container) + 220px);
  margin: 0 auto;
  padding: 0 32px;
}
.mission-tracker {
  position: sticky;
  top: 100px;
  height: fit-content;
  padding: 24px 20px;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  backdrop-filter: blur(20px);
}
.tracker-title {
  font-size: 11px;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 1.5px;
  margin-bottom: 20px;
}
.tracker-list { list-style: none; display: flex; flex-direction: column; gap: 14px; }
.tracker-list a {
  display: flex;
  align-items: center;
  gap: 12px;
  color: var(--text-muted);
  text-decoration: none;
  font-size: 14px;
  padding: 8px 10px;
  border-radius: 10px;
  transition: all 0.2s;
}
.tracker-list a:hover { color: var(--text-primary); background: rgba(255,255,255,0.04); }
.tracker-list a.active { color: var(--accent); background: rgba(34,197,94,0.08); }
.tracker-list a.done { color: var(--text-secondary); }
.tracker-list a.done .tracker-num { background: var(--accent); color: #000; border-color: var(--accent); }
.tracker-num {
  width: 26px; height: 26px;
  border-radius: 50%;
  border: 1px solid var(--border-strong);
  display: grid; place-items: center;
  font-size: 11px; font-weight: 700;
  flex-shrink: 0;
  transition: all 0.2s;
}
.tracker-list a.active .tracker-num { border-color: var(--accent); color: var(--accent); box-shadow: 0 0 12px var(--accent-glow); }

.mobile-progress {
  display: none;
  position: fixed;
  top: 68px; left: 0; right: 0;
  height: 2px;
  background: var(--border);
  z-index: 99;
}
.mobile-progress-bar {
  height: 100%;
  width: 0%;
  background: linear-gradient(90deg, var(--accent), var(--accent-highlight));
  transition: width 0.3s ease;
}
```

- [ ] **Step 4：更新滚动监听脚本**

把现有的 section 监听替换为同时更新追踪器：

```js
const trackerLinks = document.querySelectorAll('.tracker-list a');
const mobileProgressBar = document.getElementById('mobileProgressBar');

function updateActiveSection() {
  let current = '';
  let currentIndex = -1;
  sections.forEach((sec, idx) => {
    const top = sec.offsetTop - 150;
    if (window.scrollY >= top) {
      current = sec.getAttribute('id');
      currentIndex = idx;
    }
  });

  navLinks.forEach(a => {
    a.classList.toggle('active', a.getAttribute('href') === '#' + current);
  });

  trackerLinks.forEach(a => {
    const target = a.getAttribute('data-target');
    a.classList.toggle('active', target === current);
    const isDone = (trackerLinks.indexOf ? Array.from(trackerLinks).indexOf(a) : [...trackerLinks].indexOf(a)) < currentIndex;
    a.classList.toggle('done', isDone);
  });

  if (mobileProgressBar && sections.length) {
    const docHeight = document.documentElement.scrollHeight - window.innerHeight;
    const pct = Math.min(100, Math.max(0, (window.scrollY / docHeight) * 100));
    mobileProgressBar.style.width = pct + '%';
  }
}

window.addEventListener('scroll', () => {
  if (window.scrollY > 50) navbar.classList.add('scrolled');
  else navbar.classList.remove('scrolled');
  updateActiveSection();
});

updateActiveSection();
```

- [ ] **Step 5：添加响应式隐藏/显示**

在媒体查询中添加：

```css
@media (max-width: 1024px) {
  .page-layout { grid-template-columns: 1fr; padding: 0 20px; }
  .mission-tracker { display: none; }
  .mobile-progress { display: block; }
}
```

- [ ] **Step 6：验证**

- 桌面左侧显示 Mission Log，滚动时当前步骤高亮。
- 已完成的步骤编号变为绿色填充。
- 移动端左侧追踪器隐藏，顶部出现绿色进度条。

---

## Task 6：把四步内容重构为任务关卡布局

**Files：**
- Modify: `index.html` 中四个 `<section>` 的结构。

- [ ] **Step 1：定义统一的 mission 容器 CSS**

```css
.mission-card {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  padding: 40px;
  margin-bottom: 32px;
  backdrop-filter: blur(20px);
  transition: border-color 0.25s ease, box-shadow 0.25s ease;
}
.mission-card:hover {
  border-color: var(--border-strong);
  box-shadow: 0 0 40px rgba(34, 197, 94, 0.06);
}
.mission-header {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 28px;
}
.mission-avatar {
  width: 42px; height: 42px;
  border-radius: 12px;
  background: #0a0a0a;
  border: 1px solid var(--accent);
  display: grid; place-items: center;
  flex-shrink: 0;
  overflow: hidden;
}
.mission-avatar img { width: 70%; height: 70%; object-fit: contain; }
.mission-bubble {
  flex: 1;
  background: rgba(255,255,255,0.04);
  border: 1px solid var(--border);
  border-radius: 18px 18px 18px 4px;
  padding: 16px 20px;
  font-size: 15px;
  line-height: 1.7;
  color: var(--text-tertiary);
}
.mission-bubble strong { color: var(--accent); font-weight: 600; }
```

- [ ] **Step 2：在每个 section 内部开头插入 mission-header 与 choice-cards**

以 `#sow` 为例，在 `<div class="container">` 后、`<div class="section-head reveal">` 之前插入：

```html
<div class="mission-card reveal">
  <div class="mission-header">
    <div class="mission-avatar"><img src="assets/trae_dark.png" alt="TRAE Work"></div>
    <div class="mission-bubble">
      欢迎来到灵感孵化舱。第一步，我们需要把大赛模板喂给我，并开启 <strong>brainstorming</strong> 技能。完成下方任意动作即可推进。
    </div>
  </div>
  <div class="choice-grid" data-mission="sow">
    <button class="choice-card" data-action="download-template" type="button">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      <span>下载模板</span>
    </button>
    <button class="choice-card" data-action="install-skill" type="button">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/></svg>
      <span>安装 brainstorming 技能</span>
    </button>
    <button class="choice-card" data-action="copy-prompt-sow" type="button">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
      <span>复制播种提示词</span>
    </button>
    <button class="choice-card" data-action="screenshot-sow" type="button">
      <svg width="22" height="22" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="18" height="18" rx="2"/><circle cx="8.5" cy="8.5" r="1.5"/><polyline points="21 15 16 10 5 21"/></svg>
      <span>查看截图示意</span>
    </button>
  </div>
</div>
```

- [ ] **Step 3：为 02/03/04 插入类似结构**

每个 section 的 `data-mission` 分别为 `water`、`prune`、`bloom`。

- `water` 选择卡片：`复制浇灌提示词`（如有）、`查看对话截图` → 触发 `showScreenshot2()`。
- `prune` 选择卡片：`整理思路`、`导出 md`、`查看修剪截图` → 触发 `showScreenshot3()`。
- `bloom` 选择卡片：`生成 HTML Demo`、`查看绽放截图`、`去报名` → 分别触发 `showScreenshot6()` 与外链投稿。

文案保持与现有 section 内容一致即可。

- [ ] **Step 4：添加选择卡片 CSS**

```css
.choice-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
  margin-top: 20px;
}
.choice-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  padding: 18px 14px;
  background: rgba(255,255,255,0.03);
  border: 1px solid var(--border);
  border-radius: 14px;
  color: var(--text-secondary);
  font-family: var(--font-sans);
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}
.choice-card:hover {
  transform: translateY(-4px);
  border-color: var(--accent);
  color: var(--text-primary);
  box-shadow: 0 8px 24px rgba(34, 197, 94, 0.12);
}
.choice-card.completed {
  border-color: var(--accent);
  background: rgba(34, 197, 94, 0.08);
  color: var(--accent);
}
.choice-card.completed::after {
  content: "✓";
  position: absolute;
  top: 8px; right: 10px;
  font-size: 12px;
  color: var(--accent);
}
.choice-card { position: relative; }
.choice-card svg { stroke: currentColor; }
```

- [ ] **Step 5：验证**

- 每个 section 顶部出现 TRAE Work 头像 + 任务气泡 + 选择卡片。
- 选择卡片悬停有发光效果。
- 现有提示词卡片、时间线等内容仍然保留在下方。

---

## Task 7：实现选择卡片交互与 localStorage 持久化

**Files：**
- Modify: `index.html` 脚本区域。

- [ ] **Step 1：添加选择卡片状态管理脚本**

在现有脚本末尾添加：

```js
// ---- Choice Cards ----
const CHOICE_KEY = 'trae-incubator-choices';

function getChoices() {
  try {
    return JSON.parse(localStorage.getItem(CHOICE_KEY)) || {};
  } catch (e) {
    return {};
  }
}

function saveChoices(state) {
  try {
    localStorage.setItem(CHOICE_KEY, JSON.stringify(state));
  } catch (e) {
    // silently fail in private mode
  }
}

function toggleChoice(btn, action) {
  const state = getChoices();
  const mission = btn.closest('.choice-grid')?.dataset.mission || 'global';
  const key = `${mission}:${action}`;
  const completed = !state[key];
  state[key] = completed;
  saveChoices(state);
  btn.classList.toggle('completed', completed);

  // Optional user feedback bubble
  const missionCard = btn.closest('.mission-card');
  let feedback = missionCard?.querySelector('.user-feedback');
  if (!feedback && missionCard) {
    feedback = document.createElement('div');
    feedback.className = 'user-feedback';
    feedback.innerHTML = '<span>已完成该动作，继续下一步吧。</span>';
    missionCard.appendChild(feedback);
  }
  if (feedback) {
    feedback.style.opacity = '1';
    feedback.style.transform = 'translateY(0)';
  }
}

document.querySelectorAll('.choice-card').forEach(btn => {
  const action = btn.dataset.action;
  const mission = btn.closest('.choice-grid')?.dataset.mission || 'global';
  const state = getChoices();
  if (state[`${mission}:${action}`]) btn.classList.add('completed');

  btn.addEventListener('click', () => {
    // Execute mapped action
    if (action === 'download-template') downloadTemplate();
    else if (action === 'copy-prompt-sow') {
      const copyBtn = document.querySelector('#prompt-sow')?.closest('.prompt-card')?.querySelector('.copy-btn');
      if (copyBtn) copyBtn.click();
    }
    else if (action === 'copy-prompt-prune') {
      const copyBtn = document.querySelector('#prompt-prune')?.closest('.prompt-card')?.querySelector('.copy-btn');
      if (copyBtn) copyBtn.click();
    }
    else if (action === 'copy-prompt-bloom') {
      const copyBtn = document.querySelector('#prompt-bloom')?.closest('.prompt-card')?.querySelector('.copy-btn');
      if (copyBtn) copyBtn.click();
    }
    else if (action === 'screenshot-sow') showScreenshot();
    else if (action === 'screenshot-water') showScreenshot2();
    else if (action === 'screenshot-prune') showScreenshot3();
    else if (action === 'screenshot-bloom') showScreenshot6();
    else if (action === 'go-submit') {
      window.open('https://forum.trae.cn/c/38-category/39-category/39', '_blank');
    }

    toggleChoice(btn, action);
  });
});
```

- [ ] **Step 2：添加用户反馈气泡 CSS**

```css
.user-feedback {
  display: flex;
  justify-content: flex-end;
  margin-top: 18px;
  opacity: 0;
  transform: translateY(10px);
  transition: all 0.3s ease;
}
.user-feedback span {
  background: rgba(34, 197, 94, 0.12);
  border: 1px solid rgba(34, 197, 94, 0.3);
  color: var(--accent-highlight);
  border-radius: 18px 18px 4px 18px;
  padding: 10px 16px;
  font-size: 13px;
}
```

- [ ] **Step 3：验证**

- 点击“下载模板”触发下载，同时卡片变绿并出现 ✓。
- 刷新页面后，已完成卡片保持绿色。
- 隐私模式下点击不报错。

---

## Task 8：升级内容卡片为玻璃风格

**Files：**
- Modify: `index.html` 中 `.prompt-card`、`.step-card`、`.timeline`、`.result-card`、`.quote-block`、`.upload-hint`、`.cta-card`、`.skill-tag` 的 CSS。

- [ ] **Step 1：统一内容卡片基础样式**

把以下通用样式追加到 CSS 中（原有具体样式保留并做覆盖）：

```css
.prompt-card,
.step-card,
.result-card,
.quote-block,
.upload-hint,
.cta-card,
.mindmap-node {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  backdrop-filter: blur(20px);
}
```

- [ ] **Step 2：调整 hover 发光与边框颜色**

确保所有卡片 hover 使用统一的绿色光晕：

```css
.prompt-card:hover,
.step-card:hover,
.result-card:hover,
.mindmap-node:hover {
  border-color: var(--accent);
  box-shadow: 0 0 30px rgba(34, 197, 94, 0.08);
}
```

- [ ] **Step 3：修复 `.cta-card` 的渐变错误**

原 CSS 中 `.cta-card` 使用了未定义的 `--card` 变量。替换为：

```css
.cta-card {
  margin-top: 64px;
  padding: 48px 32px;
  background: linear-gradient(135deg, var(--accent-soft) 0%, rgba(255,255,255,0.02) 100%);
  border-radius: var(--radius-card);
  border: 1px solid var(--accent);
  text-align: center;
  position: relative;
  overflow: hidden;
}
```

- [ ] **Step 4：验证**

- 所有卡片呈现半透明玻璃效果。
- 悬停时边框变绿、出现柔和光晕。
- CTA 卡片无 CSS 错误。

---

## Task 9：添加滚动揭示与交互动效

**Files：**
- Modify: `index.html` CSS 与 JS 滚动监听区域。

- [ ] **Step 1：更新 reveal 动画为 stagger 支持**

替换 `.reveal` 相关 CSS：

```css
.reveal {
  opacity: 0;
  transform: translateY(28px);
  transition: opacity 0.7s cubic-bezier(0.22, 1, 0.36, 1), transform 0.7s cubic-bezier(0.22, 1, 0.36, 1);
}
.reveal.visible {
  opacity: 1;
  transform: translateY(0);
}
.reveal.visible:nth-child(1) { transition-delay: 0s; }
.reveal.visible:nth-child(2) { transition-delay: 0.08s; }
.reveal.visible.visible:nth-child(3) { transition-delay: 0.16s; }
.reveal.visible:nth-child(4) { transition-delay: 0.24s; }
.reveal.visible:nth-child(5) { transition-delay: 0.32s; }
```

- [ ] **Step 2：为对话气泡添加滑入动画**

```css
.mission-bubble {
  opacity: 0;
  transform: translateX(-20px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}
.mission-card.visible .mission-bubble,
.mission-card.in-view .mission-bubble {
  opacity: 1;
  transform: translateX(0);
}
```

- [ ] **Step 3：更新 IntersectionObserver 以支持 mission-card**

```js
const reveals = document.querySelectorAll('.reveal, .mission-card, .timeline-item');
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible', 'in-view');
    }
  });
}, { threshold: 0.12, rootMargin: '0px 0px -60px 0px' });

reveals.forEach(el => observer.observe(el));
```

- [ ] **Step 4：添加 reduced-motion 媒体查询**

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
  #particle-canvas { display: none; }
}
```

- [ ] **Step 5：验证**

- 向下滚动时卡片依次淡入。
- 对话气泡从左侧滑入。
- 系统开启“减少动效”后，粒子画布隐藏，动画消失。

---

## Task 10：响应式与无障碍收尾

**Files：**
- Modify: `index.html` CSS 媒体查询与 HTML 属性。

- [ ] **Step 1：完善移动端样式**

在 `@media (max-width: 1024px)` 中追加：

```css
.hero h1 { font-size: 40px; }
.hero-cta { justify-content: center; }
.mission-card { padding: 24px; }
.choice-grid { grid-template-columns: repeat(2, 1fr); }
```

在 `@media (max-width: 768px)` 中追加：

```css
.hero h1 { font-size: 34px; letter-spacing: -1px; }
.choice-grid { grid-template-columns: 1fr; }
.mission-bubble { font-size: 14px; padding: 14px 16px; }
.tracker-title { display: none; }
```

- [ ] **Step 2：为选择卡片添加焦点样式**

```css
.choice-card:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}
```

- [ ] **Step 3：为任务追踪器链接添加 aria 属性**

每个 tracker 链接已包含 `data-target`，无需额外 JS。HTML 中已使用 `<aside aria-label="任务进度">` 与 `<ul>`。

- [ ] **Step 4：验证**

- 在 375px 宽度下，Hero 标题、选择卡片、气泡均正常显示。
- 使用 Tab 键可聚焦选择卡片，Enter 触发。
- 无动画偏好下页面不闪烁。

---

## Task 11：最终测试与清理

**Files：**
- Modify: `index.html`（按需微调）。

- [ ] **Step 1：功能清单逐项验证**

在浏览器中打开 `index.html`，检查：

- [ ] 四个 Hero CTA：开始孵化滚动到 #sow；其余三个外链在新标签打开。
- [ ] 任务追踪器：滚动时当前步骤高亮；点击步骤平滑滚动。
- [ ] 选择卡片：下载、复制、查看截图、外链动作均正常；完成状态刷新后保留。
- [ ] 复制按钮：点击后文案变为“已复制”，2 秒后恢复。
- [ ] 模态框与轮播：能打开、切换、关闭（Esc / 背景 / ×）。
- [ ] 响应式：320px、768px、1024px、1440px 无布局崩坏。
- [ ] 无障碍：Tab 导航完整；减少动效下无强制动画。
- [ ] 控制台：无 JS 报错。

- [ ] **Step 2：提交代码**

```bash
git add index.html
git commit -m "feat: redesign TRAE incubator with mission-based UI and glassmorphism"
```

---

## 自检清单

- **Spec coverage：** 视觉方向（Task 1）、Hero 与 CTA（Task 4）、任务追踪器（Task 5）、对话气泡与选择卡片（Task 6/7）、玻璃卡片（Task 8）、动效（Task 9）、响应式/无障碍（Task 10）、测试（Task 11）均已覆盖。
- **Placeholder scan：** 无 TBD/TODO/空代码块。
- **Type consistency：** 所有选择卡片使用 `data-action` 与 `data-mission`；localStorage key 统一为 `${mission}:${action}`；追踪器使用 `data-target`。
