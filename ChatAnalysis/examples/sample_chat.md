# Sample Chat Log Format

This file demonstrates the expected chat log format for the analysis skill.

## QQ Format (Primary)

```
## 2026-01-01 09:30

用户A：早上好
用户B：早
用户A：今天天气不错
用户B：[图片]
用户B：确实

## 2026-01-01 12:15

用户A：吃了吗
用户B：还没
用户A：一起去食堂？
用户B：ok

## 2026-01-02 23:45

用户A：困死了
用户B：早点睡
用户A：还想再刷会视频
用户B：[图片]
```

## Key Format Rules

1. **Date header**: `## YYYY-MM-DD HH:MM` (one per time segment)
2. **Sender line**: `Name：content` (Chinese colon ：)
3. **Image tag**: `[图片]` on its own line
4. **Card message**: `[卡片消息: title]`
5. **Empty lines**: Separate date blocks

## Other Supported Formats

### WeChat Export
```
2026-01-01 09:30:00
用户A
早上好

2026-01-01 09:31:00
用户B
早啊
```

### Telegram Export
```
[2026.01.01 09:30:00] User A: Good morning
[2026.01.01 09:31:00] User B: Morning!
```

### Generic Format
```
2026-01-01 09:30 - Alice: Hello
2026-01-01 09:31 - Bob: Hi there
```

## Tips for Best Results

- Ensure consistent date headers (no missing dates)
- Use consistent sender names (don't alternate between "User A" and "usera")
- Keep one message per line
- Include image/card tags even though content is missing
- Remove system messages (撤回, 加入群聊) before analysis if not needed
