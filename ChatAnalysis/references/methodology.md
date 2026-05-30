# Chat Analysis Methodology

## 1. Data Parsing Strategy

### 1.1 Chat Log Structure Detection

Most chat exports follow one of two patterns:

**Text Format:**
```
[Date/Time Header]
[Sender]: [Content]
[Sender]: [Content]
...
[Next Date/Time Header]
```

**JSONL Format:**
```
{"id":"...","timestamp":1234567890000,"sender":{"uin":"123","name":"张三"},"type":"text","content":{"text":"内容"}}
{"id":"...","timestamp":1234567891000,"sender":{"uin":"456","name":"李四"},"type":"text","content":{"text":"回复"}}
```

Detection approach:
1. Check if input is a directory (JSONL) or file (text)
2. For text: Read first 200 lines, identify date/sender patterns
3. For JSONL: Read first 5-10 lines, identify field structure

### 1.2 Message Classification

Each parsed message should be classified:

| Type | Detection | Handling |
|------|-----------|----------|
| Text | Normal text content | Count words, extract keywords |
| Image | `[图片]`, `[Image]`, image URL, `type: "image"` | Count, categorize by context |
| Card/Link | `[卡片消息:...]`, URL preview | Extract title, categorize |
| Forward | `type: "forward"`, forwarded content | Count, analyze source patterns |
| Reply | `> quoted text`, `type: "reply"` | Parse reply target, count reply chains |
| System | 撤回, 红包, 加入群聊, `system: true` | Filter out or categorize separately |
| Recalled | `recalled: true` | Note but exclude from content analysis |

### 1.3 Sender Identification

**Private chat (2 senders):**
- Identify "me" (the user) vs "other"
- Normalize "me" label for consistent analysis

**Group chat (N senders):**
- Use UIN (QQ number) or equivalent as stable key
- Resolve display name: groupCard > name > nickname
- Track name changes over time (groupCard may change)
- Identify bot accounts (high message volume, automated patterns)

## 2. Statistical Analysis Dimensions

### 2.1 Basic Statistics

| Metric | How to Compute |
|--------|---------------|
| Total messages | Count of all parsed messages |
| Valid messages | Exclude system and recalled messages |
| Active days | Count of unique dates with ≥1 message |
| Date range | min(date) to max(date) |
| Messages per sender | Group by sender, count |
| Message types | Group by message_type, count |
| Average message length | mean(char_count) per text message |

### 2.2 Time Distribution

| Dimension | Grouping | Insight |
|-----------|----------|---------|
| Hourly | Group by hour (0-23) | Peak activity hours |
| Daily | Group by date | Activity trends, streaks |
| Weekday | Group by day_of_week | Work vs weekend patterns |
| Monthly | Group by month | Long-term trends |

Cross-analysis:
- Hour x Weekday matrix → find hottest/coldest time slots
- Daily first/last message → sleep pattern inference
- Gap analysis → silence periods, conversation boundaries
- Activity cliffs → tier boundaries in group chat rankings

### 2.3 Content Analysis

**Word Frequency (Chinese):**
1. Filter to text messages only
2. Extract Chinese character n-grams (2-4 chars)
3. Remove stop words (的, 了, 是, 我, 你, 他, etc.)
4. Count frequency, take top 50-100

**Word Frequency (English/Mixed):**
1. Extract English words (regex: `[a-zA-Z]{2,}`)
2. Filter common stop words
3. Count frequency

**Keyword Matching:**
Define topic-specific keyword lists, count matches per topic. Adapt to actual chat content — read multiple segments to discover topic-specific vocabulary.

### 2.4 Interaction Patterns

**Conversation Rounds:**
- Define a "round" as continuous messages without a long gap (default: 30 min)
- Count messages per round
- Track who initiates each round
- Compute round length distribution

**Reply Speed:**
- For consecutive messages from different senders, compute time delta
- Report median and mean reply times
- Only consider replies within 1 hour
- Group by sender for per-user analysis

**Initiation Patterns (Private Chat):**
- Count topic initiations per sender
- Track who responds vs who initiates
- Compute active/passive ratio per sender

**Social Network (Group Chat):**
- Track reply targets (who replies to whom)
- Compute social connection strength
- Identify key connectors and isolated users

### 2.5 Media Analysis

| Metric | Method |
|--------|--------|
| Image count | Count image-type messages |
| Image density | images per message |
| Forward messages | Count and categorize forwarded content |
| Image-to-text ratio | total_images / total_text_messages |
| Image sender distribution | Who shares the most images |
| Image time distribution | When images are shared |

## 3. Deep Analysis Framework

### 3.1 Activity Cliff Detection (Group Chat)

For group chats with many participants, identify "activity cliffs":

```
User A: 2638 messages  ─┐
User B: 2308 messages   │ Tier 1 (very active)
User C: 1500 messages  ─┘ ← Cliff 1 (43% drop)
User D: 1200 messages  ─┐
User E: 800 messages    │ Tier 2 (active)
                       ─┘ ← Cliff 2
User F: 300 messages   ──  Tier 3 (casual)
```

Detection criteria:
1. Drop percentage > 15% AND absolute drop > 80 messages
2. OR drop is 2.5x larger than average of next 3 drops

Report tiers to user for decisions on how many users to profile individually.

### 3.2 Topic Analysis Pipeline

1. **Discovery Phase**: Read 8-10 segments across the full date range to identify all major topics
2. **Classification Phase**: Create keyword lists for each topic, scan full dataset
3. **Deep Dive Phase**: For each topic, collect representative quotes, analyze patterns
4. **Cross-Analysis Phase**: Topic x Time, Topic x Sender, Topic x Emotion

**Group chat specific topics:**
- Bot interaction patterns (wife-collecting, gacha systems, economy)
- Forward/搬屎 culture
- Group events (K歌, 线下聚会)
- Internal conflicts and drama
- Meme propagation chains

### 3.3 Topic Report Structure

Each topic report should contain:
1. 话题概览 (Overview) — frequency, percentage, time span
2. 详细子话题分析 (Sub-topics) — 5+ sub-topics with evidence
3. 时间演变 (Evolution) — how the topic changes over time
4. 情感维度 (Emotional dimension) — sentiment within the topic
5. 关系意义 (Relationship significance) — what this topic reveals
6. 社会文化解读 (Sociocultural interpretation) — broader context
7. 总结 (Summary) — key insights

### 3.4 Personality Profiling Framework

**Private chat structure:**
1. 基本信息 (Basic info)
2. 语言风格 (Language style)
3. 沟通策略 (Communication strategy)
4. 兴趣图谱 (Interest map)
5. 情感表达 (Emotional expression)
6. 社交互动 (Social interaction)
7. 关键时刻 (Key moments)
8. 局限性声明 (Limitations)

**Group chat structure (extended):**
1. 基本信息 (Basic info) — message count, active days, rank
2. 语言风格 (Language style) — vocabulary, sentence patterns, signature phrases
3. 沟通策略 (Communication strategy) — topic initiation, response patterns
4. 兴趣图谱 (Interest map) — keyword-based interest areas
5. 社交互动 (Social interaction) — reply targets, social role
6. 情感表达 (Emotional expression) — humor style, emotional range
7. Bot互动模式 (Bot interaction) — wife-collecting, gacha, economy
8. 关键时刻 (Key moments) — 5-10 representative excerpts
9. 局限性声明 (Limitations)
10. 附录 (Appendix) — additional data tables

### 3.5 Brilliant & Legendary Quotes Curation (Group Chat)

**Selection criteria for 精彩发言 (Brilliant):**
- 接龙/刷屏: Mass coordinated responses
- 神回复: Unexpectedly clever or funny replies
- 经典对话: Multi-turn conversations with comedic timing
- 梗传播: Moments when memes spread or evolve
- 犀利吐槽: Sharp wit or observational humor
- 荒诞对话: Surreal or absurdist exchanges

**Selection criteria for 神人发言 (Legendary):**
- Abstract/absurdist literary works
- Copypasta-worthy monologues
- Dramatic emotional outbursts
- Philosophical rants
- Viral moments that defined the group culture

**For each entry, provide:**
1. Original chat text (preserve integrity)
2. Context explanation (cause and effect)
3. Quoteblock analysis (why it's funny/notable)
4. High-risk warning if applicable (red border, no censorship)

### 3.6 Evidence Standards

**Good evidence:**
```
用户A：今天考试考砸了，心情很差
用户B：别想了，出去吃顿好的
```
→ Evidence of: emotional support behavior, coping through food

**Bad evidence:**
"用户A seems to be a negative person"
→ No direct quote, subjective judgment without basis

**Rules:**
- Every claim needs ≥1 direct quote
- Quotes should be 2-5 lines of context (not single words)
- Distinguish observation from inference
- Use qualifying language for inferences

## 4. Report Writing Standards

### 4.1 Structure

Every report file should follow:
```markdown
# Title

## 一、Section 1
### 1.1 Sub-section
Content with evidence...

## 二、Section 2
...
```

### 4.2 Evidence Format

Use code blocks for chat quotes:
```markdown
**观察描述：**
```
发送者：原始聊天内容
发送者：原始聊天内容
```
**分析：** 基于证据的分析...
```

### 4.3 Table Usage

Use tables for:
- Comparative data (Person A vs Person B)
- Frequency distributions
- Timeline summaries
- Classification matrices
- Per-user statistics

### 4.4 Length Guidelines

| Report Type | Minimum Lines | Target Lines |
|-------------|---------------|--------------|
| Statistical analysis | 200 | 300-400 |
| Comprehensive report | 300 | 500-700 |
| Topic report | 300 | 400-600 |
| Personality profile (private) | 400 | 600-800 |
| Personality profile (group) | 350 | 400-600 |
| Brilliant quotes | 300 | 400-600 |
| Legendary quotes | 300 | 400-600 |

## 5. Output Rendering

### 5.1 Output Modes

| Mode | Description | When to use |
|------|-------------|-------------|
| `html` (default) | Styled HTML with navigation | Most cases — polished, shareable |
| `md` | Raw Markdown files | Quick review, further editing |
| `all` | Both HTML and Markdown | When user wants both formats |

### 5.2 HTML Design Principles

- **Apple-style aesthetic**: Clean, minimal, whitespace-heavy
- **CSS variables**: Enable light/dark theme with single attribute toggle
- **localStorage persistence**: Theme choice survives page navigation
- **Responsive**: Mobile-friendly with media queries
- **No external dependencies**: Pure CSS, no JS frameworks
- **Breadcrumb navigation**: Every page shows its location
- **Leaderboard**: Group chat homepage features interactive ranking

### 5.3 Navigation Structure

```
index.html              — Main page (leaderboard for group chat)
├── brilliant.html      — 精彩发言展示页
├── legendary.html      — 神人发言展示页
├── analyse/index.html  — Category index
│   └── *.html          — Individual reports
├── report/index.html
│   └── *.html
├── topic/index.html
│   └── *.html
└── personality/index.html
    └── *.html
```

Each page has breadcrumb navigation: 首页 > 分类 > 文章

## 6. File Cleanup

After task completion, clean up temporary and intermediate files while preserving essential outputs.

### Preserve
- Original data (user-provided chat files, JSONL batches)
- Finalized markdown documents (all report .md files)
- Final output (html/ directory, chat_stats.json)
- Generated scripts (in output/scripts/ for user reference)

### Clean Up
- Intermediate text extracts from sub-agents (e.g., `*_messages.txt`)
- Debug logs, temporary test outputs
- Duplicate files from failed/retried agent runs
- Empty or near-empty files that were superseded

### Timing
Execute cleanup after Phase 3 rendering is complete, before reporting task completion to the user.
