# ChatAnalysis.SKILL

English | [中文](README.md)

A deep analysis tool for chat records. Feed in raw chat data, get back structured statistical reports, personality profiles, topic breakdowns, and curated highlights -- all rendered as browsable HTML pages.

## What It Does

```
Raw chat data ──→ Parsing & Stats ──→ Deep Analysis Reports ──→ Markdown (core)
                   │                    │                         │
                   │                    ├─ Statistical (5)        └─→ HTML (recommended)
                   │                    ├─ Comprehensive (4)          ├─ 5 preset themes
                   │                    ├─ Topic reports (N)          ├─ Light / dark mode
                   │                    ├─ Personality profiles       ├─ TOP N leaderboard
                   │                    ├─ Highlights / Legends       ├─ Breadcrumb nav
                   │                    └─ Chat Adventures (opt.)     └─ Responsive layout
                   └─ Message stats, time distribution, word freq,
                      interaction patterns, reply speed, activity cliffs
```

## Supported Chat Scenarios

| Scenario | Description | Highlights |
|----------|-------------|------------|
| **Private Chat** | One-on-one conversation | Relationship dynamics, side-by-side profiles |
| **Group Chat** | Multi-user group | Social network, activity rankings, Bot interactions, meme culture |

## Supported Data Formats

**No format restrictions.** As long as your data contains three elements -- timestamps, senders, and messages -- it can be analyzed. Scripts are generated at runtime based on the actual data format.

Common format examples:

| Format | Description |
|--------|-------------|
| JSONL batch files | Structured JSON Lines (e.g., exported by QQ Chat Exporter), with `timestamp` + `sender` fields |
| Plain text | Chat log text from any platform, with date headers and sender prefixes (e.g., `## 2026-01-01 09:30` / `UserA: content`) |

## Design Philosophy

### Breadth-first, Deep-parallel

1. **Extract dimensions broadly first** -- Before diving deep into any single dimension, scan the full picture: statistical metrics, time distribution, content features, interaction patterns, media usage, topic distribution, user profiles.
2. **Then parallelize deep analysis** -- Use sub-agents to process deep reports for each dimension in parallel, with each agent focusing on one domain.

### Leverage Sub-agents

**Principle: When the workload is large and parallelism is needed, sub-agents are the most efficient approach.** Tasks that can be done by sub-agents should not be executed synchronously in the main conversation.

- Report generation -> multiple sub-agents in parallel (core strategy of Phase 2)
- Data cleaning / script execution -> sub-agents run while the main conversation continues configuration
- Repetitive work (batch file processing, format conversion) -> sub-agents batch process

**Exception for small tasks:** When there is little work, do it directly in the main conversation without spawning sub-agents.

### Evidence-based Analysis

- **Data and evidence driven** -- Every conclusion must be supported by original chat record quotes.
- **Rational analysis first** -- Objectively describe observable behavior; emotional interpretation is secondary.
- **No assumed stance** -- Do not pre-assume legal, moral, or ethical positions in analysis.
- **User bears all responsibility** -- Output only provides sensitive content warnings (md blockquotes / HTML red-bordered divs); no suppression, hiding, or blurring of content.

### Relative Path Principle (T0)

**All paths in output must use relative paths.** This is a T0-level principle that applies to all output formats (md and html):

- `<a>` links -- use relative paths (e.g., `../index.html`, `personality/username.html`)
- Media references -- use relative paths (e.g., `resources/images/xxx.jpg`)
- Navigation / breadcrumb links -- use relative paths
- CSS/JS references -- use relative paths (e.g., `.assets/style.css`)
- **Forbidden**: absolute paths, `file://` protocol, hardcoded domains

> In-page anchors (`#id`) are not paths and are exempt from this rule. `<a href="#section-id">` is a same-page jump and must use the bare `#id` format without any relative path prefix.

### Partial Perspective

- Chat records are only a slice of a user's life.
- Analysis of any user is partial and lacks real-life context.
- Use qualifying language: "observable from chat", "exhibits in this scenario".
- **Subjective outputs** (sharp commentary, highlights, legendary quotes) should avoid overgeneralizing, while staying true to user intent.

## Key Rules

- **No git operations during the task** (commit/push/merge, etc.) unless the user explicitly requests it.
- **All data for a single task must come from the same private or group chat** -- mixing across chats is prohibited.
- **TOKEN warning**: This SKILL consumes significant TOKEN (often millions). Make sure you can afford it before proceeding.
- **Incremental data support**: You may append new chat data mid-task (e.g., newly exported JSONL files), but must confirm it belongs to the same chat.

## Installation

### Prerequisites

- An Agent platform compatible with the [AgentSkill specification](https://github.com/anthropics/claude-code/blob/main/docs/agent_skills.md) (e.g., [Claude Code](https://docs.anthropic.com/en/docs/claude-code))

### Recommended Environment

- Python 3.8+ (parsing and HTML rendering scripts are generated at runtime; Python is recommended)
- `pip install markdown` (required for HTML rendering; the tool will prompt you at runtime)

> This SKILL is a general-purpose specification, not tied to any specific Agent platform or environment. The Agent adapts the script implementation based on what's actually available.

### Installation Steps

```bash
# Clone the repository
git clone https://github.com/JularDepick/ChatAnalysis.SKILL.git
cd ChatAnalysis.SKILL
```

## How to Use

Register this SKILL with your Agent platform, then trigger a chat analysis task. Example with Claude Code:

```
/chat-analysis
```

Then follow the prompts to configure step by step:

1. **Chat platform** -- QQ, WeChat, Telegram, Discord, WhatsApp, or auto-detect
2. **Chat mode** -- Private / group (auto-detectable)
3. **Focus user** -- Your own account
4. **Output directory** -- Where to write results
5. **Output style** -- `md` (core only), `html` (core + rendering, recommended), or `all` (core + rendering + preserve md)
6. **HTML visual theme** -- Choose from 5 preset themes (default: Apple)
7. **HTML page layout** -- Choose from 4 layout templates (default: Standard)
8. **Full chat record long page** (optional) -- Whether to generate a single-page HTML with all messages (default: off)
9. **Footer tech support link** (optional) -- Whether to show a project link in the HTML footer
10. **Narrative perspective** -- Third-person (default) or first-person
11. **Sensitive content scope** -- Media files only (default) or all content
12. **Sensitive content showcase** (optional) -- Must be explicitly requested to enable
13. **Max parallel sub-agents** -- Upper limit for parallel report generation (private: default 3, group: default 5, max 9)
14. **Avatar embedding** (optional) -- Whether to embed avatars from the raw data (default: off)

### Extra Options for Group Chats

In group chat mode, you will be asked additionally:

- **TOP N** -- Generate personality profiles for the top N most active users (default: 10, max: 20)
- **more/ standalone pages** (all off by default, select individually) -- Highlights page (`more/brilliant/index.html`, 20-30 curated quotes with meme culture analysis) and Legends page (`more/legendary/index.html`, 10-20 extraordinary quotes)
- **Chat Adventures** -- Whether to write adventure diaries from different users' first-person perspectives
- **Real identity data** -- Whether to allow real identity data (real names, phone numbers, etc.) in group chat output (default: anonymized)

### What You Get

**Private Chat:**
- Homepage title (both nicknames, e.g., "Chat Analysis: Zhang San & Li Si")
- Side-by-side statistics comparison
- Relationship dynamics analysis
- Topic reports
- Dual personality profiles

**Group Chat:**
- Homepage title (group name) + TOP N activity leaderboard (clickable to jump to profiles)
- Personality profiles per user
- Topic reports
- Highlights / Legends showcase pages
- Activity cliff detection (tier-based)
- Chat Adventures (optional)

## Output Directory Structure

The `md/` directory contains the core cleaned output; `html/` is rendered from `md/` (recommended). Below is the full directory structure for HTML output mode:

```
workdir/
├── data/                          <- Raw data (read-only, never modified)
└── .output/                      <- All output
    ├── chat_stats.json           <- Statistics (JSON)
    ├── README.ai.md              <- Reuse guide for agents
    ├── scripts/                  <- Runtime-generated scripts
    ├── .cache/                   <- Temp files / intermediates (cleaned after completion)
    ├── md/                       <- Markdown reports (core output)
    │   ├── *.md                  <- Flat by default
    │   ├── analyse/              <- Statistical analysis (when grouped)
    │   ├── report/               <- Comprehensive reports
    │   ├── topic/                <- Topic reports
    │   ├── personality/          <- Personality profiles
    │   ├── brilliant.md          <- Highlights
    │   ├── legendary.md          <- Legends
    │   └── adventures/              <- Chat Adventures (optional)
    └── html/                     <- HTML pages (each .xxx/ is independently deployable)
        ├── .full/    <- Full version (default; sensitive content hidden in <details>)
        │   ├── .assets/          <- CSS/JS
        │   ├── resources/        <- Media files
        │   ├── index.html        <- Homepage + leaderboard
        │   ├── analyse/          <- Statistical analysis
        │   ├── report/           <- Comprehensive reports
        │   ├── topic/            <- Topic reports
        │   ├── personality/      <- Personality profiles
        │   ├── adventures/          <- Chat Adventures
        │   │   ├── index.html   <- Entry page (user list / navigation)
        │   │   └── <user>.html
        │   └── more/             <- Standalone fun pages
        │       ├── brilliant/
        │       │   └── index.html
        │       └── legendary/
        │           └── index.html
        ├── .parts/     <- Chunked output (optional)
        │   ├── .assets/          <- CSS/JS
        │   └── resources/        <- Media files
        ├── .whole/<- Full chat record long page (optional)
        │   ├── .assets/          <- CSS/JS
        │   ├── resources/        <- Media files
        │   └── chat.html         <- All messages in a single page
        ├── .pure/                <- Clean version (sensitive blocks removed, same structure)
        │   ├── .assets/          <- CSS/JS
        │   ├── resources/        <- Media files
        │   ├── index.html
        │   ├── analyse/
        │   ├── report/
        │   ├── topic/
        │   ├── personality/
        │   ├── adventures/
        │   │   ├── index.html
        │   │   └── <user>.html
        │   └── more/
        │       ├── brilliant/
        │       │   └── index.html
        │       └── legendary/
        │           └── index.html
        └── .sensitive/           <- Sensitive markers / showcase (optional)
            ├── .assets/          <- CSS/JS
            └── resources/        <- Media files
```

Open `html/.full/index.html` to browse all content.

## Optional Fun Modules

The following modules are off by default and must be explicitly enabled when starting a task:

### Highlights / Legends

Automatically suggested in group chat mode. Highlights: 20-30 curated quotes. Legends: 10-20 extraordinary quotes. Each comes with deep analysis and meme culture tracing.

### Chat Adventures

Adventure diaries written from different users' first-person perspectives. Each entry is 150-250 lines, based on real chat events -- creative but not fabricated. Strictly marked with AI-generated disclaimers. The directory contains an `index.html` entry page (user list / navigation) and individual pages per user.

### Full Chat Record Long Page

Generates a single-page HTML with all messages, browsable along the timeline. Messages are grouped into 5-minute intervals. Sensitive content is individually collapsible via `<details>`. Media files are inserted as links (not embedded). Contains footer links (if allowed by user). Outputs to `html/.whole/`, independently deployable.

### TOP N Leaderboard + Activity Cliff Detection

In group chat mode, generates an activity ranking. Top three are highlighted; clicking jumps to the personality profile. Automatically detects activity cliffs (tier-based), distinguishing core active, moderately engaged, and lurking users.

### HTML Style Themes

5 preset visual styles, chosen at task startup:

| Theme | Style | Accent |
|-------|-------|--------|
| Apple (default) | Apple-inspired, clean and minimal | `#007aff` blue |
| Nord | Nordic cool tones, soft and muted | `#88c0d0` frost blue |
| Dracula | Dark purple, predominantly dark | `#bd93f9` purple |
| Gruvbox | Retro warm tones, nostalgic feel | `#d79921` gold |
| Solarized | Balanced low contrast, eye-friendly | `#268bd2` cyan |

### HTML Page Layouts

4 layout structures, chosen at task startup:

| Layout | Max Width | Navigation | Description |
|--------|-----------|------------|-------------|
| Standard (default) | 880px | Breadcrumb | Single-column centered, suits most scenarios |
| Wide | 1200px | Right-side TOC | Wide-screen layout, great for long reports |
| Magazine | 880px | Breadcrumb | Homepage two-column card grid, great for multi-topic group chats |
| Compact | 880px | Breadcrumb | Dense and compact, ideal for browsing lots of content |

## Output Page Features

- **Pure frontend** -- Strictly HTML + CSS + JS, no external dependencies
- **Independently deployable** -- Each `.xxx/` directory is fully self-contained (.assets + resources + pages), deployable to any static server
- **Light / dark mode** -- One-click toggle, persisted across pages
- **Relative paths** -- All links use relative paths, freely portable
- **Responsive** -- Mobile-friendly, 768px breakpoint

## Important Notes

### Sensitive Content Handling

- Serious illegal / pornographic content is automatically collapsed with `<details>` (HTML) or wrapped in comment markers (Markdown)
- General sensitive content (controversial remarks, etc.) is kept in the main text with visual warnings (md blockquotes / HTML red-bordered divs)
- By default, sensitive content judgment applies to media files only; optionally extended to all content
- Three form directories derive from the base output: `.full/` (full version with `<details>`), `.pure/` (sensitive blocks removed), `.sensitive/` (sensitive markers / showcase)
- Severity distinction: serious violations (illegal/pornographic) are auto-hidden; general sensitive content (value differences, controversial remarks) receives visual warnings only
- Optional "Sensitive Content Showcase" module -- must be explicitly requested to enable

### Privacy

- Real identities (real names, phone numbers, etc.) of group chat members are anonymized by default
- In private chats, both parties' identities are allowed (both already know each other)
- At task startup, you will be asked whether to allow real identity data in the output

### Data Requirements

- All data for a single task must come from **the same private or group chat** -- mixing across chats is prohibited
- You may append new chat data mid-task, but must confirm it belongs to the same chat
- To analyze multiple chats, start separate tasks for each

### Narrative Perspective

- Third-person narrative by default (no "I")
- First-person available (you must specify which account "I" refers to)

### Key Rules

- Do not perform git operations (commit/push/merge, etc.) during the task unless explicitly requested
- All output paths must use relative paths; absolute paths and `file://` protocol are forbidden

## FAQ

**Q: What Python command should I use on Windows?**
Use `python`, not `python3`.

**Q: How long does analysis take?**
Depends on message volume and sub-agent count: under 1K messages takes about 2-5 minutes; 1K-5K messages takes about 5-10 minutes; 5K-25K messages takes about 10-20 minutes; over 25K messages takes about 20-30 minutes.

**Q: A Bot account generates too many messages -- what do I do?**
Consider excluding the Bot from the "active users" ranking and analyzing it separately.

**Q: Can I deploy the output to a web page?**
Yes. All output is pure frontend HTML with relative paths. Just upload to any static hosting.

## Related Links

| Project | Description |
|---------|-------------|
| [QQ Chat Exporter (QCE)](https://github.com/shuakami/qq-chat-exporter) | QQ chat record export tool. The JSONL format in this SKILL is primarily designed for QCE exports |
| [ChatAnalysis.SKILL](https://github.com/JularDepick/ChatAnalysis.SKILL) | This project's repository |

## License

This project is for personal study and research purposes only. Please comply with the relevant platform terms of service and local laws and regulations.
