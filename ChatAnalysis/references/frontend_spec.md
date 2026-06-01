# 前端设计规范

> 本文档定义 HTML 输出的完整设计系统。Agent 在 Phase 3 渲染时参照本规范生成 CSS 和页面结构。
> 与 `SKILL.md`（流程定义）、`script_writing_guide.md`（脚本模式）配合使用。

---

## 1. 技术约束

- **技术栈：** 纯 HTML + CSS + JS，零外部依赖（无框架、无 CDN、无字体服务）
- **独立部署：** 每个 `.xxx/` 形式目录完全自包含（`.assets/` + `resources/` + 页面），可单独部署到任意静态服务器
- **相对路径：** 所有链接、引用使用相对路径，禁止绝对路径、`file://` 协议、硬编码域名
- **浏览器存储：** 允许使用 `localStorage` 持久化用户偏好（主题切换）

---

## 2. 设计系统

### 2.1 CSS 变量体系

通过 CSS 变量实现主题切换。所有颜色、间距、圆角均通过变量引用，禁止硬编码色值。

```css
:root {
    /* 布局 */
    --max-width: 880px;          /* 由布局决定 */
    --content-padding: 24px;

    /* 圆角 */
    --radius-sm: 6px;
    --radius-md: 10px;
    --radius-lg: 16px;
    --radius-pill: 999px;

    /* 间距 */
    --space-xs: 4px;
    --space-sm: 8px;
    --space-md: 16px;
    --space-lg: 24px;
    --space-xl: 40px;

    /* 排版 */
    --font-body: 15px;
    --font-small: 13px;
    --font-caption: 12px;
    --line-height: 1.75;
    --heading-weight: 700;
}
```

### 2.2 字体栈

```css
body {
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display",
                 "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
    font-size: var(--font-body);
    line-height: var(--line-height);
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}

code, pre {
    font-family: "SF Mono", "Fira Code", "JetBrains Mono",
                 "Cascadia Code", Consolas, monospace;
}
```

### 2.3 主题色值

5 种预设主题，每种含 light/dark 变体。用户在 Phase 0.3 选择主题，Agent 在运行时生成对应 CSS。

#### Apple（默认）

```python
APPLE_LIGHT = {
    "bg": "#ffffff", "surface": "#f5f5f7", "surface2": "#e8e8ed",
    "text": "#1d1d1f", "text-muted": "#86868b",
    "accent": "#007aff", "accent-hover": "#0056b3",
    "link": "#007aff", "link-hover": "#0056b3",
    "border": "rgba(0,0,0,0.08)", "border-strong": "rgba(0,0,0,0.15)",
    "code-bg": "#f5f5f7", "code-color": "#c0392b", "pre-color": "#6c5ce7",
    "table-header-bg": "#f5f5f7",
    "hover-bg": "rgba(0,0,0,0.03)",
    "shadow": "0 1px 3px rgba(0,0,0,0.06)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.08)",
    "blockquote-bg": "rgba(0,122,255,0.04)", "blockquote-border": "#007aff",
    "success": "#34c759", "warning": "#ff9500", "danger": "#ff3b30",
}

APPLE_DARK = {
    "bg": "#000000", "surface": "#1c1c1e", "surface2": "#2c2c2e",
    "text": "#f5f5f7", "text-muted": "#98989f",
    "accent": "#0a84ff", "accent-hover": "#409cff",
    "link": "#0a84ff", "link-hover": "#409cff",
    "border": "rgba(255,255,255,0.1)", "border-strong": "rgba(255,255,255,0.2)",
    "code-bg": "#1c1c1e", "code-color": "#f48fb1", "pre-color": "#ce93d8",
    "table-header-bg": "#1c1c1e",
    "hover-bg": "rgba(255,255,255,0.04)",
    "shadow": "0 1px 3px rgba(0,0,0,0.3)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.4)",
    "blockquote-bg": "rgba(10,132,255,0.08)", "blockquote-border": "#0a84ff",
    "success": "#30d158", "warning": "#ff9f0a", "danger": "#ff453a",
}
```

#### Nord

```python
NORD_LIGHT = {
    "bg": "#eceff4", "surface": "#e5e9f0", "surface2": "#d8dee9",
    "text": "#2e3440", "text-muted": "#4c566a",
    "accent": "#5e81ac", "accent-hover": "#4c6d94",
    "link": "#5e81ac", "link-hover": "#4c6d94",
    "border": "rgba(46,52,64,0.1)", "border-strong": "rgba(46,52,64,0.2)",
    "code-bg": "#e5e9f0", "code-color": "#bf616a", "pre-color": "#b48ead",
    "table-header-bg": "#d8dee9",
    "hover-bg": "rgba(46,52,64,0.04)",
    "shadow": "0 1px 3px rgba(46,52,64,0.08)",
    "shadow-lg": "0 4px 12px rgba(46,52,64,0.12)",
    "blockquote-bg": "rgba(94,129,172,0.06)", "blockquote-border": "#5e81ac",
    "success": "#a3be8c", "warning": "#ebcb8b", "danger": "#bf616a",
}

NORD_DARK = {
    "bg": "#2e3440", "surface": "#3b4252", "surface2": "#434c5e",
    "text": "#eceff4", "text-muted": "#d8dee9",
    "accent": "#88c0d0", "accent-hover": "#8fbcbb",
    "link": "#88c0d0", "link-hover": "#8fbcbb",
    "border": "rgba(236,239,244,0.08)", "border-strong": "rgba(236,239,244,0.15)",
    "code-bg": "#3b4252", "code-color": "#bf616a", "pre-color": "#b48ead",
    "table-header-bg": "#3b4252",
    "hover-bg": "rgba(236,239,244,0.03)",
    "shadow": "0 1px 3px rgba(0,0,0,0.2)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.3)",
    "blockquote-bg": "rgba(136,192,208,0.06)", "blockquote-border": "#88c0d0",
    "success": "#a3be8c", "warning": "#ebcb8b", "danger": "#bf616a",
}
```

#### Dracula

```python
DRACULA_LIGHT = {
    "bg": "#f8f8f2", "surface": "#f0efe8", "surface2": "#e6e5de",
    "text": "#282a36", "text-muted": "#6272a4",
    "accent": "#9b59b6", "accent-hover": "#8e44ad",
    "link": "#6272a4", "link-hover": "#44475a",
    "border": "rgba(40,42,54,0.1)", "border-strong": "rgba(40,42,54,0.2)",
    "code-bg": "#f0efe8", "code-color": "#ff5555", "pre-color": "#9b59b6",
    "table-header-bg": "#e6e5de",
    "hover-bg": "rgba(40,42,54,0.04)",
    "shadow": "0 1px 3px rgba(40,42,54,0.06)",
    "shadow-lg": "0 4px 12px rgba(40,42,54,0.1)",
    "blockquote-bg": "rgba(155,89,182,0.06)", "blockquote-border": "#9b59b6",
    "success": "#50fa7b", "warning": "#f1fa8c", "danger": "#ff5555",
}

DRACULA_DARK = {
    "bg": "#282a36", "surface": "#343746", "surface2": "#3e4155",
    "text": "#f8f8f2", "text-muted": "#6272a4",
    "accent": "#bd93f9", "accent-hover": "#a87de8",
    "link": "#8be9fd", "link-hover": "#6ed0e8",
    "border": "rgba(248,248,242,0.06)", "border-strong": "rgba(248,248,242,0.12)",
    "code-bg": "#343746", "code-color": "#ff5555", "pre-color": "#bd93f9",
    "table-header-bg": "#343746",
    "hover-bg": "rgba(248,248,242,0.03)",
    "shadow": "0 1px 3px rgba(0,0,0,0.3)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.4)",
    "blockquote-bg": "rgba(189,147,249,0.06)", "blockquote-border": "#bd93f9",
    "success": "#50fa7b", "warning": "#f1fa8c", "danger": "#ff5555",
}
```

#### Gruvbox

```python
GRUVBOX_LIGHT = {
    "bg": "#fbf1c7", "surface": "#f2e5bc", "surface2": "#ebdbb2",
    "text": "#3c3836", "text-muted": "#665c54",
    "accent": "#b57614", "accent-hover": "#9a6412",
    "link": "#076678", "link-hover": "#458588",
    "border": "rgba(60,56,54,0.1)", "border-strong": "rgba(60,56,54,0.2)",
    "code-bg": "#f2e5bc", "code-color": "#cc241d", "pre-color": "#8f3f71",
    "table-header-bg": "#ebdbb2",
    "hover-bg": "rgba(60,56,54,0.04)",
    "shadow": "0 1px 3px rgba(60,56,54,0.08)",
    "shadow-lg": "0 4px 12px rgba(60,56,54,0.12)",
    "blockquote-bg": "rgba(181,118,20,0.06)", "blockquote-border": "#b57614",
    "success": "#98971a", "warning": "#d79921", "danger": "#cc241d",
}

GRUVBOX_DARK = {
    "bg": "#282828", "surface": "#3c3836", "surface2": "#504945",
    "text": "#ebdbb2", "text-muted": "#a89984",
    "accent": "#d79921", "accent-hover": "#fabd2f",
    "link": "#83a598", "link-hover": "#458588",
    "border": "rgba(235,219,178,0.08)", "border-strong": "rgba(235,219,178,0.15)",
    "code-bg": "#3c3836", "code-color": "#fb4934", "pre-color": "#d3869b",
    "table-header-bg": "#3c3836",
    "hover-bg": "rgba(235,219,178,0.03)",
    "shadow": "0 1px 3px rgba(0,0,0,0.3)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.4)",
    "blockquote-bg": "rgba(215,153,33,0.06)", "blockquote-border": "#d79921",
    "success": "#b8bb26", "warning": "#fabd2f", "danger": "#fb4934",
}
```

#### Solarized

```python
SOLARIZED_LIGHT = {
    "bg": "#fdf6e3", "surface": "#eee8d5", "surface2": "#ddd6c1",
    "text": "#657b83", "text-muted": "#93a1a1",
    "accent": "#268bd2", "accent-hover": "#1a6da0",
    "link": "#268bd2", "link-hover": "#1a6da0",
    "border": "rgba(101,124,131,0.1)", "border-strong": "rgba(101,124,131,0.2)",
    "code-bg": "#eee8d5", "code-color": "#dc322f", "pre-color": "#6c71c4",
    "table-header-bg": "#ddd6c1",
    "hover-bg": "rgba(101,124,131,0.04)",
    "shadow": "0 1px 3px rgba(101,124,131,0.08)",
    "shadow-lg": "0 4px 12px rgba(101,124,131,0.12)",
    "blockquote-bg": "rgba(38,139,210,0.06)", "blockquote-border": "#268bd2",
    "success": "#859900", "warning": "#b58900", "danger": "#dc322f",
}

SOLARIZED_DARK = {
    "bg": "#002b36", "surface": "#073642", "surface2": "#094856",
    "text": "#839496", "text-muted": "#586e75",
    "accent": "#268bd2", "accent-hover": "#4aa3e0",
    "link": "#2aa198", "link-hover": "#44b0a7",
    "border": "rgba(131,148,150,0.08)", "border-strong": "rgba(131,148,150,0.15)",
    "code-bg": "#073642", "code-color": "#dc322f", "pre-color": "#6c71c4",
    "table-header-bg": "#073642",
    "hover-bg": "rgba(131,148,150,0.03)",
    "shadow": "0 1px 3px rgba(0,0,0,0.3)",
    "shadow-lg": "0 4px 12px rgba(0,0,0,0.4)",
    "blockquote-bg": "rgba(42,161,152,0.06)", "blockquote-border": "#2aa198",
    "success": "#859900", "warning": "#b58900", "danger": "#dc322f",
}
```

**Python → CSS 注入：** Agent 在运行时根据用户选择的主题，调用 `build_css(theme_name)` 生成完整 CSS 字符串，写入 `.assets/style.css`。主题变量通过 `:root` 和 `[data-theme="dark"]` 选择器注入。

### 2.4 主题切换

每个页面必须包含无闪烁的主题初始化和切换按钮。

**Head 中（可见内容之前）：**
```html
<script>
(function() {
    var t = localStorage.getItem('theme') || 'light';
    if (t === 'dark') document.documentElement.setAttribute('data-theme', 'dark');
})();
</script>
```

**切换按钮（固定右上角）：**
```html
<button class="theme-toggle" id="themeToggle" aria-label="切换主题">🌙 暗色</button>
```

```css
.theme-toggle {
    position: fixed; top: 16px; right: 16px; z-index: 1000;
    padding: 8px 14px; border: 1px solid var(--border);
    border-radius: var(--radius-pill); background: var(--surface);
    color: var(--text); font-size: var(--font-small); cursor: pointer;
    transition: background 0.2s, border-color 0.2s;
    box-shadow: var(--shadow);
}
.theme-toggle:hover { background: var(--hover-bg); border-color: var(--border-strong); }
```

**切换脚本（`</body>` 之前）：**
```javascript
(function() {
    var btn = document.getElementById('themeToggle');
    if (!btn) return;
    function apply(t) {
        if (t === 'dark') {
            document.documentElement.setAttribute('data-theme', 'dark');
            btn.textContent = '☀️ 浅色';
        } else {
            document.documentElement.removeAttribute('data-theme');
            btn.textContent = '🌙 暗色';
        }
    }
    apply(localStorage.getItem('theme') || 'light');
    btn.addEventListener('click', function() {
        var next = localStorage.getItem('theme') === 'dark' ? 'light' : 'dark';
        localStorage.setItem('theme', next);
        apply(next);
    });
})();
```

---

## 3. 布局系统

### 3.1 预设布局

| 布局 | `--max-width` | 导航方式 | 适用场景 |
|------|-------------|----------|----------|
| **Standard**（默认） | 880px | 面包屑 | 大多数场景，单栏居中 |
| **Wide** | 1200px | 右侧悬浮 TOC | 长文报告，宽屏浏览 |
| **Magazine** | 880px | 面包屑 | 首页双栏卡片网格，内页标准单栏 |
| **Compact** | 880px | 面包屑 | 大量内容快速浏览，更小间距 |

**布局选择机制：** 用户在 Phase 0.3 选择布局后，Agent 在 `:root` 中注入对应的 CSS 变量覆盖。Standard 为默认值无需额外覆盖；Wide/Magazine/Compact 通过 `data-layout` 属性选择：

```css
body[data-layout="wide"] { --max-width: 1200px; }
body[data-layout="compact"] { --space-lg: 16px; --space-xl: 24px; }
```

### 3.2 页面骨架

```css
body {
    margin: 0; padding: 0;
    background: var(--bg); color: var(--text);
}
.container {
    max-width: var(--max-width);
    margin: 0 auto;
    padding: var(--space-xl) var(--content-padding);
}
```

### 3.3 响应式断点

```css
/* 移动端适配 */
@media (max-width: 768px) {
    .container { padding: var(--space-md) var(--space-md); }
    .theme-toggle { top: 8px; right: 8px; padding: 6px 10px; font-size: 12px; }
    table { display: block; overflow-x: auto; -webkit-overflow-scrolling: touch; }
    .leaderboard-bar { display: none; }  /* 隐藏排行榜柱状图 */
    .magazine-grid { grid-template-columns: 1fr; }  /* Magazine 布局单列 */
    .danger-block { padding: 12px 14px; }
    blockquote { padding: 10px 12px; }
    footer { margin-top: var(--space-lg); }
    .toc-sidebar { display: none; }  /* 移动端隐藏 TOC 侧栏 */
}
```

### 3.4 Wide 布局 TOC 侧栏

Wide 布局使用右侧悬浮 TOC 替代面包屑导航：

```html
<body data-layout="wide">
<div class="wide-layout">
    <div class="content"><!-- 页面内容 --></div>
    <aside class="toc-sidebar">
        <nav><!-- JS 从 h2/h3 生成 TOC --></nav>
    </aside>
</div>
</body>
```

```css
body[data-layout="wide"] .wide-layout {
    display: flex; gap: var(--space-lg);
    max-width: var(--max-width); margin: 0 auto;
    padding: var(--space-xl) var(--content-padding);
}
body[data-layout="wide"] .content { flex: 1; min-width: 0; }
.toc-sidebar {
    position: sticky; top: var(--space-lg);
    flex: 0 0 240px; max-height: calc(100vh - var(--space-xl));
    overflow-y: auto; font-size: var(--font-small);
    padding: var(--space-md); background: var(--surface);
    border-radius: var(--radius-md); align-self: flex-start;
}
.toc-sidebar a { color: var(--text-muted); text-decoration: none; display: block; padding: 4px 0; }
.toc-sidebar a:hover { color: var(--link); }
```

---

## 4. 组件模式

### 4.1 面包屑导航

每个子页面顶部显示位置路径：

```html
<nav class="breadcrumb">
    <a href="../index.html">首页</a>
    <span class="sep">›</span>
    <a href="index.html">统计分析</a>
    <span class="sep">›</span>
    <span>消息概览</span>
</nav>
```

```css
.breadcrumb {
    padding: var(--space-sm) 0; margin-bottom: var(--space-lg);
    font-size: var(--font-small); color: var(--text-muted);
}
.breadcrumb a { color: var(--text-muted); text-decoration: none; }
.breadcrumb a:hover { color: var(--link); }
.breadcrumb .sep { margin: 0 8px; opacity: 0.5; }
```

### 4.2 排行榜（群聊首页）

```html
<a class="leaderboard-item" href="personality/用户名.html">
    <span class="leaderboard-rank top3">#1</span>
    <span class="leaderboard-name">用户名</span>
    <span class="leaderboard-count">2638 条</span>
    <div class="leaderboard-bar">
        <div class="leaderboard-bar-fill" style="width:100%"></div>
    </div>
</a>
```

```css
.leaderboard-item {
    display: grid; grid-template-columns: 40px 1fr auto 120px;
    align-items: center; gap: var(--space-sm);
    padding: 12px 16px; margin: 4px 0;
    border-radius: var(--radius-md); text-decoration: none; color: var(--text);
    transition: background 0.15s;
}
.leaderboard-item:hover { background: var(--hover-bg); }
.leaderboard-rank {
    font-weight: 700; font-size: 1.1em; color: var(--text-muted); text-align: center;
}
.leaderboard-rank.top3 { color: var(--accent); }
.leaderboard-name { font-weight: 600; }
.leaderboard-count { font-size: var(--font-small); color: var(--text-muted); }
.leaderboard-bar {
    height: 6px; background: var(--surface2); border-radius: var(--radius-pill); overflow: hidden;
}
.leaderboard-bar-fill {
    height: 100%; background: var(--accent); border-radius: var(--radius-pill);
    transition: width 0.6s ease;
}

@media (max-width: 768px) {
    .leaderboard-item { grid-template-columns: 32px 1fr auto; }
    .leaderboard-bar { display: none; }
}
```

### 4.3 危险/警告区块（高风险内容）

```html
<div class="danger-block">
    <p>原始聊天内容在这里...</p>
</div>
```

```css
.danger-block {
    border-left: 4px solid var(--danger);
    background: color-mix(in srgb, var(--danger) 6%, transparent);
    padding: 14px 18px; margin: var(--space-md) 0;
    border-radius: 0 var(--radius-md) var(--radius-md) 0;
}
.danger-block::before {
    content: "⚠️ 高危提醒";
    display: block; font-weight: 700; color: var(--danger);
    margin-bottom: 8px; font-size: 0.9em;
}
```

### 4.4 引用块（一般敏感内容警告）

```html
<blockquote>
    <p>此消息包含敏感内容，已自动标记。</p>
</blockquote>
```

```css
blockquote {
    border-left: 3px solid var(--blockquote-border);
    background: var(--blockquote-bg);
    padding: 12px 16px; margin: var(--space-md) 0;
    border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
    color: var(--text-muted); font-size: 0.95em;
}
```

### 4.5 折叠块（同类链接收起）

当某个分类下链接超过 5 个时，使用 `<details>` 收起：

```html
<details class="collapsed-links">
    <summary>话题专题（12 篇）</summary>
    <ul>
        <li><a href="topic/xxx.html">话题1</a></li>
        <!-- ... -->
    </ul>
</details>
```

```css
.collapsed-links { margin: var(--space-sm) 0; }
.collapsed-links summary {
    cursor: pointer; font-weight: 600; padding: 8px 0;
    color: var(--text-muted); font-size: var(--font-small);
}
.collapsed-links summary:hover { color: var(--link); }
```

### 4.6 时间分组（聊天记录长页）

```html
<div class="time-group">
    <div class="time-header">2026-03-17 22:30</div>
    <div class="msg"><span class="sender">用户A：</span>消息内容</div>
    <div class="msg"><span class="sender">用户B：</span>回复内容</div>
</div>
```

```css
.time-group { margin-bottom: var(--space-md); }
.time-header {
    text-align: center; font-size: var(--font-caption); color: var(--text-muted);
    margin: var(--space-md) 0 var(--space-sm);
    padding: 4px 14px; background: var(--surface2);
    border-radius: var(--radius-pill); display: inline-block;
}
.msg { padding: 3px 0; line-height: 1.7; }
.msg .sender { font-weight: 600; color: var(--accent); }
.msg.system { color: var(--text-muted); font-size: var(--font-small); text-align: center; }
.msg .card { color: var(--text-muted); font-style: italic; }
```

### 4.7 完整聊天记录入口按钮

```html
<a class="btn-whole-page" href=".whole/chat.html">📄 查看完整聊天记录</a>
```

```css
.btn-whole-page {
    display: inline-block; padding: 10px 22px; margin: var(--space-md) 0;
    background: var(--accent); color: #fff; border-radius: var(--radius-md);
    text-decoration: none; font-size: var(--font-small); font-weight: 500;
    transition: opacity 0.15s;
}
.btn-whole-page:hover { background: var(--accent-hover); }
```

---

## 5. 排版规范

### 5.1 标题层级

```css
h1 { font-size: 1.8em; font-weight: var(--heading-weight); margin: 0 0 var(--space-lg); line-height: 1.3; }
h2 { font-size: 1.4em; font-weight: var(--heading-weight); margin: var(--space-xl) 0 var(--space-md); line-height: 1.35; }
h3 { font-size: 1.15em; font-weight: 600; margin: var(--space-lg) 0 var(--space-sm); line-height: 1.4; }
h4 { font-size: 1em; font-weight: 600; margin: var(--space-md) 0 var(--space-xs); }
```

### 5.2 表格

```css
table {
    width: 100%; border-collapse: collapse; margin: var(--space-md) 0;
    font-size: var(--font-small);
}
th, td {
    padding: 10px 14px; text-align: left;
    border-bottom: 1px solid var(--border);
}
th {
    background: var(--table-header-bg); font-weight: 600;
    position: sticky; top: 0;
}
tr:hover td { background: var(--hover-bg); }
```

### 5.3 代码块

```css
pre {
    background: var(--code-bg); color: var(--pre-color);
    padding: 16px 20px; border-radius: var(--radius-md);
    overflow-x: auto; font-size: 0.88em; line-height: 1.6;
    margin: var(--space-md) 0;
}
code {
    background: var(--code-bg); color: var(--code-color);
    padding: 2px 6px; border-radius: 4px; font-size: 0.9em;
}
pre code { background: none; padding: 0; color: inherit; }
```

### 5.4 链接

```css
a { color: var(--link); text-decoration: none; }
a:hover { color: var(--link-hover); text-decoration: underline; }
```

---

## 6. 媒体嵌入

报告中引用媒体文件时，必须使用 HTML 标签（`<img>`、`<audio>`、`<video>`），不能用纯文本占位符。固定一个维度到 `min(360px, 50vw)`，另一维度 `auto` 保持比例。所有媒体引用使用相对路径。

**图片：**
```html
<img src="resources/images/photo.jpg" alt="描述"
     style="max-width: min(360px, 50vw); height: auto; border-radius: var(--radius-md);">
```

**音频：**
```html
<audio controls style="max-width: min(360px, 50vw); width: 100%;">
    <source src="resources/audio/voice.mp3" type="audio/mpeg">
</audio>
```

**视频：**
```html
<video controls style="max-width: min(360px, 50vw); border-radius: var(--radius-md);">
    <source src="resources/video/clip.mp4" type="video/mp4">
</video>
```

**Markdown 中的写法：** 在 md 文件中直接写 HTML 标签，不用 `![alt](url)` 语法（无法控制尺寸）。

**善用原始媒体：** 报告引用聊天原文时，如果原始消息包含图片/音频/视频，应主动将对应媒体嵌入报告。图文并茂的报告比纯文字更有质感。

**选择性迁移：** 只有实际被报告引用的媒体文件才从 `data/` 迁移到对应 `.xxx/` 形式目录的 `resources/` 下。各 `.xxx/` 各自独立迁移，不共享 `resources/`。

---

## 7. 页面生成模式

### 7.1 Markdown → HTML 转换

使用 `markdown` Python 库，配合扩展：

```python
EXTENSIONS = ["tables", "fenced_code", "toc", "nl2br", "sane_lists", "smarty"]
```

将 Markdown 引用块转换为危险区块：
```python
html = re.sub(
    r'<blockquote>\s*<p>\s*⚠️\s*<strong>高危提醒</strong>\s*⚠️(.*?)</blockquote>',
    r'<div class="danger-block">\1</div>',
    html, flags=re.DOTALL
)
```

### 7.2 分类名自动检测

```python
DEFAULT_CAT_NAMES = {
    "analyse": "统计分析",
    "report": "综合报告",
    "topic": "话题专题",
    "personality": "人格画像",
}
```

运行时扫描目录，未知分类名直接用目录名。

### 7.3 页脚链接

通过 CLI 参数 `--footer-link`（布尔标志，无额外值）控制。用户在 Phase 0.3 允许时注入页脚：

```html
<footer>
    <a href="https://github.com/JularDepick/ChatAnalysis.SKILL" target="_blank" rel="noopener noreferrer">
        SKILL技术支持·ChatAnalysis
    </a>
</footer>
```

```css
footer {
    margin-top: var(--space-xl); padding: var(--space-md) 0;
    border-top: 1px solid var(--border);
    text-align: center; font-size: var(--font-caption); color: var(--text-muted);
}
footer a { color: var(--text-muted); }
footer a:hover { color: var(--link); }
```

---

## 8. 首页设计

### 8.1 私聊首页

大标题显示双方昵称（如"张三 & 李四 的聊天分析"），下方展示统计概览和报告分类链接。

### 8.2 群聊首页

大标题显示群聊名称，下方展示：
- **TOP N 排行榜** — 柱状图可视化（纯 CSS），前三名高亮强调色，点击跳转人格画像
- **报告分类链接** — analyse/、report/、topic/、personality/ 等入口
- **more/ 链接（按需）** — 仅展示 Phase 0 用户选择的 more/ 页面

### 8.3 Magazine 布局首页

```css
.magazine-grid {
    display: grid; grid-template-columns: 1fr 1fr;
    gap: var(--space-md); margin: var(--space-lg) 0;
}
.magazine-card {
    background: var(--surface); border-radius: var(--radius-md);
    padding: var(--space-lg); box-shadow: var(--shadow);
    transition: box-shadow 0.2s, transform 0.2s;
}
.magazine-card:hover {
    box-shadow: var(--shadow-lg); transform: translateY(-2px);
}

@media (max-width: 768px) {
    .magazine-grid { grid-template-columns: 1fr; }
}
```

---

## 9. 样式文件组织

每个 `.xxx/` 形式目录（`.full/`、`.pure/`、`.parts/`、`.whole/`、`.sensitive/`）下的 `.assets/` 包含：

```
.assets/
├── style.css    ← 主样式（变量 + 布局 + 组件 + 响应式 + 排版）
└── theme.js     ← 主题切换脚本（可内联到各页面 <head>）
```

**推荐做法：** 将主题切换脚本内联到每个页面的 `<head>` 中（避免额外 HTTP 请求），`style.css` 通过 `<link>` 引用。同级目录页面引用 `".assets/style.css"`，子目录页面引用 `"../.assets/style.css"`。
