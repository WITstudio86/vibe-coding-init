---
description: 给特定会话 id 发送消息
argument-hint: <会话 id> <消息内容>
---

执行 Bash 命令向下游会话发送消息（末尾 & 不阻塞当前会话）：

```bash
export PATH="/usr/local/opt/node@22/bin:$PATH"
node /Applications/ZCode.app/Contents/Resources/glm/zcode.cjs \
  --prompt "<消息内容>" \
  --resume <会话 id> \
  --cwd /Users/wuzexian/ZCodeProject --mode yolo &
```
执行后回复"✅ 已触发 <会话 id>"。