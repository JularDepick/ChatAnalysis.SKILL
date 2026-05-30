---
name: chat-analysis
description: >
  This skill should be used when the user asks to "analyze chat records",
  "analyze chat logs", "analyze conversation history", "做聊天记录分析",
  "分析聊天记录", "聊天数据分析", "对话记录分析", or mentions parsing
  messaging app exports (QQ, WeChat, Telegram, Discord, WhatsApp, etc.).
  Provides a complete pipeline from raw chat logs to structured analysis
  reports. Core output is Markdown; optional HTML rendering is recommended.
compatibility: Recommends Python 3.8+ (scripts generated at runtime)
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent]
argument-hint: <chat-file-or-dir> [--mode group|private] [--focus-users uin1,uin2] [--output <dir>] [--output-style html|md|all]
metadata:
  author: chat-analysis-skill
  category: data-analysis
  tags: [chat, nlp, analysis, report, html, group-chat, private-chat, jsonl]
---

# 聊天记录分析

A complete pipeline for deep analysis of chat/messaging records. Supports both **private chat** (1v1) and **group chat** (multi-user) scenarios. Parses structured chat logs, extracts multi-dimensional statistics, generates topic reports, personality profiles. Core output is Markdown; optional HTML rendering is recommended.

## 范围

本 SKILL 只负责：**输入 → 分析 → 输出**，最多附加数据清洗。

**不负责：**
- 聊天记录的导出、提取、抓取
- 聊天平台 API 调用
- 原始数据的获取

用户需自行从聊天平台获取原始数据，并提供明确的输入数据本体（文件路径或目录路径）。

**关键规则（任务启动时必须提醒用户）：**
- 本 SKILL 会消耗大量 TOKEN（动辄百万），请确保用户能承担再选择继续使用
- 不要在任务过程中执行 git 操作（commit/push/merge 等），除非用户明确要求
- 一次任务的所有数据必须来自**同一个私聊/群聊**，禁止跨聊天混合数据
- 如需分析多个聊天，应分别启动独立任务
- 允许用户在任务中途追加新的聊天记录数据（如新导出的批次文件），但必须确认追加数据属于同一聊天

**增量数据支持：** 允许用户指定新的聊天记录内容，将其整合进现有输出中。追加数据时，Agent 需确认新旧数据来自同一聊天，然后重新运行统计脚本并更新相关报告。

## 支持场景

| Scenario | Description | Key Differences |
|----------|-------------|-----------------|
| **Private Chat** | 1-on-1 conversation | Relationship-focused, 2 sender profiles, reply chain analysis |
| **Group Chat** | Multi-user group | Social dynamics, N sender profiles, activity rankings, bot interaction, forward culture |

## 分析哲学

### 广度优先，深度并行

1. **先广泛提取维度/方面** — 在深入任何单一维度之前，先扫描全貌：统计指标、时间分布、内容特征、互动模式、媒体使用、话题分布、用户画像
2. **再并行深度分析** — 用子代理并行处理各维度的深度报告，每个代理专注一个领域

### 善用子代理

**原则：无论何时，开子代理总是最方便、最不阻塞的方式。** 能用子代理完成的任务，不要在主对话中同步执行。

- 报告生成 → 多个子代理并行（Phase 2 核心策略）
- 数据清洗/脚本执行 → 子代理运行，主对话继续配置
- SKILL 迭代 → 子代理维护 SKILL，主对话推进任务（反之亦然）
- 重复性工作（批量文件处理、格式转换）→ 子代理批量处理

主对话负责：决策、确认、用户交互、质量审查。子代理负责：执行、生成、转换、批量操作。

### 就事论事，理性优先

- **基于数据和证据** — 每个结论必须有聊天记录原文支撑
- **理性分析为主** — 客观描述可观察行为，感性解读为辅
- **不代入立场** — 不预设法律、道德、伦理立场做分析
- **用户全责** — 输出仅做敏感内容警告（md 引用块 / HTML 红边框），不兜底、不隐藏、不模糊内容，保持原文完整性

### 相对路径原则（T0）

**所有输出中的路径必须使用相对路径。** 这是 T0 级原则，适用于全部输出形式（md 和 html）：

- `<a>` 跳转链接 — 使用相对路径（如 `analyse/overview.md`、`../index.html`）
- 媒体引用 — 使用相对路径（如 `resources/images/xxx.jpg`）
- 导航栏/面包屑跳转 — 使用相对路径
- CSS/JS 引用 — 使用相对路径（如 `.assets/style.css`）
- **禁止**：绝对路径、`file://` 协议、硬编码域名

### 隐私保护

| 场景 | 真实身份数据 |
|------|-------------|
| **私聊** | 允许出现（双方身份已知） |
| **群聊** | 默认禁止，除非用户在任务启动前明确确认允许 |

群聊中的人物真实身份敏感数据（真名、备注名、手机号、地址等）必须脱敏处理（替换为 `[已脱敏]`），除非用户在 Agent 询问环节明确确认允许出现。

**脱敏检查：** Phase 3 完成后、输出交付前，扫描所有输出文件，确认无未授权的真实身份数据泄露。

### 局部视角，不以偏概全

- 聊天记录只是用户生活的片面切片
- 对任何用户的分析都只是局部的、缺乏现实生活数据的
- 使用限定性语言："从聊天中可观察到"、"在此场景中表现出"
- **主观性输出**（锐评、精彩发言、神人发言）需在贴切用户要求的前提下，避免以偏概全的论断

---

## 敏感内容处理

本节集中定义敏感内容的分类、默认处置流程和可选模块。各 Phase 中涉及敏感内容的具体操作均引用本节。

### 定义

| 类型 | 是否敏感 | 说明 |
|------|----------|------|
| **严重违反法律法规** | 是 | 媒体文件涉及未成年人、暴力犯罪、毒品、恐怖主义等 |
| **漏点色情** | 是 | 媒体文件暴露关键部位的色情内容 |
| 价值观不同 | 否 | 政治观点、宗教信仰、生活方式差异 |
| 软色情 | 否 | 媒体文件擦边、暗示性内容，但未暴露关键部位 |
| 无露出色情 | 否 | 色情内容但未暴露关键部位 |
| 争议性言论 | 否 | 不当言论、冒犯性表达，按一般敏感内容处理（md 引用块 / HTML 红边框警告） |

**默认仅针对媒体文件。** 文字内容一般不做敏感处理（争议性言论仅加红边框警告，不隐藏不脱敏）。默认敏感判断范围：图片、音频、视频等媒体文件。

**媒体文件的敏感性判断：** 媒体文件是否属于敏感内容，不能仅看文件本身，必须结合其在聊天记录中的上下文判断。同一张图片在不同语境下敏感性可能不同。判断时应参考：文件名、前后消息内容、发送者意图、讨论话题。

**敏感内容判断范围（Phase 0 必须询问用户）：**

> 敏感内容判断的范围是？
>
> 1. **仅媒体文件（默认）** — 只对图片、音频、视频做敏感判断，文字内容不隐藏
> 2. **所有内容** — 文字和媒体均做敏感判断，敏感文字同样用标记包裹隐藏

### 默认行为（无需用户提示）

发现严重违法/色情内容时，自动执行以下流程：

**Markdown 输出（核心）：** 敏感内容在 md 文件中以注释标记包裹，不直接隐藏：
```markdown
<!-- sensitive-start: 严重违法/色情 -->
敏感内容原文
<!-- sensitive-end -->
```
每条敏感内容独立标记，标记对之间一一对应。

**HTML 输出（从 md 渲染时）：** 渲染器将 `<!-- sensitive-start/end -->` 标记转换为 `<details>` 块：
```html
<details>
  <summary>⚠️ 敏感内容（严重违法/色情）— 点击展开</summary>
  <div class="danger-block">
    <!-- 敏感内容原文 -->
  </div>
</details>
```
**每条敏感内容独立收起/展开，互不影响。** 不得使用联动逻辑（展开一条即展开全部）。每个 `<details>` 块独立运作。

**形式目录派生：**
1. **`html/.completed-hidden/`** — 包含转换后的 `<details>` 块
2. **`html/.sensitive/`** — 仅放置敏感标记（来源、位置、类型），不存储完整内容
3. **`html/.pure/`** — 通过集合运算生成：从 `.completed-hidden/` 移除整个 `<details>` 敏感块
   - `.completed-hidden = .pure ∪ 隐藏的敏感块`，`.pure = .completed-hidden - 敏感块`
4. **任务结束后报告** — 向用户说明发现了哪些敏感内容、Agent 做了什么处理、如何查看各版本

### 可选模块：敏感内容大赏（默认关闭，不主动引导）

仅当用户**主动明确要求**时才开启：

1. 填充 `html/.sensitive/` — 从仅标记升级为完整内容摘录
2. 生成 `html/.completed-parts/` — 完整内容打碎为分块（纯净分块 + 敏感分块）
3. 用户可通过分块自由组合：纯净块 → 纯净版，全部块 → 完整版
4. md/ 中的 `<!-- sensitive-start/end -->` 标记保持不变，作为敏感内容的原始来源

**不主动引导用户开启此模块。** 仅在用户自行提出时响应。

### 一般敏感内容（非严重）

一般敏感内容（价值观不同、软色情、无露出色情、争议性言论等）— 保留在正文中，添加视觉警告：

1. **DO NOT** modify, redact, or censor the original chat text
2. **Markdown 中**：在相关内容前添加 `> ⚠️ 以下内容可能包含争议性言论` 引用块警告
3. **HTML 渲染时**：转为红边框警告 div（red border, warning icon）
4. **DO** paraphrase analysis rather than verbatim quoting sensitive portions
5. **DO** focus on communication pattern analysis rather than content endorsement
6. **DO** frame analysis academically ("从沟通行为分析角度...")

---

## 工作流概览

```
workdir/
├── row/               ← 原始数据（用户放置或迁移至此）
│
├─► Phase 0: Directory Init + Input Detection & Configuration
│
├─► Phase 1: Parse & Stat → .output/chat_stats.json
│
├─► Phase 2: Deep Analysis → .output/md/*.md
│
└─► Phase 3: Output Rendering（读取 md/，渲染到 html/）
                → .output/html/（推荐，从 md 渲染）
                  ← .output/md/（Phase 2 的核心输出，Phase 3 读取）
                  ├─ .completed-hidden/             — 完整版（默认，敏感内容<details>隐藏）
                  ├─ .completed-parts/       — 分块输出（可选）
                  ├─ .completed-whole-page/  — 完整聊天记录长页（可选）
                  ├─ .pure/                  — 纯净版（移除敏感块）
                  └─ .sensitive/             — 敏感标记/大赏（可选）
```

---

## Phase 0：输入检测与配置

初始化工作目录，检测数据格式，收集用户配置。完成后进入 Phase 1。

### 0.0 目录初始化

在开始任何分析之前，检查并整理工作目录结构，使其符合标准布局：

**标准结构：**
```
workdir/
├── row/       ← 原始数据
└── .output/   ← 所有输出
```

**检查流程：**

1. 检查 `row/` 是否存在且包含原始数据
2. 检查 `.output/` 是否存在
3. 如果工作目录不符合标准结构，执行迁移：

| 情况 | 操作 |
|------|------|
| 原始数据直接在 workdir 根目录 | 创建 `row/`，将原始数据移入 |
| 原始数据在其他子目录 | 创建 `row/`，将原始数据移入或软链接 |
| `row/` 已存在且有数据 | 跳过，直接使用 |
| `.output/` 已存在 | 跳过，保留现有输出（增量模式） |
| `.output/` 不存在 | 创建 `.output/` 及子目录（含 `.cache/`） |

**迁移时必须：**
- 先向用户确认迁移计划，获得同意后执行
- 不修改原始数据内容，仅移动位置
- 迁移后验证数据完整性（文件数量、大小一致）
- 如果用户指定的输入路径已经是 `row/`，跳过迁移

**启动提醒（必须在任务开始时告知用户）：**
> 本次分析的所有数据必须来自同一个私聊/群聊。
> 如需分析多个聊天，请分别启动独立任务。
> 任务中途可追加新的聊天记录数据，但需确认属于同一聊天。
>
> ⚠️ 风险提示：分析输出基于聊天记录文本数据，AI 生成内容不代表用户真实主观感受。
> 输出中可能包含敏感或争议性内容，所有内容仅做警告提醒，不兜底、不隐藏、不模糊。
> 使用本 SKILL 生成的全部内容，由用户自行承担责任。

### 0.1 格式检测

**不做格式限制。** 只要数据中包含时间、人物、对话三项要素，即可分析。脚本不是通用的，而是根据实际输入数据运行时编写。

以下是两种常见模式，Agent 应根据实际数据灵活适配：

#### 文本格式（单文件）
Plain text chat logs with date headers and sender prefixes:
```
## 2026-01-01 09:30
用户A：早上好
用户B：早
用户B：[图片]
```

#### JSONL 格式（批量文件目录）
Structured JSON Lines, typically from QQ Chat Exporter (QCE) or similar tools:
```
message_batches/
├── batch_000001.jsonl
├── batch_000002.jsonl
└── ...
```

Each line is a JSON object:
```json
{
  "id": "xxx",
  "seq": "123",
  "timestamp": 1779257844000,
  "time": "2026-05-20T06:17:24.000Z",
  "sender": {
    "uid": "u_xxx",
    "uin": "123456",
    "name": "张三",
    "groupCard": "张三",
    "title": "xxx"
  },
  "type": "text",
  "content": {
    "text": "消息内容",
    "html": "<msg>...</msg>",
    "elements": [...],
    "resources": [...],
    "mentions": []
  },
  "recalled": false,
  "system": false
}
```

**Detection method:** If the input path is a directory containing `*.jsonl` files, use JSONL parser. If it's a single file, use text parser.

### 0.2 聊天模式检测

| Indicator | Group Chat | Private Chat |
|-----------|-----------|--------------|
| Sender count | >2 unique senders | Exactly 2 senders |
| System messages | Join/leave notifications | Rare |
| @mentions | Common | Rare |
| Forward messages | Common | Rare |
| Bot interactions | Common (bot accounts) | Rare |

**Auto-detection:** Count unique senders in first 500 messages. If >2, classify as group chat.

**Manual override:** User can specify `--mode group` or `--mode private`.

### 0.3 用户配置

Ask the user (or detect from data):

1. **Chat platform** — QQ, WeChat, Telegram, Discord, WhatsApp, or auto-detect
2. **Chat mode** — group chat or private chat
3. **Focus users** — UINs or names of users to analyze in depth (typically the user's own accounts)
4. **Output directory** — where to write results
5. **Output style** — `md`（仅核心）、`html`（核心+渲染，推荐）、`all`（核心+渲染+保留md）
6. **HTML 风格主题** — 选择输出 HTML 的视觉风格（默认 Apple）：
   - **Apple** — 苹果风格，简洁明亮，蓝色强调色
   - **Nord** — 北欧冷色调，柔和低饱和
   - **Dracula** — 暗紫风格，深色背景，紫色强调色
   - **Gruvbox** — 复古暖色调，怀旧质感
   - **Solarized** — 平衡低对比度，护眼配色

   仅输出模式为 `html` 或 `all` 时询问。用户也可通过命令行 `--theme <name>` 直接指定。
7. **HTML 页面布局** — 选择页面排版结构（默认 Standard）：
   - **Standard** — 单栏居中，最大宽度 880px，面包屑导航，适合大多数场景
   - **Wide** — 宽屏布局（1200px），右侧悬浮目录导航（TOC），适合长文报告
   - **Magazine** — 杂志风，首页双栏卡片网格，内页标准单栏，适合群聊多主题展示
   - **Compact** — 紧凑密集布局，更小间距和字号，适合大量内容快速浏览

   仅输出模式为 `html` 或 `all` 时询问。用户也可通过命令行 `--layout <name>` 直接指定。
8. **完整聊天记录长页（可选）** — 是否生成完整聊天记录的单页 HTML（默认不生成）：
   - 敏感内容用 `<details>` 隐藏（与 `.completed-hidden/` 同逻辑）
   - 媒体文件用 `<a>` 链接插入（不内嵌），避免页面过大
   - 消息按时间合并显示（5 分钟分组，详见 Phase 3 Time Merging 段落）
   - 含页脚链接（如用户允许）

   `.completed-whole-page/` 作为并列的 `.xxx/` 形式目录输出到 `html/` 下，与 `.completed-hidden/` 等平级。可独立部署。
9. **页脚技术支持链接（可选）** — 如果输出模式为 `html`，在开始输出前询问用户：
   > 本 SKILL 制作不易，是否允许制作方在你的输出页面底部加入 GitHub 技术支持链接？
   >
   > 链接内容：页脚显示「SKILL技术支持·ChatAnalysis」，指向本项目 GitHub 仓库。
   > 选择「允许」将在所有生成的 HTML 页面底部添加此链接；选择「不允许」则不添加。

   用户允许时，HTML 渲染阶段为所有页面添加页脚链接：`<footer><a href="https://github.com/JularDepick/ChatAnalysis.SKILL">SKILL技术支持·ChatAnalysis</a></footer>`。
   用户拒绝或输出模式为 `md` 时跳过此步骤。

10. **叙述人称** — 输出报告的叙事视角：
    - **第三人称（默认）** — 人称无关叙事，不出现"我"，使用用户名或"该用户"等中性称呼
    - **第一人称** — 用户指定自己在聊天中的身份（如"我"对应哪个昵称/UIN），输出中以此人称叙事

    默认采用第三人称，不涉及"我"的指定。仅当用户主动选择第一人称时，才在输出中明确"我"的指向。
11. **敏感内容判断范围** — 默认仅针对媒体文件：
    > 1. **仅媒体文件（默认）** — 只对图片、音频、视频做敏感判断，文字内容不隐藏
    > 2. **所有内容** — 文字和媒体均做敏感判断，敏感文字同样用标记包裹隐藏
12. **敏感内容大赏（可选）** — 用户主动要求时才开启（见 [敏感内容处理](#敏感内容处理)）
13. **最大并行子代理数** — Phase 2 并行生成报告时的子代理上限（默认 3）：
    > 子代理越多效率越高，但并行请求过多可能导致 API 阻塞或限流。
    > 建议：小数据（<5K 消息）2-3 个，大数据（>25K 消息）3-5 个。
    >
    > 希望最多同时运行多少个子代理？（默认 3，最大 9）

    用户选择后，Agent 在 Phase 2 按此上限分批调度子代理。
14. **头像嵌入（可选）** — 如果原始数据中包含用户头像（如 JSONL 的 avatar 字段），是否将头像嵌入报告：
    > 嵌入头像可增强人格画像和排行榜的视觉效果。
    > 头像将以固定尺寸显示（推荐 48×48px 或 64×64px）。
    >
    > 是否嵌入头像？（默认不嵌入）

    用户选择嵌入时，从 row/ 按需迁移头像文件到各 `.xxx/resources/avatars/`，在人格画像和排行榜中以 `<img>` 引用。

For group chats, additionally ask:
15. **TOP N ranking** — how many top users to profile individually (default: 10, max: 20)
16. **精彩发言 / 神人发言** — 是否生成精彩发言/神人发言展示页
17. **XXX历险记** — 是否生成"群聊历险日记"模块（默认不输出）：
    - 以不同聊天用户的第一人称视角，撰写群聊历险日记
    - 输出到 `SpeakAs/` 目录，每个用户一个文件
    - 需严格注明 AI 生成内容不代表具体用户的主观感受
    - 用户选择开启后，需指定撰写哪些用户（默认 TOP N）

### 0.4 媒体资源检测

扫描原始数据目录，检查是否存在媒体文件（图片、视频、文件等）。常见位置：`resources/images/`、`resources/files/`、`data/`。

如果发现媒体文件，**必须询问用户**选择整合策略：

> 在原始数据中发现了 X 张图片、Y 个文件（共 Z MB）。
> 是否要将这些媒体文件整合到报告输出中？
>
> 选项：
> 1. **全部使用** — 按需迁移到输出形式目录的 resources/ 下，在报告中引用
> 2. **选择性使用** — 仅迁移精彩发言/神人发言中涉及的媒体
> 3. **不使用** — 仅输出文本分析，不引用媒体

**含文本图片识别建议：** 如果发现可能含文本的图片（聊天截图、长文图片、文字表情包），主动提醒用户使用 OCR 工具提取文本后补充。不阻塞任务，建议先完成现有分析。

**媒体引用方式（按输出格式区分）：**
- **HTML 输出**：使用 `<img>`、`<audio>`、`<video>` 标签，固定一个维度（推荐 360px），使用相对路径
- **Markdown 输出**：使用 `![描述](resources/xxx.jpg)` / `[文件名](resources/xxx.zip)` 语法，使用相对路径

所有媒体引用**必须使用相对路径**（如 `resources/images/xxx.jpg`），不使用绝对路径或 `file://` 协议。详见 `references/script_writing_guide.md` 3.7 节。

**善用原始媒体：** 分析报告（人格画像、精彩发言、话题专题等）中引用聊天原文时，如果原始消息包含图片/音频/视频，应主动将对应的媒体文件嵌入报告中作为证据和展示素材，而非只引用文字。媒体是聊天的重要组成部分，图文并茂的报告远比纯文本质感更高。

**媒体迁移路径：** 媒体文件从 `row/`（原始数据）按需直接迁移到最终输出形式目录的 `resources/` 下，不经过 md/ 中转。只有实际被报告引用的媒体文件才迁移，不要全量复制。各 `.xxx/` 形式目录各自独立迁移，不共享 resources/。

**大文件过滤：** 默认过滤超过 10MB 的文件。用户选择「全部使用」时，先报告大文件数量，询问是否调整阈值（5MB/2MB）。允许用户选择压缩媒体质量。

**输出体积控制：** 媒体整合后报告预估总大小。如需控制体积，按优先级舍弃：topic/report 装饰图 → analyse 图 → 保留 personality/brilliant 核心证据图 → 按大小排除。仅移除媒体引用（HTML `<img>` 或 Markdown `![]()`），不删除 `resources/` 文件。

---

## Phase 1：解析与统计

解析原始数据，提取多维统计，输出 `chat_stats.json`。完成后进行数据验证和输出确认。

### 1.1 脚本生成

**This SKILL does not ship pre-built scripts.** Instead, scripts are written at runtime in the user's working directory based on the actual data format and requirements.

Read `references/script_writing_guide.md` for the complete script-writing methodology, including:
- Parser script patterns (text format and JSONL format)
- Statistics computation patterns
- Platform-specific adaptations
- Error handling conventions

**Script generation workflow:**
1. Read 20-50 lines of the actual data to understand its exact format
2. Write the parser script tailored to that format
3. Write the script to `<workdir>/.output/scripts/`
4. Run and verify the script produces correct output
5. Iterate if parsing errors are detected

**Key principle:** Every chat export has subtle format differences. The script must be adapted to the actual data, not the other way around.

### 1.2 文本格式解析器

For single-file text chat logs, write a parser script following the patterns in `references/script_writing_guide.md`. The parser must:

1. Auto-detect date header patterns from the first 100 lines
2. Auto-detect sender patterns
3. Classify message types (text, image, card, system, reply)
4. Extract timestamps
5. Compute all statistics (see Section 1.3)

### 1.3 JSONL 格式解析器

For JSONL batch directories, write a parser script following the JSONL patterns in `references/script_writing_guide.md`. The parser must:

1. Load all `.jsonl` files from the batch directory
2. Parse nested sender/content objects
3. Convert timestamps (ms → datetime)
4. Resolve sender names (groupCard > name > nickname)
5. Classify message types (text, image, forward, reply, system, recalled)
6. Compute per-user statistics with UIN-based grouping
7. Extract Chinese words via n-gram (no jieba dependency)
8. Extract English words
9. Match keyword categories
10. Detect conversation rounds (30-min gap threshold)
11. Compute reply speed
12. Analyze media (images, forwards, replies)

### 1.4 统计输出

Both parsers output `chat_stats.json` with these sections:

| Section | Key Fields |
|---------|-----------|
| `basic` | total_messages, valid_messages, date_range, active_days, senders, sender_names, message_types |
| `time` | hourly, weekday, monthly, daily, daily_per_sender, hourly_per_sender |
| `content` | chinese_word_freq, english_word_freq, avg_message_length, message_length_distribution |
| `interaction` | total_rounds, avg_round_length, round_length_distribution, reply_speed |
| `media` | total_images, total_forwards, total_replies, image_text_ratio |
| `keywords` | keyword_counts_by_category |
| `keywords_by_sender` | per-user keyword profile |
| `user_stats` | per-user detailed stats (messages, active_dates, peak_hours, reply_targets, keyword_profile) |
| `top10_users` | list of top 10 UINs by message count |

### 1.5 活动断层检测（群聊）

For group chats with many participants, identify "activity cliffs" — points where the next user's message count drops dramatically:

```python
def detect_activity_cliffs(senders_sorted_by_count):
    """
    Find cliffs where:
    1. Drop percentage > 15% AND absolute drop > 80 messages
    2. OR drop is 2.5x larger than average of next 3 drops
    Returns: list of (rank_before_cliff, user_name, count, drop_pct)
    """
```

Report the cliffs to the user with:
- Rank before cliff
- User name and message count
- Drop percentage
- Tier boundaries (users above cliff = Tier 1, between cliffs = Tier 2, etc.)

### 1.6 数据完整性验证

After parsing, verify:
- [ ] Total message count matches expectations
- [ ] Date range is correct
- [ ] Sender names are properly distinguished
- [ ] No parsing errors (zero-message days that shouldn't be zero)
- [ ] Focus users are present in the data
- [ ] Message type distribution is reasonable

### 1.7 输出前确认

Phase 1 完成后、Phase 2 开始前，Agent 必须向用户报告数据扫描结果并确认输出架构：

**报告内容：**
- 数据概况（消息数、日期范围、发言者数）
- 是否发现敏感内容（如有，说明类型、数量、大致位置）
- 是否发现媒体文件

**询问用户选择输出架构（仅 HTML 渲染时适用，md 模式跳过此步）：**

> 数据扫描完成，准备生成报告。请选择 HTML 输出架构：
>
> 选项：
> 1. **完整版 + 纯净版（默认）** — `.completed-hidden/`（敏感内容用<details>隐藏）+ `.pure/`（移除敏感块）
> 2. **仅纯净版** — 只生成 `.pure/`，不含敏感内容
> 3. **仅完整版** — 只生成 `.completed-hidden/`，敏感内容可手动展开
> 4. **含敏感内容大赏** — 完整版 + `.sensitive/`（填充完整内容）+ `.completed-parts/`（分块）

**默认采用选项 1。** 如果数据中无敏感内容，直接告知用户无需选择，按默认输出生成。md 模式下不生成 html/ 目录，无需选择架构。

---

## Phase 2：深度分析报告

并行生成多维度分析报告：统计分析、综合报告、话题专题、人格画像，以及可选的精彩发言和群聊历险日记。

### 2.1 输出结构

标准工作目录结构：

```
workdir/                          — 工作目录
├── row/                          — 用户提供的原始数据（聊天文件/JSONL 目录）
└── .output/                      — 输出目录（隐藏目录）
    ├── chat_stats.json           — Phase 1 统计数据
    ├── scripts/                  — 运行时生成的辅助脚本
    ├── README.ai.md              — Agent 复用指南
    ├── .cache/                   — 临时文件/中间产物（任务完成后清理）
    ├── md/                       — 清洗后的 Markdown（核心输出）
    │   ├── *.md                  — 默认扁平放置（除非用户要求分组）
    │   ├── analyse/              — 用户要求分组时：5 篇统计分析
    │   ├── report/               — 用户要求分组时：4 篇综合报告
    │   ├── topic/                — 用户要求分组时：N 篇话题专题
    │   ├── personality/          — 用户要求分组时：每用户画像
    │   ├── brilliant.md          — 精彩发言展示页
    │   ├── legendary.md          — 神人发言展示页
    │   └── SpeakAs/              — 群聊历险日记（可选）
    │       └── <user>.md         — 每用户历险日记（声明嵌入页首）
    └── html/                         — HTML 输出（每个 .xxx/ 可独立部署）
        ├── .completed-hidden/               — 完整版（默认，敏感内容用<details>隐藏）
        │   ├── .assets/              — CSS/JS
        │   ├── resources/            — 媒体文件
        │   ├── index.html            — 首页（排行榜）
        │   ├── analyse/              — 统计分析 HTML
        │   ├── report/               — 综合报告 HTML
        │   ├── topic/                — 话题专题 HTML
        │   ├── personality/          — 人格画像 HTML
        │   ├── SpeakAs/              — 历险日记 HTML
        │   └── more/                 — 单页玩法
        │       ├── brilliant.html
        │       └── legendary.html
        ├── .completed-parts/         — 分块输出（可选）
        │   ├── .assets/
        │   ├── resources/
        │   └── parts/               — 分块页面
        ├── .completed-whole-page/    — 完整聊天记录长页（可选）
        │   ├── .assets/
        │   ├── resources/
        │   └── chat.html
        ├── .pure/                    — 纯净版（移除敏感块）
        │   ├── .assets/
        │   ├── resources/
        │   ├── index.html
        │   ├── analyse/
        │   ├── report/
        │   ├── topic/
        │   ├── personality/
        │   ├── SpeakAs/
        │   └── more/
        └── .sensitive/               — 敏感标记 / 敏感内容大赏（可选）
            ├── .assets/
            └── resources/
```

**关键约定：**
- `row/` 存放原始数据，不做任何修改
- `.output/` 为隐藏目录，所有产出均在此下
- **`md/` 是核心清洗输出**，所有分析报告首先以 Markdown 形式写入 md/
- **`html/` 从 md/ 渲染生成**，是推荐的可选形式。md 模式下不生成 html/
- 媒体文件迁移规则见 Phase 0.4
- `md/` 默认扁平放置，除非用户在 Phase 0 要求按分类分子目录（此时生成 analyse/、report/、topic/、personality/、SpeakAs/ 等子目录）
- 所有 `.` 开头的目录在 `html/` 下并列，按字母序排列：`.completed-hidden/`、`.completed-parts/`、`.completed-whole-page/`、`.pure/`、`.sensitive/`
- 报告内容不直接输出到 `html/` 根目录，全部放入 `.completed-hidden/`（或其他形式目录）
- `chat_stats.json` 在 `.output/` 根目录，供所有报告引用
- 每个 `.xxx/` 形式目录完全自包含：`.assets/`（CSS/JS）+ `resources/`（媒体文件）+ 报告页面，可独立部署
- 临时文件和中间产物统一放入 `.output/.cache/`，任务完成后清理

### 2.2 子代理调度策略

使用并行后台子代理生成报告。每个子代理将 Markdown 报告文件写入 `.output/md/`。

#### 私聊模式（2 人）

分批调度 3-4 个子代理（受 Phase 0 配置的子代理上限约束，超出时分批执行）：

| 子代理 | 输出文件 | 职责 |
|--------|---------|------|
| analyse-agent | analyse/ 下 5 个文件 | 统计分析深度报告 |
| report-agent | report/ 下 4 个文件 | 综合报告 |
| topic-agent | topic/ 下 N 个文件 | 话题专题报告 |
| personality-agent | personality/ 下 2 个文件 | 双方人格画像 |

#### 群聊模式（多人）

分批调度 5-9 个子代理（受 Phase 0 配置的子代理上限约束，超出时分批执行）：

| 子代理 | 输出文件 | 职责 |
|--------|---------|------|
| analyse-agent | analyse/ 下 5 个文件 | 统计分析（含交叉分析） |
| report-agent | report/ 下 4 个文件 | 多维综合报告 |
| topic-agent | topic/ 下 N 个文件 | 话题专题（典型 9-12 篇） |
| personality-batch-A | TOP 1-5 用户画像 | 排名前 5 的人格画像 |
| personality-batch-B | TOP 6-10 + 焦点用户画像 | 排名 6-10 及焦点用户画像 |
| personality-batch-C | TOP 11-N 用户画像（如需要） | 额外人格画像 |
| brilliant-agent | brilliant.md | 精彩发言展示页（含梗文化深度分析） |
| legendary-agent | legendary.md | 神人发言展示页（含神人解析） |

**调度规则：**
- **并行子代理数不超过用户在 Phase 0 配置的上限（默认 3）。** 超出时分批调度：先启动一批，待完成后再启动下一批
- 所有子代理使用 `run_in_background: true`
- 每个子代理必须自包含上下文（聊天文件路径、统计数据路径、输出路径）
- 不得为同一任务重复启动子代理
- 通过任务通知监控子代理完成状态
- If an agent is rejected as "high risk", relaunch with adjusted prompt (see Section 2.9)

### 2.3 写作规范（所有子代理必须遵守）

每篇报告**必须**：
1. 使用表格呈现结构化数据
2. 以代码块引用原始聊天记录作为证据
3. 引用含媒体的消息时，嵌入实际媒体文件 — Markdown 使用 `![](resources/xxx.jpg)` 语法，HTML 渲染时转为 `<img>`/`<audio>`/`<video>` 标签（固定一个维度，推荐 360px）
4. 引用具体统计数据（数量、百分比、日期）
5. 承认局限性（单一视角、仅文本、时间有限）
6. 使用与聊天内容相同的语言撰写
7. 每个文件 ≥ 300 行，追求深度而非广度
8. 阅读聊天文件的 6-10 个不同片段（不要只看开头）
9. 对照 chat_stats.json 抽检数据
10. 遵循 Phase 0 配置的叙事视角：
    - **第三人称（默认）**：使用"该用户"、用户名、"发言者"等中性称呼，全文不出现"我"
    - **第一人称**：用户指定的焦点用户以"我"叙事，其他用户使用用户名

每篇报告**禁止**：
1. 在无聊天记录证据的情况下做出论断
2. 推测超出可观测范围的私人生活
3. 在示例中包含真实手机号、密码等敏感个人信息
4. 撰写空洞的总结 — 每一段都必须有数据或引文支撑

### 2.4 话题检测方法

采用两阶段方法识别话题：

**阶段 A — 发现：** 阅读 8-10 个均匀分布在时间范围内的片段，记录话题、关键词、话题发起者。

**阶段 B — 分类：** 从发现的话题构建关键词列表，然后全量扫描：

| 话题分类 | 示例关键词 |
|---------|-----------|
| 科技/AI | AI, ChatGPT, code, program, VPN, server, GitHub, python, api |
| 娱乐 | game, anime, video, movie, music, B站, 抖音 |
| 学业/职场 | exam, homework, GPA, professor, class, 考试, 作业, 绩点 |
| 社交/关系 | friend, classmate, dating, family, 同学, 朋友, 室友 |
| 日常生活 | food, weather, sleep, commute, shopping, 吃, 喝, 食堂 |
| 情绪 | happy, sad, stressed, lonely, emo, 开心, 难过, 焦虑 |
| 二次元/游戏 | 二次元, 番剧, 原神, 崩坏, 王者, steam |
| Bot 互动 | 老婆, 抽取, 市场, 银币, 表情合成 |

根据实际聊天内容调整关键词列表。群聊中 Bot 互动模式是重要的话题分类。

### 2.5 人格画像规则

撰写人格画像报告时：

**私聊模式（2 人）：**
- 聚焦关系动态、沟通模式、情感交流
- 双方并排对比
- 包含关系随时间的演变

**群聊模式（多人）：**
- 聚焦个人沟通风格、社交角色、影响力模式
- 分析 Bot 互动模式（老婆收集、抽卡系统等）
- 追踪回复目标以绘制社交关系图
- 识别角色原型：发起者、回应者、潜水者、Bot 用户、梗传播者、话题驱动者
- 为每个用户计算：消息数、文字/图片比、平均消息长度、活跃天数、高峰时段、主要回复对象、关键词画像

**通用规则：**
- 所有观察必须基于可验证的聊天行为
- 每个论断必须附带原始聊天记录作为证据
- 使用限定性语言："从聊天中可观察到"、"在这段关系中表现出"
- 以"局限性声明"段落结尾

### 2.6 精彩发言 / 神人发言展示页（群聊）

群聊模式下生成两个独立展示页：

#### 精彩发言展示页 (brilliant.md)
- 20-30 条精彩发言/对话
- 分类：神回复、经典对话、接龙/刷屏、梗传播、犀利吐槽、荒诞对话
- 每条包含：原始聊天记录、上下文说明、幽默/梗文化深度解析
- 保留完整上下文（过长时先总结因果再展示核心）
- 附录："群内高频梗速查表"

**推荐功能扩展（可选，向用户提议）：**
- 群内高频梗速查表 — 梗的起源、演变、使用场景
- 精彩内容集锦 — 按主题/时间/用户分类的精华摘录
- 深度点评/锐评 — 对群聊文化、社交现象的批判性分析
- 时间线大事记 — 群聊重要事件的 chronological 记录
- 社交网络图 — 基于回复关系的用户互动可视化

#### 神人发言展示页 (legendary.md)
- 10-20 条神人/传奇发言
- 每条包含：原始聊天记录、5 星评级、详细"神人解析"
- 聚焦：抽象文学、复制粘贴、戏剧独白、哲学长文、病毒传播

**两个页面通用规则：**
- 高风险内容（价值观不当、违背道德）添加警告指示器（md 中为引用块警告，HTML 渲染时为红边框 div）
- 仅警告 — 不审查、不修改内容
- 保持原始文本完整性
- 有效利用媒体资源（图片、转发消息）

### 2.7 XXX历险记 — 群聊历险日记（可选，默认不生成）

以不同聊天用户的第一人称视角，撰写群聊历险日记。这是一个创意性附加玩法，**默认关闭**，需用户在 Phase 0 主动开启。

#### 输出结构

```
md/SpeakAs/
├── <用户A>.md           — 用户A视角的历险日记
├── <用户B>.md           — 用户B视角的历险日记
└── ...
```

#### 内容规范

**AI 生成声明（嵌入每篇日记页首，不单独成页）：**

每篇历险日记开头必须嵌入以下声明（作为页面内容的一部分，不是独立文件）：

> ⚠️ AI 生成声明：本文由 AI 基于群聊记录生成，以 [用户名] 的第一人称视角撰写。
> 文中"我"不代表该用户的真实主观感受、想法或立场，仅供参考娱乐。

**每篇历险日记的写作要求：**

1. **第一人称叙事** — 以该用户的视角写"我"，但必须在文首重复声明
2. **基于真实聊天事件** — 日记中提到的群聊事件必须有聊天记录依据
3. **时间线忠实** — 按照聊天记录中的时间顺序组织叙事
4. **引用原文** — 关键事件引用原始聊天记录（代码块格式）
5. **创意但不虚构** — 可以用文学化语言描述场景和心理活动，但不能捏造未发生的事件
6. **文首声明** — 每篇日记开头嵌入 AI 生成声明（见上方"AI 生成声明"段落），不单独成页

**日记内容结构建议：**

| 章节 | 内容 |
|------|------|
| 开篇 | 入群初印象，第一次发言，对群聊的第一感觉 |
| 日常 | 典型的一天——什么时候看群、什么时候发言、关注什么话题 |
| 高光时刻 | 该用户参与的精彩对话、名场面、被群友记住的时刻 |
| 社交关系 | 与哪些群友互动最多、印象最深的群友、有趣的互动 |
| 梗与文化 | 该用户参与传播的梗、对群文化的贡献 |
| 尾声 | 对群聊的整体感受、未来期待 |

#### 撰写范围

- 默认撰写 TOP N 用户（与 Phase 0 配置一致）
- 用户可指定额外用户或排除特定用户
- 私聊模式下不适用（无意义）

### 2.8 高风险内容处理

**一般敏感内容** — 保留在正文中，添加视觉警告（详见 [一般敏感内容（非严重）](#一般敏感内容非严重)）。

**严重违法/色情内容** — 按 [默认行为（无需用户提示）](#默认行为无需用户提示) 执行：
- Markdown：以 `<!-- sensitive-start/end -->` 标记包裹敏感内容
- HTML 渲染时：标记转为 `<details>` 块，`.completed-hidden/` 保留完整但隐藏，`.sensitive/` 放置标记，`.pure/` 移除敏感块
- 用户开启"敏感内容大赏"时：`.sensitive/` 填充完整内容，额外生成 `.completed-parts/` 分块

**任务结束后必须向用户报告：** 发现了哪些敏感内容、Agent 做了什么处理、如何查看各版本。

### 2.9 子代理拒绝恢复

如果子代理被安全过滤器拒绝：

1. 调整提示词，强调：
   - 学术/分析目的
   - 聚焦沟通行为分析
   - 转述敏感内容而非逐字引用
   - 以观察性研究框架呈现
2. 缩小范围（每批更少的用户）
3. 使用调整后的提示词重新启动
4. 如果仍被拒绝，在主对话中手动写入文件

---

## Phase 3：输出渲染与清理

Phase 2 生成的 md/ 是核心输出。如用户选择 HTML 模式，从 md 渲染生成 html/。最后执行清理和交付。

### 3.1 输出模式

md/ 始终生成，是核心清洗输出。html/ 从 md 渲染而来，是推荐的可选形式。

| Mode | Description | Behavior |
|------|-------------|----------|
| `md` | 仅核心输出 | 生成 md/ + chat_stats.json，不渲染 HTML |
| `html` (default, 推荐) | 核心 + 渲染 | 生成 md/，再从 md 渲染 html/（`.completed-hidden/` 等形式目录） |
| `all` | 核心 + 渲染 + 保留 md | 同 `html`，额外保留 md/ 原始文件不被覆盖 |

### 3.2 HTML 渲染

按照 `references/script_writing_guide.md` 的模式编写 HTML 转换脚本。转换器读取 md/ 下所有 `.md` 文件，生成带样式的 HTML 页面。渲染过程中自动将 md 中的 `<!-- sensitive-start/end -->` 标记转换为 `<details>` 块。

```bash
python <workdir>/.output/scripts/md2html.py <workdir>/.output/md <workdir>/.output/html/.completed-hidden --stats <workdir>/.output/chat_stats.json
```

**页脚链接：** 如果用户在 Phase 0.3 中允许了技术支持链接，渲染时添加 `--footer-link` 参数：
```bash
python <workdir>/.output/scripts/md2html.py <workdir>/.output/md <workdir>/.output/html/.completed-hidden --stats <workdir>/.output/chat_stats.json --footer-link
```

**敏感内容派生：** 渲染完成后，从 `.completed-hidden/` 派生其他形式：
- `.pure/` — 移除 `.completed-hidden/` 中的 `<details>` 敏感块
- `.sensitive/` — 提取敏感标记
- `.completed-parts/` — 打碎为分块（用户开启时）

#### 设计系统

The HTML output supports multiple preset style themes and layout structures. Users choose in Phase 0.3.

**Preset Themes:**

| Theme | Character | Accent | Description |
|-------|-----------|--------|-------------|
| **Apple** (default) | Clean, minimal | `#007aff` blue | 苹果风格，简洁明亮 |
| **Nord** | Cool, muted | `#88c0d0` frost | 北欧冷色调，柔和低饱和 |
| **Dracula** | Dark, rich | `#bd93f9` purple | 暗紫风格，深色为主 |
| **Gruvbox** | Warm, retro | `#d79921` gold | 复古暖色调，怀旧质感 |
| **Solarized** | Balanced, soft | `#268bd2` cyan | 平衡低对比度，护眼配色 |

主题变量由 Agent 在运行时根据用户选择的主题生成，写入 `.assets/` 下的 CSS 文件中，不依赖预置文件。

**Preset Layouts:**

| Layout | Width | Nav | Description |
|--------|-------|-----|-------------|
| **Standard** (default) | 880px | 面包屑 | 单栏居中，适合大多数场景 |
| **Wide** | 1200px | 右侧 TOC | 宽屏布局，适合长文报告 |
| **Magazine** | 880px | 面包屑 | 首页双栏卡片网格，适合多主题展示 |
| **Compact** | 880px | 面包屑 | 紧凑密集，适合大量内容浏览 |

**Font Stack (all themes):**
```css
font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display",
             "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
```

**Theme Switching:**
- 用户选择的主题作为默认主题写入 CSS
- 页面仍保留浅色/暗色切换按钮（同一主题的两个变体）
- 主题选择通过 `data-theme` 属性和 CSS 变量实现

#### 主题切换

Every page includes a theme toggle button:
- Fixed position (top-right corner)
- Persists choice via `localStorage.getItem('theme')`
- Applies via `data-theme="dark"` attribute on `<html>`
- Initial load reads localStorage before rendering (no flash)

```javascript
(function() {
    var t = localStorage.getItem('theme') || 'light';
    if (t === 'dark') document.documentElement.setAttribute('data-theme', 'dark');
})();
```

#### 首页

**私聊模式**：首页大标题显示双方昵称（如"张三 & 李四 的聊天分析"），下方展示统计概览和报告分类链接。

**群聊模式**：首页大标题显示群聊名称（从数据中提取或用户指定），下方展示：
- **TOP N 活跃排行** — 柱状图可视化（纯 CSS，无 JS 库），前三名高亮强调色，点击跳转对应人格画像页，显示消息数和相对活跃百分比
- **报告分类链接** — analyse/、report/、topic/ 等分类入口
- **精彩发言 / 神人发言** — `more/brilliant.html`、`more/legendary.html` 以样式化按钮链接展示

#### 导航

Every page has breadcrumb navigation:
```
首页 › 分类名 › 文章标题
```

#### 同类链接收起

当某个页面中同类链接超过 5 个时，使用 `<details>` 标签收起，默认折叠，用户点击展开。适用于所有包含链接列表的页面（首页、子索引页等）。示例：
```html
<details>
  <summary>话题专题（N 篇）</summary>
  <ul>
    <li><a href="topic/xxx.html">话题1</a></li>
    <!-- ... -->
  </ul>
</details>
```

#### 响应式设计

- Max width: 由布局决定（Standard/Magazine/Compact 880px，Wide 1200px）
- Mobile breakpoint: 768px
- Tables scroll horizontally on mobile
- Leaderboard bars hidden on mobile

#### 时间合并

前端展示聊天记录时，合并连续消息的时间显示：5 分钟内无中断的消息归入同一个起始时间戳下。原始数据保留每条消息的完整时间，仅在渲染层合并。详见 `references/script_writing_guide.md` 3.14 节。

#### 技术架构

**技术栈严格限制为 HTML + CSS + JS。** 不使用任何外部框架、库、CDN 依赖。所有功能通过原生 HTML/CSS/JS 实现，允许使用浏览器存储（localStorage）。

**所有引用必须使用相对路径。** 不使用绝对路径、`file://` 协议或硬编码域名。

#### CSS/JS 拆分复用

每个 `.xxx/` 形式目录完全自包含，可独立部署：

```
html/
├── .completed-hidden/            ← 完整版（默认）
│   ├── .assets/           ← CSS/JS
│   ├── resources/         ← 媒体文件
│   ├── index.html
│   ├── analyse/
│   └── ...
├── .completed-parts/      ← 分块输出（可选）
│   ├── .assets/
│   └── resources/
├── .completed-whole-page/ ← 完整聊天记录长页（可选）
│   ├── .assets/
│   ├── resources/
│   └── chat.html
├── .pure/                 ← 纯净版（同样结构）
│   ├── .assets/
│   ├── resources/
│   └── ...
└── .sensitive/            ← 敏感标记/大赏（可选）
    ├── .assets/
    └── resources/
```

**拆分复用规则：**
- 报告内容不直接输出到 `html/` 根目录，全部放入 `.completed-hidden/`（或其他形式目录）
- 每个 `.xxx/` 形式目录各自包含 `.assets/` 和 `resources/`，完全自包含
- **每个 `.xxx/` 可独立部署** — 将任意一个 `.xxx/` 目录复制到任意静态服务器即可正常运行
- 迁移时以 `.xxx/` 为单位整体迁移，不会丢失依赖文件
- **不跨输出形式文件夹引用** — `.completed-hidden/` 的页面只引用 `.completed-hidden/.assets/` 和 `.completed-hidden/resources/`，不引用其他形式目录的文件

### 3.3 可移植性

所有输出文件（md 和 html）必须使用**相对路径**。

**Markdown 输出：**
- 文件间链接：`[统计分析](analyse/overview.md)`、`[返回首页](index.md)`
- 媒体引用：`![描述](resources/images/xxx.jpg)`
- 不使用绝对路径、`file://` 协议或硬编码域名

**HTML 输出：**
- Navigation links: `../index.html`, `../analyse/index.html`
- CSS/JS 引用: `../.assets/style.css`, `.assets/theme.js`
- User profile links: `personality/用户名.html`
- Resource references: `../resources/xxx.jpg`
- No absolute paths, no `file://` protocol, no hardcoded domains

### 3.4 验证

**Markdown 输出验证：**
- [ ] 所有 md 文件存在且内容完整
- [ ] 媒体引用路径指向实际存在的文件
- [ ] 统计数据与 chat_stats.json 一致
- [ ] 敏感内容标记（`<!-- sensitive-start/end -->`）成对出现
- [ ] 文件间链接路径正确

**HTML 渲染验证（HTML 模式时）：**
- [ ] All files are linked from index pages
- [ ] Navigation breadcrumbs work correctly
- [ ] Tables render properly
- [ ] Code blocks display with correct syntax highlighting
- [ ] Chinese characters render without garbling
- [ ] Theme toggle works and persists across pages
- [ ] Leaderboard links point to correct personality pages
- [ ] Standalone pages (more/brilliant.html, more/legendary.html) are accessible
- [ ] Responsive layout works on mobile viewport
- [ ] 同类链接过多时使用 `<details>` 收起（见 3.2 Design System）

### 3.5 文件清理

任务过程中所有临时文件和中间产物**从生成时即写入 `.output/.cache/`**，不混入正式输出目录。

| 保留（正式输出） | .cache/（临时/中间产物） |
|------------------|--------------------------|
| `row/` 原始数据（不做任何修改） | 子代理提取的中间文本片段 |
| `.output/md/` 核心 Markdown 输出 | 调试用的临时脚本、日志 |
| `.output/html/` 最终 HTML 输出 | 重复运行产生的冗余文件 |
| `.output/chat_stats.json` 统计数据 | 过期的中间 JSON/缓存 |
| `.output/scripts/` 运行时生成的脚本 | — |

**操作时机：** Phase 3 完成后、向用户报告之前，清理 `.cache/` 目录。

### 3.6 README.ai.md 生成

任务完成后，在 `.output/` 根目录生成 `README.ai.md`，内容包括：
- 分析概况（消息数、日期范围、发言者数）
- 输出架构说明（md/ 为核心输出，html/ 从 md 渲染生成）
- 目录结构说明（含 md/ 和 html/ 各形式目录）
- 数据来源和格式
- chat_stats.json 关键字段说明
- 焦点用户列表
- 复用指南（如何浏览、引用、二次分析）

此文件方便后继 Agent 快速了解输出结构，实现高效复用。

---

## 质量检查清单

在宣布分析完成之前：

### 数据完整性
- [ ] 统计数据与原始数据一致（抽检 3-5 个数据点）
- [ ] 各报告中消息数一致
- [ ] 日期范围准确
- [ ] 发送者名称正确解析

### 报告质量
- [ ] 每个主要论断都有聊天记录引文支撑
- [ ] 话题专题覆盖所有重要对话主题（>5% 的消息）
- [ ] 人格画像包含局限性声明
- [ ] 所有文件满足最低行数要求（≥300 行）
- [ ] 示例中无敏感个人数据泄露

### 输出质量
- [ ] Markdown 文件存在且内容完整（md 模式核心验证）
- [ ] 媒体引用路径有效
- [ ] 敏感内容标记成对出现
- [ ] HTML 在浏览器中渲染正确（HTML 模式时）
- [ ] 所有索引页链接到子页面
- [ ] 主题切换正常且跨页持久化
- [ ] 排行榜链接正确
- [ ] 响应式布局正常

### 文件组织
- [ ] 所有文件在正确的输出目录结构中
- [ ] chat_stats.json 存在且有效
- [ ] md/ 目录结构正确（核心输出）
- [ ] html/ 目录结构正确（HTML 模式时）
- [ ] `.cache/` 已清理
- [ ] 无孤立或缺失文件

---

## 自迭代规则

基于实际任务经验维护本 SKILL 时：

1. **禁止将任务数据写入 SKILL。** 不得包含实际聊天内容、用户名、UIN 或任何来自分析任务的真实数据。
2. **禁止泄露敏感内容。** 不得包含可识别真实用户的示例引文。
3. **禁止包含迭代日志。** SKILL 正文中不得包含变更日志、版本历史、更新记录或任何迭代修改痕迹。SKILL 应读起来像一次写成、始终正确。
4. **SKILL 纯净性是 t0 级。** SKILL 必须始终纯净、独立、自包含，仅反映最新状态。不引用旧版本，不写"原名 X"、"之前的做法是 Y"。SKILL 就是 SKILL — 不是自身的 diff。
5. **编码模式，而非实例。** 写"50 人以上的群聊适合分批生成人格画像"，而不是"往日种种群有 196 人"。
6. **更新方法论。** 如果新的分析技术被证明有效，将其加入参考文档。
7. **保持引用通用。** 所有示例应使用合成数据（用户A、用户B）。
8. **提供预设选项。** 当 SKILL 写"如果用户选择/要求"时，Agent 必须使用提问工具提供具体预设选项（2-4 个带清晰描述的选择），而非开放式提问。用户应点击选择，而非手动输入。

### 任务后提醒

**默认关闭，不主动提示。** 完成任务后，Agent 不得主动询问用户是否要迭代 SKILL。

仅当用户**主动明确要求**迭代 SKILL 时，Agent 才执行迭代，并遵守以下规范：
- 不将任务数据（聊天内容、用户名、UIN 等）写入 SKILL
- 不泄露敏感内容
- 不在 SKILL 中留下迭代日志/记录
- 仅编码通用模式和方法论
- 保持 SKILL 的纯净性和独立性

### 任务中 SKILL 迭代

当用户在任务进行过程中提出 SKILL 迭代/维护要求时：
- **用子代理维护 SKILL** — 将 SKILL 迭代工作交给后台子代理，主对话继续推进分析任务
- **或用子代理做任务** — 如果 SKILL 迭代更紧急，将分析任务交给后台子代理，主对话处理 SKILL

原则：不要让 SKILL 迭代阻塞任务进度，也不要让任务阻塞 SKILL 迭代。并行处理。

---

## 备注

### 性能指南
- 使用 `Agent` 工具配合 `run_in_background: true` 进行并行报告生成
- 每个子代理应阅读聊天文件的 6-10 个不同片段（不要只看开头）
- 大文件（>10K 行）使用 `Read` 的 `offset` 和 `limit` 参数
- JSONL 批量目录将所有消息加载到内存进行跨批分析
- 抽检子代理输出时阅读实际文件 — 不要信任子代理的摘要

### 规模指南
| 聊天规模 | 推荐子代理数 | 预计耗时 |
|---------|------------|---------|
| <1K 消息 | 2-3 子代理 | 2-5 分钟 |
| 1K-5K 消息 | 3-4 子代理 | 5-10 分钟 |
| 5K-25K 消息 | 5-9 子代理 | 10-20 分钟 |
| >25K 消息 | 9 子代理（分批调度） | 20-30 分钟 |

> 子代理数受 Phase 0 配置的上限约束（默认 3，最大 9），超出时分批调度。

### 常见陷阱
- Windows 上不要用 `python3` — 用 `python`
- Windows 终端可能乱码中文输出 — 在脚本中添加 `sys.stdout.reconfigure(encoding='utf-8')`
- JSONL 时间戳是毫秒 — 除以 1000 转换为 datetime
- 发送者名称可能随时间变化（groupCard 更新）— 使用 UIN 作为稳定标识
- Bot 账号产生大量消息 — 考虑从"活跃用户"排行中排除或单独分析
