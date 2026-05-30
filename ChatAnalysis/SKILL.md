---
name: chat-analysis
description: >
  This skill should be used when the user asks to "analyze chat records",
  "analyze chat logs", "analyze conversation history", "做聊天记录分析",
  "分析聊天记录", "聊天数据分析", "对话记录分析", or mentions parsing
  messaging app exports (QQ, WeChat, Telegram, Discord, WhatsApp, etc.).
  Provides a complete pipeline from raw chat logs to structured analysis
  reports and rendered output (HTML/Markdown/PDF).
compatibility: Recommends Python 3.8+ (scripts generated at runtime)
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, Agent]
argument-hint: <chat-file-or-dir> [--mode group|private] [--focus-users uin1,uin2] [--output <dir>] [--output-style html|md|all]
metadata:
  author: chat-analysis-skill
  category: data-analysis
  tags: [chat, nlp, analysis, report, html, group-chat, private-chat, jsonl]
---

# Chat Record Analysis Skill

A complete pipeline for deep analysis of chat/messaging records. Supports both **private chat** (1v1) and **group chat** (multi-user) scenarios. Parses structured chat logs, extracts multi-dimensional statistics, generates topic reports, personality profiles, and renders everything to styled output.

## Scope

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

## Supported Scenarios

| Scenario | Description | Key Differences |
|----------|-------------|-----------------|
| **Private Chat** | 1-on-1 conversation | Relationship-focused, 2 sender profiles, reply chain analysis |
| **Group Chat** | Multi-user group | Social dynamics, N sender profiles, activity rankings, bot interaction, forward culture |

## Analysis Philosophy

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
- **用户全责** — 输出仅做敏感内容警告（红边框提醒），不兜底、不隐藏、不模糊内容，保持原文完整性

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

## Sensitive Content Handling

本节集中定义敏感内容的分类、默认处置流程和可选模块。各 Phase 中涉及敏感内容的具体操作均引用本节。

### Definition

| 类型 | 是否敏感 | 说明 |
|------|----------|------|
| **严重违反法律法规** | 是 | 媒体文件涉及未成年人、暴力犯罪、毒品、恐怖主义等 |
| **漏点色情** | 是 | 媒体文件暴露关键部位的色情内容 |
| 价值观不同 | 否 | 政治观点、宗教信仰、生活方式差异 |
| 软色情 | 否 | 媒体文件擦边、暗示性内容，但未暴露关键部位 |
| 无露出色情 | 否 | 色情内容但未暴露关键部位 |
| 争议性言论 | 否 | 不当言论、冒犯性表达，按一般敏感内容处理（红边框警告） |

**敏感内容主要针对媒体文件。** 文字内容一般不需要做敏感处理（争议性言论仅加红边框警告，不隐藏不脱敏）。敏感判断的核心对象是图片、音频、视频等媒体文件。

**媒体文件的敏感性判断：** 媒体文件是否属于敏感内容，不能仅看文件本身，必须结合其在聊天记录中的上下文判断。同一张图片在不同语境下敏感性可能不同。判断时应参考：文件名、前后消息内容、发送者意图、讨论话题。

**用户可重新定义敏感内容范围**（仅在当前任务中生效，不影响 SKILL 本体）。

### Default Behavior (no user prompt needed)

发现严重违法/色情内容时，自动执行以下流程：

1. **`html/.completed-hidden/`** — 完整包含所有内容，但敏感部分用 `<details>` 隐藏：
   ```html
   <details>
     <summary>⚠️ 敏感内容（严重违法/色情）— 点击展开</summary>
     <div class="danger-block">
       <!-- 敏感内容原文 -->
     </div>
   </details>
   ```
2. **`html/.sensitive/`** — 仅放置敏感标记（来源、位置、类型），不存储完整内容
3. **`html/.pure/`** — 通过集合运算生成：从 `.completed-hidden/` 移除整个 `<details>` 敏感块
   - `.completed-hidden = .pure ∪ 隐藏的敏感块`，`.pure = .completed-hidden - 敏感块`
4. **任务结束后报告** — 向用户说明发现了哪些敏感内容、Agent 做了什么处理、如何查看各版本

### Optional Module: Sensitive Content Showcase (default off, never guided)

仅当用户**主动明确要求**时才开启：

1. 填充 `html/.sensitive/` — 从仅标记升级为完整内容摘录
2. 生成 `html/.completed-parts/` — 完整内容打碎为分块（纯净分块 + 敏感分块）
3. 用户可通过分块自由组合：纯净块 → 纯净版，全部块 → 完整版

**不主动引导用户开启此模块。** 仅在用户自行提出时响应。

### General Sensitive Content (non-severe)

一般敏感内容（价值观不同、软色情、无露出色情、争议性言论等）— 保留在正文中，添加视觉警告：

1. **DO NOT** modify, redact, or censor the original chat text
2. **DO** add a visual warning indicator (red border, warning icon)
3. **DO** paraphrase analysis rather than verbatim quoting sensitive portions
4. **DO** focus on communication pattern analysis rather than content endorsement
5. **DO** frame analysis academically ("从沟通行为分析角度...")

---

## Workflow Overview

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
└─► Phase 3: Output Rendering → .output/html/
      ├─ .completed-hidden/ — 完整版（敏感内容<details>隐藏，默认）
      ├─ .pure/            — 纯净版（移除敏感块，默认）
      ├─ .sensitive/       — 敏感标记（默认）/ 大赏内容（可选）
      └─ .completed-parts/ — 分块输出（可选，用户开启时）
```

---

## Phase 0: Input Detection & Configuration

初始化工作目录，检测数据格式，收集用户配置。完成后进入 Phase 1。

### 0.0 Directory Initialization

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
| `.output/` 不存在 | 创建 `.output/` 及子目录 |

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
> 本 SKILL 支持多种输出玩法，如需了解可输入"查看玩法"获取预设玩法列表及说明。
>
> ⚠️ 风险提示：分析输出基于聊天记录文本数据，AI 生成内容不代表用户真实主观感受。
> 输出中可能包含敏感或争议性内容，所有内容仅做警告提醒，不兜底、不隐藏、不模糊。
> 使用本 SKILL 生成的全部内容，由用户自行承担责任。

### 0.1 Format Detection

Two primary formats are supported:

#### Text Format (single file)
Plain text chat logs with date headers and sender prefixes:
```
## 2026-01-01 09:30
用户A：早上好
用户B：早
用户B：[图片]
```

#### JSONL Format (directory of batch files)
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

### 0.2 Chat Mode Detection

| Indicator | Group Chat | Private Chat |
|-----------|-----------|--------------|
| Sender count | >2 unique senders | Exactly 2 senders |
| System messages | Join/leave notifications | Rare |
| @mentions | Common | Rare |
| Forward messages | Common | Rare |
| Bot interactions | Common (bot accounts) | Rare |

**Auto-detection:** Count unique senders in first 500 messages. If >2, classify as group chat.

**Manual override:** User can specify `--mode group` or `--mode private`.

### 0.3 User Configuration

Ask the user (or detect from data):

1. **Chat platform** — QQ, WeChat, Telegram, Discord, WhatsApp, or auto-detect
2. **Chat mode** — group chat or private chat
3. **Focus users** — UINs or names of users to analyze in depth (typically the user's own accounts)
4. **Output directory** — where to write results
5. **Output style** — `html` (default), `md`, or `all`
6. **完整聊天记录长页（可选）** — 是否生成完整聊天记录的单页 HTML（默认不生成）：
   - 敏感内容用 `<details>` 隐藏（与 `.completed-hidden/` 同逻辑）
   - 媒体文件用 `<a>` 链接插入（不内嵌），避免页面过大
   - 消息按时间合并显示（5 分钟分组，见 3.14 节）
   - 含页脚链接（如用户允许）

   **放置位置（必须询问用户）：**
   > 完整聊天记录长页要放在哪里？
   >
   > 1. **放入 html/ 输出目录**（推荐） — 输出到 `html/.whole-page/chat.html`，其他 HTML 页面可通过相对路径 `.whole-page/chat.html` 或 `../.whole-page/chat.html` 链接跳转
   > 2. **单独目录** — 输出到 `.output/.whole-page/chat.html`，与各输出形式平级。由于迁移后相对路径不确定，**强烈不建议在此模式下添加跨目录链接**

   选择选项 1 时，追加询问：是否在首页 `index.html` 添加跳转链接（默认不添加）。
   选择选项 2 时，不询问链接，仅生成独立页面。
7. **页脚技术支持链接（可选）** — 如果输出模式为 `html`，在开始输出前询问用户：
   > 本 SKILL 制作不易，是否允许制作方在你的输出页面底部加入 GitHub 技术支持链接？
   >
   > 链接内容：页脚显示「SKILL技术支持·ChatAnalysis」，指向本项目 GitHub 仓库。
   > 选择「允许」将在所有生成的 HTML 页面底部添加此链接；选择「不允许」则不添加。

   用户允许时，HTML 渲染阶段为所有页面添加页脚链接：`<footer><a href="https://github.com/JularDepick/ChatAnalysis.SKILL">SKILL技术支持·ChatAnalysis</a></footer>`。
   用户拒绝或输出模式为 `md` 时跳过此步骤。

8. **叙述人称** — 输出报告的叙事视角：
   - **第三人称（默认）** — 人称无关叙事，不出现"我"，使用用户名或"该用户"等中性称呼
   - **第一人称** — 用户指定自己在聊天中的身份（如"我"对应哪个昵称/UIN），输出中以此人称叙事

   默认采用第三人称，不涉及"我"的指定。仅当用户主动选择第一人称时，才在输出中明确"我"的指向。
9. **敏感内容大赏（可选）** — 用户主动要求时才开启（见 [Sensitive Content Handling](#sensitive-content-handling)）

For group chats, additionally ask:
10. **TOP N ranking** — how many top users to profile individually (default: 10, max: 20)
11. **Standalone pages** — whether to generate brilliant/legendary quote pages
12. **XXX历险记** — 是否生成"群聊历险日记"模块（默认不输出）：
    - 以不同聊天用户的第一人称视角，撰写群聊历险日记
    - 输出到 `SpeakAs/` 目录，每个用户一个文件
    - 需严格注明 AI 生成内容不代表具体用户的主观感受
    - 用户选择开启后，需指定撰写哪些用户（默认 TOP N）

### 0.4 Media Resource Detection

扫描原始数据目录，检查是否存在媒体文件（图片、视频、文件等）。常见位置：`resources/images/`、`resources/files/`、`data/`。

如果发现媒体文件，**必须询问用户**选择整合策略：

> 在原始数据中发现了 X 张图片、Y 个文件（共 Z MB）。
> 是否要将这些媒体文件整合到 HTML 输出中？
>
> 选项：
> 1. **全部使用** — 复制到输出目录，在报告中引用
> 2. **选择性使用** — 仅引用精彩发言/神人发言中涉及的媒体
> 3. **不使用** — 仅输出文本分析，不引用媒体

**含文本图片识别建议：** 如果发现可能含文本的图片（聊天截图、长文图片、文字表情包），主动提醒用户使用 OCR 工具提取文本后补充。不阻塞任务，建议先完成现有分析。

**媒体引用方式：** 报告中引用原始聊天记录时，**必须使用 HTML 标签**（`<img>`、`<audio>`、`<video>`）展示媒体内容，不要依赖纯文本占位符如 `[图片]`。所有媒体引用**必须使用相对路径**（如 `resources/images/xxx.jpg`），不使用绝对路径或 `file://` 协议。每个标签必须固定一个维度（width 或 height）到固定 px 值，不超过屏幕较短一边的 50%（推荐 360px）。详见 `references/script_writing_guide.md` 3.7 节。

**善用原始媒体：** 分析报告（人格画像、精彩发言、话题专题等）中引用聊天原文时，如果原始消息包含图片/音频/视频，应主动将对应的媒体文件嵌入报告中作为证据和展示素材，而非只引用文字。媒体是聊天的重要组成部分，图文并茂的报告远比纯文本质感更高。

**选择性复制：** 只有实际被报告引用的媒体文件才复制到输出 `resources/` 目录。不要全量复制原始数据中的媒体文件，仅挑选用到的。

**大文件过滤：** 默认过滤超过 10MB 的文件。用户选择「全部使用」时，先报告大文件数量，询问是否调整阈值（5MB/2MB）。允许用户选择压缩媒体质量。

**输出体积控制：** 媒体整合后报告预估总大小。如需控制体积，按优先级舍弃：topic/report 装饰图 → analyse 图 → 保留 personality/brilliant 核心证据图 → 按大小排除。仅移除 HTML `<img>` 引用，不删除 `resources/` 文件。

---

## Phase 1: Parse & Statistics

解析原始数据，提取多维统计，输出 `chat_stats.json`。完成后进行数据验证和输出确认。

### 1.1 Script Generation

**This SKILL does not ship pre-built scripts.** Instead, scripts are written at runtime in the user's working directory based on the actual data format and requirements.

Read `references/script_writing_guide.md` for the complete script-writing methodology, including:
- Parser script patterns (text format and JSONL format)
- Statistics computation patterns
- HTML rendering patterns
- Platform-specific adaptations
- Error handling conventions

**Script generation workflow:**
1. Read 20-50 lines of the actual data to understand its exact format
2. Write the parser script tailored to that format
3. Write the script to `<workdir>/.output/scripts/`
4. Run and verify the script produces correct output
5. Iterate if parsing errors are detected

**Key principle:** Every chat export has subtle format differences. The script must be adapted to the actual data, not the other way around.

### 1.2 Text Format Parser

For single-file text chat logs, write a parser script following the patterns in `references/script_writing_guide.md`. The parser must:

1. Auto-detect date header patterns from the first 100 lines
2. Auto-detect sender patterns
3. Classify message types (text, image, card, system, reply)
4. Extract timestamps
5. Compute all statistics (see Section 1.3)

### 1.3 JSONL Format Parser

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

### 1.4 Statistics Output

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

### 1.5 Activity Cliff Detection (Group Chat)

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

### 1.6 Data Sanity Verification

After parsing, verify:
- [ ] Total message count matches expectations
- [ ] Date range is correct
- [ ] Sender names are properly distinguished
- [ ] No parsing errors (zero-message days that shouldn't be zero)
- [ ] Focus users are present in the data
- [ ] Message type distribution is reasonable

### 1.7 Pre-Output Confirmation

Phase 1 完成后、Phase 2 开始前，Agent 必须向用户报告数据扫描结果并确认输出架构：

**报告内容：**
- 数据概况（消息数、日期范围、发言者数）
- 是否发现敏感内容（如有，说明类型、数量、大致位置）
- 是否发现媒体文件

**询问用户选择输出架构：**

> 数据扫描完成，准备生成报告。请选择输出架构：
>
> 选项：
> 1. **完整-hidden（默认）** — `.completed-hidden/`（完整版，敏感内容用<details>隐藏）+ `.pure/`（纯净版）
> 2. **仅纯净版** — 只生成 `.pure/`，不含敏感内容
> 3. **仅完整版** — 只生成 `.completed-hidden/`，敏感内容可手动展开
> 4. **含敏感内容大赏** — 完整-hidden + `.sensitive/`（填充完整内容）+ `.completed-parts/`（分块）

**默认采用选项 1（完整-hidden）。** 如果数据中无敏感内容，直接告知用户无需选择，按默认输出生成。

---

## Phase 2: Deep Analysis Reports

并行生成多维度分析报告：统计分析、综合报告、话题专题、人格画像，以及可选的精彩发言和群聊历险日记。

### 2.1 Output Structure

标准工作目录结构：

```
workdir/                          — 工作目录
├── row/                          — 用户提供的原始数据（聊天文件/JSONL 目录）
└── .output/                      — 输出目录（隐藏目录）
    ├── chat_stats.json           — Phase 1 统计数据
    ├── scripts/                  — 运行时生成的辅助脚本
    ├── README.ai.md              — Agent 复用指南
    ├── md/                       — 清洗后的 Markdown
    │   ├── *.md                  — 默认扁平放置（除非用户要求分组）
    │   ├── analyse/              — 用户要求分组时：5 篇统计分析
    │   ├── report/               — 用户要求分组时：4 篇综合报告
    │   ├── topic/                — 用户要求分组时：N 篇话题专题
    │   ├── personality/          — 用户要求分组时：每用户画像
    │   ├── brilliant.md          — 精彩发言展示页
    │   ├── legendary.md          — 神人发言展示页
    │   └── SpeakAs/              — 群聊历险日记（可选）
    │       ├── disclaimer.md     — AI 生成声明
    │       └── <user>.md         — 每用户历险日记
    └── html/                     — HTML 输出
        ├── index.html            — 首页（排行榜）
        ├── assets/               — CSS/JS（可分离）
        ├── resources/            — 媒体文件
        ├── analyse/              — 统计分析 HTML
        ├── report/               — 综合报告 HTML
        ├── topic/                — 话题专题 HTML
        ├── personality/          — 人格画像 HTML
        ├── SpeakAs/              — 历险日记 HTML
        ├── brilliant.html        — 精彩发言
        ├── legendary.html        — 神人发言
        ├── .completed-hidden/     — 完整版（敏感内容用<details>隐藏，默认生成）
        ├── .pure/                — 纯净版（移除敏感块，默认生成）
        ├── .sensitive/           — 敏感标记（默认）/ 敏感内容大赏（用户开启时填充）
        ├── .completed-parts/     — 分块输出（可选，敏感内容大赏开启时创建）
        └── .whole-page/          — 完整聊天记录长页（可选，放入 html/ 时在此）
            └── chat.html         — 全部消息的单页 HTML，敏感内容<details>隐藏
    └── .whole-page/              — 完整聊天记录长页（可选，单独目录时在此）
        └── chat.html
```

**关键约定：**
- `row/` 存放原始数据，不做任何修改
- `.output/` 为隐藏目录，所有产出均在此下
- `md/` 默认扁平放置所有 md 文件，除非用户在 Phase 0 要求按分类分子目录
- `.completed-hidden/`、`.sensitive/`、`.pure/`、`.completed-parts/`、`.whole-page/` 均在 `html/` 下，仅涉及 HTML 输出的敏感内容处置
- `chat_stats.json` 在 `.output/` 根目录，供所有报告引用

### 2.2 Agent Dispatch Strategy

Use parallel background agents for report generation. Each agent is responsible for a specific output directory.

#### Private Chat (2 senders)

Launch 3-4 agents:

| Agent | Files | Description |
|-------|-------|-------------|
| analyse-agent | 5 files in analyse/ | Statistical deep-dive |
| report-agent | 4 files in report/ | Comprehensive reports |
| topic-agent | N files in topic/ | Topic-specific reports |
| personality-agent | 2 files in personality/ | Both sender profiles |

#### Group Chat (N senders)

Launch 5-9 agents depending on scale:

| Agent | Files | Description |
|-------|-------|-------------|
| analyse-agent | 5 files in analyse/ | Statistical deep-dive with cross-analysis |
| report-agent | 4 files in report/ | Multi-dimensional comprehensive reports |
| topic-agent | N files in topic/ | Topic-specific reports (9-12 topics typical) |
| personality-batch-A | TOP 1-5 users | Personality profiles for top 5 |
| personality-batch-B | TOP 6-10 + focus users | Personality profiles for rank 6-10 plus focus users |
| personality-batch-C | TOP 11-N (if requested) | Additional personality profiles |
| brilliant-agent | brilliant.md | 精彩发言展示页 with deep meme analysis |
| legendary-agent | legendary.md | 神人发言展示页 with extraordinary quotes |

**Important dispatch rules:**
- Use `run_in_background: true` for all agents
- Launch all agents in a single message with multiple tool calls for parallelism
- Each agent must be self-contained with its own context (chat file path, stats JSON path, output paths)
- Do NOT launch duplicate agents for the same task
- Monitor agent completion via task notifications
- If an agent is rejected as "high risk", relaunch with adjusted prompt (see Section 2.9)

### 2.3 Writing Standards (enforce on all agents)

Every report MUST:
1. Use tables for structured data
2. Include original chat quotes in code blocks as evidence
3. When quoting messages that contain media (images, audio, video), embed the actual media files using HTML tags (`<img>`, `<audio>`, `<video>`) with fixed dimensions (max 360px), not just text placeholders
4. Cite specific statistics (counts, percentages, dates)
5. Acknowledge limitations (single-perspective, text-only, time-bounded)
6. Be written in the same language as the chat content
7. Each file ≥ 300 lines; aim for depth over breadth
8. Read 6-10 different segments of the chat file (not just the beginning)
9. Spot-check data against chat_stats.json
10. Follow the narrative perspective configured in Phase 0:
   - **第三人称（默认）**：使用"该用户"、用户名、"发言者"等中性称呼，全文不出现"我"
   - **第一人称**：用户指定的焦点用户以"我"叙事，其他用户使用用户名

Every report MUST NOT:
1. Make claims without evidence from the chat
2. Speculate about private life beyond what's observable
3. Include actual phone numbers, passwords, or sensitive PII in examples
4. Write vague summaries — every paragraph needs data or quotes

### 2.4 Topic Detection Approach

To identify topics, use a two-phase approach:

**Phase A — Discovery:** Read 8-10 segments evenly distributed across the date range. Note topics, keywords, who initiates each topic.

**Phase B — Classification:** Build keyword lists from discovered topics, then full-scan:

| Topic Category | Example Keywords |
|---------------|-----------------|
| Technology/AI | AI, ChatGPT, code, program, VPN, server, GitHub, python, api |
| Entertainment | game, anime, video, movie, music, B站, 抖音 |
| School/Career | exam, homework, GPA, professor, class, 考试, 作业, 绩点 |
| Social/Relationships | friend, classmate, dating, family, 同学, 朋友, 室友 |
| Daily Life | food, weather, sleep, commute, shopping, 吃, 喝, 食堂 |
| Emotions | happy, sad, stressed, lonely, emo, 开心, 难过, 焦虑 |
| Anime/Gaming | 二次元, 番剧, 原神, 崩坏, 王者, steam |
| Bot Interaction | 老婆, 抽取, 市场, 银币, 表情合成 |

Adapt keyword lists to the actual chat content. For group chats, bot interaction patterns are a significant topic category.

### 2.5 Personality Profiling Rules

When writing personality reports:

**For private chat (2 senders):**
- Focus on relationship dynamics, communication patterns, emotional exchange
- Compare both persons side-by-side
- Include relationship evolution over time

**For group chat (N senders):**
- Focus on individual communication style, social role, influence patterns
- Analyze bot interaction patterns (wife-collecting games, gacha systems)
- Track reply targets to map social connections
- Identify role archetypes: initiator, responder, lurker, bot user, meme spreader, topic driver
- For each user, compute: message count, text/image ratio, avg message length, active days, peak hours, top reply targets, keyword profile

**Common rules for both:**
- Base ALL observations on verifiable chat behavior
- Include the exact chat text as evidence for every claim
- Use qualifying language: "从聊天中可观察到", "在这段关系中表现出"
- End with a "局限性声明" (Limitations) section

### 2.6 Brilliant & Legendary Quotes Pages (Group Chat)

For group chats, generate two standalone showcase pages:

#### 精彩发言展示页 (brilliant.md)
- 20-30 notable quotes/conversations
- Categories: 神回复, 经典对话, 接龙/刷屏, 梗传播, 犀利吐槽, 荒诞对话
- Each entry includes: original chat text, context explanation, quoteblock analysis of humor/meme
- Preserve full context (if too long, summarize cause-effect then show core)
- Include a "群内高频梗速查表" appendix

**推荐功能扩展（可选，向用户提议）：**
- 群内高频梗速查表 — 梗的起源、演变、使用场景
- 精彩内容集锦 — 按主题/时间/用户分类的精华摘录
- 深度点评/锐评 — 对群聊文化、社交现象的批判性分析
- 时间线大事记 — 群聊重要事件的 chronological 记录
- 社交网络图 — 基于回复关系的用户互动可视化

#### 神人发言展示页 (legendary.md)
- 10-20 extraordinary/legendary quotes
- Each entry includes: original chat text, 5-star rating, detailed "神人解析"
- Focus on: abstract literature, copypasta, dramatic monologues, philosophical rants, viral moments

**Both pages:**
- High-risk content (inappropriate价值观, 违背道德) gets a red-border warning div
- Warning only — NO censorship, NO content modification
- Preserve original text integrity
- Utilize media resources (images, forwards) effectively

### 2.7 XXX历险记 — 群聊历险日记（可选，默认不生成）

以不同聊天用户的第一人称视角，撰写群聊历险日记。这是一个创意性附加玩法，**默认关闭**，需用户在 Phase 0 主动开启。

#### 输出结构

```
md/SpeakAs/
├── disclaimer.md        — AI 生成声明（必须包含）
├── <用户A>.md           — 用户A视角的历险日记
├── <用户B>.md           — 用户B视角的历险日记
└── ...
```

#### 内容规范

**diclaimer.md（AI 生成声明，必须放在 SpeakAs/ 目录根目录）：**

> ⚠️ AI 生成声明
>
> 本目录下的所有"历险日记"均由 AI 基于群聊记录生成，采用对应用户的第一人称视角撰写。
> **这些内容不代表该用户的真实主观感受、想法或立场。**
> 文中所有"我"均为 AI 模拟的叙事视角，不代表该用户本人的观点。
> 本内容仅供娱乐和创意阅读，请勿作为判断用户真实性格或态度的依据。

**每篇历险日记的写作要求：**

1. **第一人称叙事** — 以该用户的视角写"我"，但必须在文首重复声明
2. **基于真实聊天事件** — 日记中提到的群聊事件必须有聊天记录依据
3. **时间线忠实** — 按照聊天记录中的时间顺序组织叙事
4. **引用原文** — 关键事件引用原始聊天记录（代码块格式）
5. **创意但不虚构** — 可以用文学化语言描述场景和心理活动，但不能捏造未发生的事件
6. **文首声明** — 每篇日记开头必须包含简短声明：

> ⚠️ 本文为 AI 生成的创意内容，以 [用户名] 的第一人称视角撰写。
> 文中"我"不代表该用户的真实感受，仅供参考娱乐。

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

### 2.8 High-Risk Content Handling

**一般敏感内容** — 保留在正文中，添加视觉警告（详见 [Sensitive Content Handling — General Sensitive Content](#general-sensitive-content-non-severe)）。

**严重违法/色情内容** — 按 [Sensitive Content Handling — Default Behavior](#default-behavior-no-user-prompt-needed) 执行：
- 默认：`.completed-hidden/` 保留完整但用 `<details>` 隐藏，`.sensitive/` 放置标记，`.pure/` 移除敏感块
- 用户开启"敏感内容大赏"时：`.sensitive/` 填充完整内容，额外生成 `.completed-parts/` 分块

**任务结束后必须向用户报告：** 发现了哪些敏感内容、Agent 做了什么处理、如何查看各版本。

### 2.9 Agent Rejection Recovery

If an agent is rejected by the safety filter:

1. Adjust the prompt to emphasize:
   - Academic/analytical purpose
   - Communication behavior analysis focus
   - Paraphrasing sensitive content rather than verbatim quoting
   - Framing as observational study
2. Reduce the scope (fewer users per batch)
3. Relaunch with the adjusted prompt
4. If still rejected, write the files manually in the main conversation

---

## Phase 3: Output Rendering & Cleanup

将 Markdown 报告渲染为 HTML，执行文件清理，生成元数据，交付输出。

### 3.1 Output Modes

| Mode | Description | Command |
|------|-------------|---------|
| `html` (default) | Styled HTML with navigation | `python md2html.py <md_dir> <html_dir> --stats <stats_json>` |
| `md` | Raw Markdown files only | No rendering needed |
| `all` | Both HTML and Markdown | Run HTML converter, keep md files |

### 3.2 HTML Rendering

Write an HTML converter script in the user's working directory following the patterns in `references/script_writing_guide.md`. The converter reads all `.md` files from the output directory structure and generates styled HTML pages.

```bash
python <workdir>/.output/scripts/md2html.py <workdir>/.output/md <workdir>/.output/html --stats <workdir>/.output/chat_stats.json
```

**页脚链接：** 如果用户在 Phase 0.3 中允许了技术支持链接，渲染时添加 `--footer-link` 参数：
```bash
python <workdir>/.output/scripts/md2html.py <workdir>/.output/md <workdir>/.output/html --stats <workdir>/.output/chat_stats.json --footer-link
```

#### Design System

The HTML output uses an Apple-style design system matching modern chat viewer aesthetics:

**CSS Variables (Light Theme):**
```css
--bg-primary: #ffffff;
--bg-secondary: #f5f5f7;
--text-primary: #1d1d1f;
--text-secondary: #86868b;
--accent: #007aff;
--border-color: rgba(0,0,0,0.08);
```

**CSS Variables (Dark Theme):**
```css
--bg-primary: #000000;
--bg-secondary: #1c1c1e;
--text-primary: #f5f5f7;
--text-secondary: #98989f;
--accent: #0a84ff;
--border-color: rgba(255,255,255,0.12);
```

**Font Stack:**
```css
font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display",
             "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
```

#### Theme Toggle

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

#### Homepage Leaderboard

For group chats, the main index page features a TOP N leaderboard:
- Bar chart visualization (CSS-based, no JS libraries)
- Click on any user → navigates to their personality profile page
- Shows message count and relative activity percentage
- Top 3 users highlighted with accent color
- Below leaderboard: category sections linking to analyse/, report/, topic/
- Standalone page links (brilliant, legendary) as styled buttons

#### Navigation

Every page has breadcrumb navigation:
```
首页 › 分类名 › 文章标题
```

#### Responsive Design

- Max width: 880px centered
- Mobile breakpoint: 768px
- Tables scroll horizontally on mobile
- Leaderboard bars hidden on mobile

#### Time Merging

前端展示聊天记录时，合并连续消息的时间显示：5 分钟内无中断的消息归入同一个起始时间戳下。原始数据保留每条消息的完整时间，仅在渲染层合并。详见 `references/script_writing_guide.md` 3.14 节。

#### Architecture

**纯前端项目，不依赖后端。** 所有功能通过 HTML/CSS/JS 实现，允许使用浏览器存储（localStorage）。

CSS/JS 可以分离为独立文件以优化加载，但必须使用相对路径引用：
```
html/
├── index.html
├── assets/
│   ├── style.css          ← 分离的样式
│   └── theme.js           ← 分离的脚本
├── resources/             ← 媒体文件
└── ...
```

引用方式：`<link rel="stylesheet" href="assets/style.css">`，确保迁移部署后正常工作。

### 3.3 Portability

All output HTML must use **relative paths** for links, references, and resources. This ensures the output directory can be moved, deployed, or served from any location without breaking.

- Navigation links: `../index.html`, `../analyse/index.html`
- User profile links: `personality/用户名.html`
- Resource references: relative to the HTML file's location
- No absolute paths, no `file://` protocol, no hardcoded domains

### 3.4 Verification

After HTML rendering, verify:
- [ ] All files are linked from index pages
- [ ] Navigation breadcrumbs work correctly
- [ ] Tables render properly
- [ ] Code blocks display with correct syntax highlighting
- [ ] Chinese characters render without garbling
- [ ] Theme toggle works and persists across pages
- [ ] Leaderboard links point to correct personality pages
- [ ] Standalone pages (brilliant, legendary) are accessible
- [ ] Responsive layout works on mobile viewport

### 3.5 File Cleanup

任务完成后，清理工作产生的临时文件和中间文件，但保留三类文件：

| 保留 | 清理 |
|------|------|
| `row/` 原始数据（不做任何修改） | 子代理提取的中间文本片段 |
| `.output/md/` 清洗后的 md 文档 | 调试用的临时脚本、日志 |
| `.output/html/` 最终 HTML 输出 | 重复运行产生的冗余文件 |
| `.output/chat_stats.json` 统计数据 | — |

**操作时机：** 在 Phase 3 输出渲染完成后、向用户报告完成之前执行清理。

### 3.6 README.ai.md Generation

任务完成后，在 `.output/` 根目录生成 `README.ai.md`，内容包括：
- 分析概况（消息数、日期范围、发言者数）
- 目录结构说明
- 数据来源和格式
- chat_stats.json 关键字段说明
- 焦点用户列表
- 复用指南（如何浏览、引用、二次分析）

此文件方便后继 Agent 快速了解输出结构，实现高效复用。

---

## Quality Checklist

Before declaring analysis complete:

### Data Integrity
- [ ] Statistics match raw data (spot-check 3-5 data points)
- [ ] Message counts are consistent across stats and reports
- [ ] Date ranges are accurate
- [ ] Sender names are correctly resolved

### Report Quality
- [ ] Every major claim has chat quote evidence
- [ ] Topic reports cover all significant conversation themes (>5% of messages)
- [ ] Personality reports include limitation disclaimers
- [ ] All files meet minimum line count requirements (≥300 lines)
- [ ] No sensitive personal data leaked in examples

### Output Quality
- [ ] HTML renders correctly in browser
- [ ] All index pages link to sub-pages
- [ ] Theme toggle works and persists
- [ ] Leaderboard links are correct
- [ ] Standalone pages are accessible
- [ ] Responsive design works

### File Organization
- [ ] All files in correct output directory structure
- [ ] chat_stats.json is present and valid
- [ ] md/ and html/ directories are properly structured
- [ ] No orphaned or missing files

---

## Self-Iteration Rules

When maintaining this skill based on actual task experience:

1. **NEVER write task data into the skill.** Do not include actual chat content, user names, UINs, or any real data from analysis tasks.
2. **NEVER leak sensitive content.** Do not include example quotes that could identify real users.
3. **NEVER include iteration logs.** The SKILL body must not contain changelogs, version histories, update records, or any trace of iterative modification. The SKILL should read as if it was written once, correctly.
4. **SKILL purity is t0.** The SKILL must always be pure, independent, self-contained, reflecting only the latest state. No references to old versions, no "formerly known as", no "previously we did X". The SKILL is the SKILL — not a diff of itself.
5. **DO encode patterns, not instances.** Write "group chats with >50 senders benefit from batch personality profiling" instead of "the 往日种种 group had 196 senders".
6. **DO update methodologies.** If a new analysis technique proves effective, add it to the reference docs.
7. **DO keep references generic.** All examples should use synthetic data (用户A, 用户B).
8. **DO provide preset options.** When the SKILL says "如果用户选择/要求", the Agent must present concrete preset options (2-4 choices with clear descriptions) using the question tool, not ask open-ended questions. Users should click, not type.

### Post-Task Reminder

完成一次任务输出后，Agent 应主动提醒用户：

> 本次分析已完成。是否要从这次任务中提取经验来迭代优化 chat-analysis SKILL？
>
> 如果您同意，迭代过程将遵守以下规范：
> - 不将任务数据（聊天内容、用户名、UIN 等）写入 SKILL
> - 不泄露敏感内容
> - 不在 SKILL 中留下迭代日志/记录
> - 仅编码通用模式和方法论
> - 保持 SKILL 的纯净性和独立性
>
> 如不需要，可直接忽略此提示。

**只有在用户明确同意后，才能进行 SKILL 迭代。**

### Task-Time SKILL Iteration

当用户在任务进行过程中提出 SKILL 迭代/维护要求时：
- **用子代理维护 SKILL** — 将 SKILL 迭代工作交给后台子代理，主对话继续推进分析任务
- **或用子代理做任务** — 如果 SKILL 迭代更紧急，将分析任务交给后台子代理，主对话处理 SKILL

原则：不要让 SKILL 迭代阻塞任务进度，也不要让任务阻塞 SKILL 迭代。并行处理。

---

## Notes

### Performance Guidelines
- Use `Agent` tool with `run_in_background: true` for parallel report generation
- Each agent should read 6-10 different segments of the chat file (not just the beginning)
- For large files (>10K lines), use `Read` with `offset` and `limit` parameters
- For JSONL batch directories, load all messages into memory for cross-batch analysis
- Spot-check agent output by reading the actual files — don't trust agent summaries

### Scale Guidelines
| Chat Size | Recommended Agents | Expected Time |
|-----------|-------------------|---------------|
| <1K messages | 2-3 agents | 2-5 min |
| 1K-5K messages | 3-4 agents | 5-10 min |
| 5K-25K messages | 5-9 agents | 10-20 min |
| >25K messages | 9+ agents | 20-30 min |

### Common Pitfalls
- Don't use `python3` on Windows — use `python` instead
- Windows terminal may garble Chinese output — add `sys.stdout.reconfigure(encoding='utf-8')` to scripts
- JSONL timestamps are in milliseconds — divide by 1000 for datetime conversion
- Sender names may change over time (groupCard updates) — use UIN as stable key
- Bot accounts produce high message volumes — consider excluding from "active user" rankings or analyzing separately
