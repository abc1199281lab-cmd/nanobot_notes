# HeartbeatService - 心跳服務

**檔案路徑**: `nanobot/heartbeat/service.py`  
**職責**: 週期性喚醒 Agent 檢查任務

---

## 1. 模組概述

HeartbeatService 定期（預設每 30 分鐘）喚醒 Agent 檢查 `HEARTBEAT.md` 中的任務清單。

這類似於一個「待辦事項提醒」系統，Agent 會：
1. 讀取 `HEARTBEAT.md`
2. 執行其中列出的任務
3. 回報完成狀態

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `service.py` | HeartbeatService 類別 |

---

## 3. 核心類別: `HeartbeatService`

### 3.1 初始化

```python
def __init__(
    self,
    workspace: Path,
    on_heartbeat: Callable[[str], Coroutine[str]] | None = None,
    interval_s: int = 30 * 60,  # 30 分鐘
    enabled: bool = True,
):
```

### 3.2 HEARTBEAT.md 格式

```markdown
# Tasks for Today

- [ ] 檢查郵件
- [ ] 總結今日新聞
- [ ] 提醒會議
```

### 3.3 執行流程

1. 讀取 `HEARTBEAT.md`
2. 如為空或僅有已完成項目 → 跳過
3. 發送提示詞給 Agent
4. Agent 執行任務並回應

**提示詞**:
```
Read HEARTBEAT.md in your workspace (if it exists).
Follow any instructions or tasks listed there.
If nothing needs attention, reply with just: HEARTBEAT_OK
```

### 3.4 公開方法

| 方法 | 說明 |
|------|------|
| `async start()` | 啟動心跳服務 |
| `stop()` | 停止服務 |
| `async trigger_now() -> str \| None` | 立即觸發一次 |

---

## 4. 使用範例

```python
from nanobot.heartbeat.service import HeartbeatService

async def on_heartbeat(prompt: str) -> str:
    # 通過 Agent 處理
    return await agent.process_direct(prompt)

heartbeat = HeartbeatService(
    workspace=Path.home() / ".nanobot" / "workspace",
    on_heartbeat=on_heartbeat,
    interval_s=30 * 60
)

await heartbeat.start()
```

---

## 5. 設計考量

- **低侵入**: 無任務時幾乎無開銷
- **人類可讀**: HEARTBEAT.md 是標準 Markdown
- **可手動觸發**: `trigger_now()` 用於立即檢查

---

## 6. 相關文件

- [Agent Core](../05-agent/agent-core.md)
