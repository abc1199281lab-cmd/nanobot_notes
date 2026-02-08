# SubagentManager - 子代理管理器

**檔案路徑**: `nanobot/agent/subagent.py`  
**職責**: 管理背景執行的子代理任務

---

## 1. 模組概述

SubagentManager 實現了「背景代理」模式，允許 Agent 產生子代理來並行處理複雜或耗時的任務，而不阻塞主 Agent 循環。

**典型場景**:
- 長時間網頁搜尋和資料整理
- 大量檔案處理
- 獨立的研究任務

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `subagent.py` | SubagentManager 類別 |

---

## 3. 核心類別: `SubagentManager`

### 3.1 用途

管理子代理的生命週期，包括創建、執行和結果通知。

### 3.2 初始化 (`__init__`)

```python
def __init__(
    self,
    provider: LLMProvider,
    workspace: Path,
    bus: MessageBus,
    model: str | None = None,
    brave_api_key: str | None = None,
    exec_config: "ExecToolConfig | None" = None,
    restrict_to_workspace: bool = False,
):
```

**參數**: 與 `AgentLoop` 相同

### 3.3 公開方法

#### `async spawn(...) -> str`

創建並啟動子代理。

```python
async def spawn(
    self,
    task: str,                    # 任務描述
    label: str | None = None,     # 顯示標籤
    origin_channel: str = "cli",  # 來源頻道
    origin_chat_id: str = "direct",  # 來源聊天 ID
) -> str:
```

**回傳**: 狀態訊息，如：
```
Subagent [研究 Python 3.12] started (id: a1b2c3d4). I'll notify you when it completes.
```

#### `get_running_count() -> int`

取得當前運行中的子代理數量。

---

## 4. 內部機制

### 4.1 子代理執行流程

1. **創建背景任務**: `asyncio.create_task()`
2. **獨立工具集**: 子代理無 `message` 和 `spawn` 工具
3. **執行 Agent Loop**: 最多 15 次迭代
4. **結果通知**: 通過 `MessageBus` 發送系統訊息

### 4.2 結果通知格式

子代理通過 `channel="system"` 的 `InboundMessage` 通知主 Agent：

```
channel: "system"
chat_id: "{origin_channel}:{origin_chat_id}"  # 編碼來源位置
content: "[Subagent '{label}' completed]\n\nTask: {task}\n\nResult: {...}"
```

### 4.3 專用系統提示詞

子代理使用精簡的提示詞，強調：
- 專注於指定任務
- 不能發送訊息或產生新子代理
- 提供清晰簡潔的結果摘要

---

## 5. 使用範例

### 5.1 Agent 產生子代理

```python
# 使用 spawn 工具
await spawn_tool.execute(
    task="搜尋最新的 Python 3.12 特性並整理成表格",
    label="Python 3.12 研究"
)
# 回傳: 子代理已啟動的確認
```

### 5.2 子代理完成處理

```python
# AgentLoop._process_system_message 自動處理
async def _process_system_message(self, msg: InboundMessage):
    # 解析 chat_id 中的來源
    origin_channel, origin_chat_id = msg.chat_id.split(":")
    # 像普通訊息一樣處理，但標記為系統訊息
```

---

## 6. 設計考量

### 6.1 隔離性

- **獨立上下文**: 子代理無法存取主 Agent 的對話歷史
- **限制工具**: 移除可能導致循環的工具
- **錯誤隔離**: 子代理錯誤不影響主 Agent

### 6.2 資源控制

- **最大迭代**: 15 次（主 Agent 為 20 次）
- **任務追蹤**: 使用 `uuid` 和字典追蹤運行任務
- **自動清理**: 完成後自動從追蹤列表移除

### 6.3 已知限制

1. **無互動**: 子代理無法與使用者互動
2. **結果長度**: 長結果可能被截斷
3. **無優先級**: 所有子代理平等競爭資源

---

## 7. 相關文件

- [Agent Core](./agent-core.md) - 主 Agent 處理系統訊息
- [Tools - Spawn](./tools/tools-spawn.md) - Spawn 工具實作
