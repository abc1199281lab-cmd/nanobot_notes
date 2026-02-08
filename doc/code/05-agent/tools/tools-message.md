# Message Tool - 訊息發送工具

**檔案路徑**: `nanobot/agent/tools/message.py`  
**職責**: 允許 Agent 發送訊息到聊天頻道

---

## 1. 工具: `message`

發送訊息到指定的聊天頻道。

---

## 2. 參數

```python
{
    "content": "訊息內容",        # 要發送的內容
    "channel": "telegram",         # 可選：目標頻道
    "chat_id": "12345"             # 可選：目標聊天 ID
}
```

---

## 3. 上下文設定

通常使用當前對話上下文：

```python
message_tool.set_context(channel="telegram", chat_id="12345")
# 之後呼叫只需提供 content
```

---

## 4. 使用時機

**應該使用 `message` 的情況**:
- 需要主動通知使用者（如排程任務完成）
- 向不同頻道發送訊息

**不應使用 `message` 的情況**:
- 直接回答使用者問題（直接回應即可）

---

## 5. 重要限制

```markdown
IMPORTANT: When responding to direct questions or conversations, 
reply directly with your text response.
Only use the 'message' tool when you need to send a message to 
a specific chat channel (like WhatsApp).
```

---

## 6. 相關文件

- [Tools Base](./tools-base.md)
- [Agent Core](../../agent-core.md)
