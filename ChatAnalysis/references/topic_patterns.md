# Topic Detection Patterns

## Keyword-Based Topic Classification

Each topic category has associated keywords for automated detection.
Adapt these to the actual chat content by reading representative segments.

### Template

```python
TOPIC_KEYWORDS = {
    "topic_name": {
        "keywords": ["word1", "word2", ...],
        "description": "Brief description of what this topic covers",
        "subtopics": {
            "subtopic_1": ["specific_keyword1", "specific_keyword2"],
            "subtopic_2": ["specific_keyword3", "specific_keyword4"],
        }
    }
}
```

### Example Categories

| Category | Keywords (Chinese) | Keywords (English) |
|----------|-------------------|-------------------|
| Technology | AI, 编程, 代码, 服务器, VPN, 翻墙, GitHub, API | code, program, server, deploy |
| Entertainment | 游戏, 动漫, 视频, 电影, 音乐, B站, 抖音 | game, anime, video, movie |
| School | 考试, 作业, 绩点, 老师, 上课, 论文, 实验 | exam, homework, GPA, class |
| Food | 吃, 喝, 食堂, 外卖, 水果, 饮料, 零食 | eat, food, drink, lunch |
| Weather | 天气, 热, 冷, 下雨, 冰雹, 台风 | weather, hot, cold, rain |
| Emotion | emo, 孤独, 开心, 难过, 焦虑, 压力 | happy, sad, lonely, stressed |
| Social | 同学, 朋友, 室友, 班群, 社交 | friend, classmate, roommate |
| Family | 家, 爸, 妈, 回家, 老家 | home, family, parents |
| Hometown | 家乡, 老家, 广南, 昆明, 文山 | hometown, home city |
| Career | 面试, 实习, 工作, 简历, 考研 | interview, job, career, resume |

## Topic Discovery Process

### Step 1: Read Segments

Read 8-10 segments from the chat file, evenly distributed across the date range:
- Beginning (day 1-3)
- 25% point
- 50% point
- 75% point
- End (last 3 days)
- 3-5 random points

For each segment, note:
1. What topics appear
2. What keywords are used
3. Who initiates each topic

### Step 2: Build Keyword List

Based on discovered topics, create a comprehensive keyword list.
Include variations and slang found in the chat.

### Step 3: Full Scan

Run keyword matching against all messages to get topic frequencies.

### Step 4: Deep Analysis

For each topic with significant frequency (>5% of messages):
1. Collect all relevant messages
2. Group into sub-topics
3. Analyze temporal patterns
4. Note emotional dimensions
5. Write report with evidence

## Topic Report Template

```markdown
# 话题专题报告：[Topic Name]

## 一、话题概览
[Topic]话题在两人聊天中出现频率约X次，占比Y%。
主要集中在Z时间段。

## 二、详细子话题分析

### 2.1 [Subtopic 1]
```
用户A：原始聊天内容
用户B：原始聊天内容
```
**观察：** 基于证据的分析...

### 2.2 [Subtopic 2]
...

## 三、时间演变
[How the topic frequency changes over time]

## 四、情感维度
[Emotional content within this topic]

## 五、关系意义
[What this topic reveals about the relationship]

## 六、社会文化解读
[Broader sociocultural context]

## 七、总结
[Summary with key insights]
```
