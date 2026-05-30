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

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed and working

### Recommended Environment

- Python 3.8+ (parsing and HTML rendering scripts are generated at runtime; Python is recommended)
- `pip install markdown` (required for HTML rendering; the tool will prompt you at runtime)

> This SKILL is a general-purpose specification, not tied to any specific environment. The Agent adapts the script implementation based on what's actually available.

### Installation Steps

```bash
# Clone the repository
git clone https://github.com/JularDepick/ChatAnalysis.SKILL.git
cd ChatAnalysis.SKILL
```

## How to Use

In Claude Code, type:

```
/chat-analysis
```

Then follow the prompts to configure step by step:

1. Chat data path (file or directory)
2. Chat mode (private / group, auto-detectable)
3. Focus user (your own account)
4. Output directory and output style (HTML / Markdown / both)
5. HTML visual theme (5 presets) and page layout (4 templates)
6. Optional module toggles (avatar embedding, full chat record page, footer links, sensitive content scope, etc.)

### Extra Options for Group Chats

In group chat mode, you will be asked additionally:

- **TOP N** -- Generate personality profiles for the top N most active users (default: 10, max: 20)
- **Highlights / Legends** -- Whether to generate curated highlight pages
- **Chat Adventures** -- Whether to write adventure diaries from different users' first-person perspectives
- **Max parallel sub-agents** -- Maximum parallel sub-agents for report generation (default: 3, max: 9)

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
- Topic reports (typically 9-12)
- Highlights / Legends showcase pages
- Activity cliff detection (tier-based)
- Chat Adventures (optional)

## Output Directory Structure

The `md/` directory contains the core cleaned output; `html/` is rendered from `md/` (recommended). Below is the full directory structure for HTML output mode:

```
workdir/
├── row/                          <- Raw data (read-only, never modified)
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
    │   └── SpeakAs/              <- Chat Adventures (optional)
    └── html/                     <- HTML pages (each .xxx/ is independently deployable)
        ├── .completed-hidden/    <- Full version (default; sensitive content hidden in <details>)
        │   ├── .assets/          <- CSS/JS
        │   ├── resources/        <- Media files
        │   ├── index.html        <- Homepage + leaderboard
        │   ├── analyse/          <- Statistical analysis
        │   ├── report/           <- Comprehensive reports
        │   ├── topic/            <- Topic reports
        │   ├── personality/      <- Personality profiles
        │   ├── SpeakAs/          <- Chat Adventures
        │   └── more/             <- Standalone fun pages
        │       ├── brilliant.html
        │       └── legendary.html
        ├── .completed-parts/     <- Chunked output (optional)
        │   ├── .assets/          <- CSS/JS
        │   └── resources/        <- Media files
        ├── .completed-whole-page/<- Full chat record long page (optional)
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
        │   ├── SpeakAs/
        │   └── more/
        └── .sensitive/           <- Sensitive markers / showcase (optional)
            ├── .assets/          <- CSS/JS
            └── resources/        <- Media files
```

Open `html/.completed-hidden/index.html` to browse all content.

## Optional Fun Modules

The following modules are off by default and must be explicitly enabled when starting a task:

### Highlights / Legends

Automatically suggested in group chat mode. Highlights: 20-30 curated quotes. Legends: 10-20 extraordinary quotes. Each comes with deep analysis and meme culture tracing.

### Chat Adventures (SpeakAs/)

Adventure diaries written from different users' first-person perspectives. Each entry is 150-250 lines, based on real chat events -- creative but not fabricated. Strictly marked with AI-generated disclaimers.

### Full Chat Record Long Page

Generates a single-page HTML with all messages, browsable along the timeline. Messages are grouped into 5-minute intervals. Sensitive content is individually collapsible via `<details>`.

### TOP N Leaderboard + Activity Cliff Detection

In group chat mode, generates an activity ranking. Top three are highlighted; clicking jumps to the personality profile. Automatically detects activity cliffs (tier-based), distinguishing core active, moderately engaged, and lurking users.

### HTML Style Themes

5 preset visual styles, chosen at task startup:

| Theme | Style |
|-------|-------|
| Apple (default) | Apple-inspired, clean and minimal |
| Nord | Nordic cool tones, soft and muted |
| Dracula | Dark purple, predominantly dark |
| Gruvbox | Retro warm tones, nostalgic feel |
| Solarized | Balanced low contrast, eye-friendly |

### HTML Page Layouts

4 layout structures, chosen at task startup:

| Layout | Description |
|--------|-------------|
| Standard (default) | Single-column centered at 880px, breadcrumb nav |
| Wide | Wide-screen at 1200px, right-side floating TOC |
| Magazine | Magazine-style two-column card grid, great for multi-topic group chats |
| Compact | Dense and compact, ideal for browsing lots of content |

## Output Page Features

- **Pure frontend** -- Strictly HTML + CSS + JS, no external dependencies
- **Independently deployable** -- Each `.xxx/` directory is fully self-contained (.assets + resources + pages), deployable to any static server
- **Light / dark mode** -- One-click toggle, persisted across pages
- **Relative paths** -- All links use relative paths, freely portable
- **Responsive** -- Mobile-friendly, 768px breakpoint

## Important Notes

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

## FAQ

**Q: What Python command should I use on Windows?**
Use `python`, not `python3`.

**Q: How long does analysis take?**
Depends on message volume: under 1K messages takes about 2-5 minutes; 5K-25K messages takes about 10-20 minutes.

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
