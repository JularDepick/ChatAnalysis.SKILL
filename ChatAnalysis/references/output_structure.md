# 输出目录结构

> 本文档定义完整的输出目录结构和各目录的职责。`SKILL.md` 中仅保留简化概览。

---

## 标准布局

```
workdir/                          — 工作目录
├── data/                         ← 原始数据（只读，不做任何修改）
└── .output/                      ← 所有输出（隐藏目录）
    ├── chat_stats.json           ← Phase 1 统计数据
    ├── README.ai.md              ← Agent 复用指南（Phase 3 生成）
    ├── scripts/                  ← 运行时生成的脚本（正式保留，供增量分析复用）
    ├── .cache/                   ← 临时文件/中间产物（任务完成后清理）
    ├── md/                       ← Markdown 报告（Phase 2 核心输出）
    │   ├── *.md                  ← 扁平模式下所有 .md 直接在此目录
    │   ├── analyse/              ← （分组模式）5 篇统计分析
    │   ├── report/               ← （分组模式）4 篇综合报告
    │   ├── topic/                ← （分组模式）N 篇话题专题
    │   ├── personality/          ← （分组模式）每用户画像
    │   ├── adventures/           ← （可选）群聊历险日记
    │   │   └── <user>.md
    │   ├── brilliant.md          ← （可选）精彩发言展示页
    │   └── legendary.md          ← （可选）神人发言展示页
    └── html/                     ← HTML 输出（Phase 3 从 md/ 渲染）
        ├── .full/                ← 完整版（默认，敏感内容用 <details> 隐藏）
        │   ├── .assets/          ← CSS/JS
        │   ├── resources/        ← 媒体文件
        │   ├── index.html        ← 首页（排行榜）
        │   ├── analyse/
        │   ├── report/
        │   ├── topic/
        │   ├── personality/
        │   ├── adventures/
        │   │   ├── index.html    ← 历险日记入口
        │   │   └── <user>.html
        │   └── more/
        │       ├── brilliant/
        │       │   └── index.html
        │       └── legendary/
        │           └── index.html
        ├── .parts/               ← 分块输出（可选，敏感内容大赏模块）
        │   ├── .assets/
        │   ├── resources/
        │   └── parts/            ← 分块页面
        ├── .whole/               ← 完整聊天记录长页（可选，默认不生成）
        │   ├── .assets/
        │   ├── resources/
        │   └── chat.html
        ├── .pure/                ← 纯净版（移除敏感块）
        │   ├── .assets/
        │   ├── resources/
        │   ├── index.html
        │   ├── analyse/
        │   ├── report/
        │   ├── topic/
        │   ├── personality/
        │   ├── adventures/
        │   │   ├── index.html
        │   │   └── <user>.html
        │   └── more/
        └── .sensitive/           ← 敏感内容大赏（可选）
            ├── .assets/
            └── resources/
```

---

## 关键约定

### data/ — 原始数据

- 存放用户提供的聊天文件或 JSONL 目录
- **只读**，不做任何修改
- Phase 0 目录初始化时确认数据在此目录

### .output/ — 输出根目录

- 隐藏目录，所有产出均在此下
- `chat_stats.json` 在此根目录，供所有报告引用

### .output/md/ — 核心 Markdown 输出

- Phase 2 的核心清洗输出，所有分析报告首先以 Markdown 形式写入
- **默认扁平放置**（所有 .md 直接在 md/ 下），除非用户在 Phase 0 要求按分类分子目录
- 分组模式下的子目录：`analyse/`、`report/`、`topic/`、`personality/`、`adventures/`
- `brilliant.md` 和 `legendary.md` 始终在 md/ 根目录（无论是否启用分组模式）
- HTML 渲染时映射关系：`md/brilliant.md` → `html/.full/more/brilliant/index.html`

### .output/html/ — HTML 输出

- 从 md/ 渲染生成，是推荐的可选形式
- md 模式下不生成 html/

### 形式目录（.xxx/）

每个 `.xxx/` 形式目录完全自包含，可独立部署：

| 形式目录 | 说明 | 派生方式 |
|----------|------|----------|
| `.full/` | 完整版（默认），敏感内容用 `<details>` 隐藏 | 基准，直接从 md 渲染 |
| `.pure/` | 纯净版 | 从 `.full/` 移除整个 `<details>` 敏感块 |
| `.parts/` | 分块输出 | 完整内容打碎为分块，属"敏感内容大赏"模块 |
| `.whole/` | 完整聊天记录长页 | 可选，默认不生成 |
| `.sensitive/` | 敏感内容大赏 | 默认仅放置标记；用户开启后升级为完整内容 |

**集合关系：**
- `.full = .pure ∪ 隐藏的敏感块`
- `.pure = .full - 敏感块`

**独立部署规则：**
- 每个 `.xxx/` 各自包含 `.assets/`（CSS/JS）+ `resources/`（媒体文件）+ 页面
- **不跨形式目录引用** — `.full/` 的页面只引用 `.full/.assets/` 和 `.full/resources/`
- 迁移时以 `.xxx/` 为单位整体迁移，不丢失依赖文件

---

## 部署与迁移

每个 `.xxx/` 形式目录设计为完全自包含，支持以下部署场景：

### 场景 1：部署单个形式目录

将任意一个 `.xxx/` 目录复制到静态服务器即可正常运行。所有依赖（CSS/JS/媒体/页面）均在目录内部。

**示例：部署纯净版**
```
# 复制 .pure/ 到服务器
cp -r html/.pure/ /var/www/chat-analysis/

# 目录结构（服务器上）
/chat-analysis/
├── .assets/          ← CSS/JS（自包含）
├── resources/        ← 媒体文件（自包含）
├── index.html        ← 首页
├── analyse/          ← 统计分析页
├── report/           ← 综合报告页
├── topic/            ← 话题专题页
├── personality/      ← 人格画像页
├── adventures/       ← 历险日记页
└── more/             ← 精彩/神人发言页
```

**相对路径保证：** 目录内所有链接（`href`、`src`）均使用相对路径，迁移后无需修改。页面间链接（如首页 → 人格画像）使用 `personality/用户名.html` 格式，资源引用使用 `.assets/style.css` 格式。

### 场景 2：部署整个 html/ 目录

将整个 `html/` 目录部署到服务器，包含所有形式目录。

```
# 复制整个 html/ 到服务器
cp -r html/ /var/www/chat-analysis/

# 目录结构（服务器上）
/chat-analysis/
├── .full/            ← 完整版（独立）
├── .pure/            ← 纯净版（独立）
├── .parts/           ← 分块版（独立）
├── .whole/           ← 聊天记录长页（独立）
└── .sensitive/       ← 敏感内容大赏（独立）
```

**注意：** 各形式目录之间**没有交叉引用**。部署整个 html/ 不会比部署单个 `.xxx/` 多出任何依赖关系。用户可以根据需要选择部署哪些形式目录。

### 场景 3：选择性部署多个形式目录

用户可以选择部署任意组合的形式目录：

```
# 只部署完整版和纯净版
cp -r html/.full/ /var/www/chat-full/
cp -r html/.pure/ /var/www/chat-pure/
```

每个目录独立运行，互不依赖。

### 迁移检查清单

迁移前确认：
- [ ] 目标 `.xxx/` 目录包含 `.assets/` 子目录
- [ ] 目标 `.xxx/` 目录包含 `resources/` 子目录（如有媒体文件）
- [ ] 所有 HTML 文件中的链接使用相对路径（`href="..."` 而非 `href="/..."` 或 `href="file://..."`)
- [ ] 页面内锚点使用裸 `#id` 格式
- [ ] 部署后在浏览器中测试：首页 → 子页面跳转、CSS 加载、媒体显示、主题切换

### .output/scripts/ — 运行时脚本

- Phase 1 生成的解析器脚本
- Phase 3 生成的 HTML 渲染脚本
- 正式保留，供后续增量分析复用

### .output/.cache/ — 临时文件

- 任务过程中所有临时文件和中间产物统一写入此目录
- Phase 3 完成后清理

---

## 扁平模式 vs 分组模式

| 模式 | md/ 目录结构 | 适用场景 |
|------|-------------|----------|
| **扁平（默认）** | 所有 .md 直接在 md/ 下 | 文件数量少，结构简单 |
| **分组** | 按类型分子目录（analyse/、report/ 等） | 文件数量多，需要分类浏览 |

用户可在 Phase 0 选择。扁平模式下，各子代理输出文件名加类型前缀避免冲突：
- personality-agent → `personality-张三.md`
- adventures-agent → `adventures-张三.md`
- analyse-agent → `analyse-overview.md`
- report-agent → `report-overview.md`
- brilliant-agent / legendary-agent → 无需前缀

---

## 形式目录列表

所有 `.xxx/` 形式目录在 `html/` 下并列，按字母序排列：

```
html/
├── .full/          — 完整版（默认）
├── .parts/         — 分块输出（可选）
├── .pure/          — 纯净版
├── .sensitive/     — 敏感内容大赏（可选）
└── .whole/         — 完整聊天记录长页（可选）
```

打开 `html/.full/index.html` 即可浏览全部内容。
