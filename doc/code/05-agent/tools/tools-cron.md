# Cron Tool - 排程工具

**檔案路徑**: `nanobot/agent/tools/cron.py`  
**職責**: 管理定時任務

---

## 1. 工具: `cron`

與 CronService 互動，管理排程任務。

---

## 2. 參數

```python
{
    "action": "add",              # "add", "list", "remove"
    "message": "提醒內容",        # add 時使用
    "every_seconds": 300,         # add: 每 N 秒執行
    "cron_expr": "0 9 * * *",     # add: cron 表達式
    "job_id": "abc123"            # remove 時使用
}
```

---

## 3. 動作說明

### 3.1 `add`

創建新任務：
- 需指定 `message`
- 需指定 `every_seconds` 或 `cron_expr`
- 自動使用當前會話的 channel/chat_id 傳送

### 3.2 `list`

列出所有排程任務。

### 3.3 `remove`

刪除指定 ID 的任務。

---

## 4. 使用範例

```python
# 添加每 5 分鐘提醒
await cron_tool.execute(
    action="add",
    message="檢查郵件",
    every_seconds=300
)

# 列出任務
await cron_tool.execute(action="list")

# 刪除任務
await cron_tool.execute(action="remove", job_id="abc123")
```

---

## 5. 相關文件

- [Tools Base](./tools-base.md)
- [Cron Service](../../../08-cron/cron.md)
