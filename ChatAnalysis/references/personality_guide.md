# Personality Profiling Guide

## Core Principle

> 严格基于聊天记录中的可观察行为，不做推测，不以偏概全。
> 只因为我们看到了他们在一段关系中的一个面向，不代表这就是他们的全部。

## Chat Mode Differences

### Private Chat (2 senders)
- Focus on relationship dynamics between two people
- Communication style comparison
- Emotional exchange patterns
- Relationship evolution over time
- Who initiates, who responds

### Group Chat (N senders)
- Focus on individual communication style and social role
- Bot interaction patterns (wife-collecting, gacha, economy)
- Reply target mapping (social network)
- Role archetype identification
- Influence and reach within the group

## Analysis Dimensions

### 1. Language Style (语言风格)

Analyze observable patterns in how each person writes:

| Dimension | What to Look For |
|-----------|-----------------|
| Vocabulary | High-frequency words, slang, abbreviations, signature phrases |
| Sentence length | Short bursts vs. complete sentences, avg chars per message |
| Language mixing | English/Chinese code-switching patterns |
| Punctuation | Period usage, exclamation marks, ellipsis, 。。 vs ... |
| Emoji/Stickers | Frequency, types, contexts, custom emoji usage |
| Tone markers | Formal vs. casual, serious vs. playful |
| Signature phrases | Repeated expressions unique to this person |

**Evidence format:**
```
高频词汇表：
| 词汇 | 功能 | 频率 | 典型语境 |
|------|------|------|----------|
| xxx  | 惊讶 | 高频 | "xxx太离谱了" |
```

### 2. Communication Strategy (沟通策略)

| Dimension | Analysis |
|-----------|----------|
| Initiative ratio | Who starts conversations/topics? |
| Response style | Minimal ("ok") vs. elaborate |
| Topic switching | When and how they change topics |
| Silence patterns | When they go quiet, for how long |
| Message density | Burst patterns vs. even distribution |
| Active hours | Peak activity time slots |
| Message type preference | Text vs image vs forward |

### 3. Interest Map (兴趣图谱)

For each interest area, provide:
- Evidence (specific chat quotes)
- Frequency (how often mentioned, keyword count)
- Depth (casual mention vs. detailed discussion)
- Role (active pursuer vs. passive recipient)

**Keyword-based interest detection:**
Use the keyword profile from chat_stats.json to identify interests:
```
| 兴趣领域 | 关键词命中 | 占比 | 典型语境 |
|----------|-----------|------|----------|
| 游戏     | 120       | 35%  | "开黑"、"排位" |
| 二次元   | 80        | 23%  | "番剧"、"声优" |
```

### 4. Emotional Expression (情感表达)

| Type | Description | Evidence Required |
|------|-------------|-------------------|
| Direct | "我很伤心" | Exact quote |
| Indirect | "困死了" (implies frustration) | Quote + context |
| Humor | Jokes about serious topics | Quote showing humor |
| Avoidance | Topic switching when emotional | Pattern of switches |
| Dramatic | Exaggerated emotional expressions | Quote showing intensity |

### 5. Social Interaction (社交互动)

**Private chat:**
- Who initiates more
- Response time patterns
- Emotional support patterns
- Conflict resolution style

**Group chat:**
- Reply target analysis (who do they talk to most?)
- Social role: initiator, responder, lurker, bot user, meme spreader
- Influence: do others reply to them? Do they start trends?
- Connection strength with specific users

**Reply target table:**
```
| 回复对象 | 次数 | 占比 | 关系特征 |
|----------|------|------|----------|
| 用户A    | 100  | 30%  | 密切互动 |
| 用户B    | 50   | 15%  | 偶尔交流 |
```

### 6. Bot Interaction Patterns (Group Chat)

Many group chats have bot accounts that provide interactive systems:

| Pattern | Description | Analysis Focus |
|---------|-------------|----------------|
| Wife-collecting | "老婆" gacha/market system | Engagement frequency, trading patterns |
| Economy | Virtual currency (coins, silver) | Earning/spending behavior |
| Card system | Trading card games | Collection patterns, rarity awareness |
| Custom commands | User-defined bot commands | Creativity, community contribution |

### 7. Key Moments (关键时刻)

Select 5-10 defining chat excerpts that best illustrate the person's character:

**Format for each moment:**
```markdown
### 场景X：[Brief Title]

**原始对话：**
```
完整的聊天记录（3-5行）
```

**分析：**
这段对话体现了[person]的[trait]。
具体表现为：[evidence-based analysis]。
```

## Limitations Template

Every personality report MUST end with this section:

```markdown
## 局限性声明

本报告存在以下根本性局限：

1. **单一关系视角：** 聊天记录仅展示了[Name]在此群聊/对话中的表现，
   无法了解其在家庭、其他朋友、恋人、老师等关系中的表现。

2. **文字沟通局限：** 聊天记录只能反映文字表达，无法观察到
   语气、表情、肢体语言等非文字信息。

3. **选择性表达：** 人在不同社交场景中会有不同的自我呈现方式，
   这段聊天中的[Name]不一定代表其在其他场景中的状态。

4. **时间范围有限：** 聊天记录仅覆盖[start]至[end]（约X天），
   无法反映长期人格特征的变化。

5. **信息不完整：** 聊天记录中的许多信息需要上下文才能准确理解，
   缺失的上下文可能导致误读。

6. **群聊特殊性（如适用）：** 群聊中的表现可能与私聊不同，
   群体动态会影响个体行为。

**因此，本报告中的所有描述都应被视为"在这段特定关系中的可观察行为"，
而非对[Name]整体人格的判断。**
```

## Writing Quality Standards

### DO:
- Use tables for structured comparisons
- Include 2-5 line chat quotes as evidence
- Use qualifying language ("从聊天中可观察到", "在这段关系中表现出")
- Acknowledge uncertainty ("可能", "推测")
- Compare users side-by-side (private chat)
- Analyze social role and influence (group chat)
- Include keyword statistics as evidence
- Track temporal patterns (when they're active, how behavior changes)

### DON'T:
- Make sweeping personality judgments
- Speculate about mental health or disorders
- Assume motivations without evidence
- Use absolute language ("总是", "从不")
- Include actual personal information in examples (phone numbers, addresses)
- Quote sensitive content verbatim without context
- Present single observations as established patterns

## Minimum Content Requirements

| Section | Minimum Evidence |
|---------|-----------------|
| Language style | 10+ example quotes |
| Communication strategy | 5+ pattern examples |
| Interest map | 3+ interests with evidence |
| Emotional expression | 3+ direct/indirect examples |
| Social interaction | 3+ reply target examples |
| Bot interaction (group) | 2+ bot interaction examples |
| Key moments | 5-10 full chat excerpts |
| Limitations | 5+ specific limitations listed |

## Group Chat Role Archetypes

When profiling group chat users, identify their role archetype:

| Archetype | Characteristics | Evidence |
|-----------|----------------|----------|
| 话题驱动者 | Frequently starts new topics | High topic_initiation count |
| 回应者 | Primarily responds to others | Low initiation, high response |
| 潜水者 | Rarely speaks, mostly observes | Low message count, long gaps |
| Bot达人 | Heavy bot interaction | High bot mention count |
| 梗传播者 | Spreads and evolves memes | High forward count, meme keywords |
| 情感枢纽 | Connects emotionally with many | High reply_target diversity |
| 图片党 | Primarily communicates via images | High image/text ratio |
| 文学家 | Writes long, detailed messages | High avg_message_length |
