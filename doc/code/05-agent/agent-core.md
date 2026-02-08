# AgentLoop - Agent 核心處理引擎

**檔案路徑**: `nanobot/agent/loop.py`  
**職責**: 訊息接收 → LLM 呼叫 → 工具執行 → 回應回傳的完整循環

---

## 1. 模組概述

AgentLoop 是 nanobot 的智能核心，負責處理所有傳入訊息的完整生命週期。它實現了一個典型的 ReAct (Reasoning + Acting) 循環架構：

```
接收訊息 → 建構上下文 → 呼叫 LLM → 執行工具 → 回傳結果 → (循環直到完成)
```

這個模組整合了 nanobot 的大部分核心功能：
- **MessageBus**: 非同步訊息佇列
- **ContextBuilder**: 系統提示詞建構
- **ToolRegistry**: 工具註冊與執行
- **SessionManager**: 會話歷史管理
- **SubagentManager**: 背景子代理

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `loop.py` | AgentLoop 主類別，核心處理引擎 |

---

## 3. 核心類別: `AgentLoop`

### 3.1 用途

AgentLoop 是 nanobot 的中央處理單元，協調所有元件來完成使用者請求。

### 3.2 初始化 (`__init__`)

```python
def __init__(
    self,
    bus: MessageBus,
    provider: LLMProvider,
    workspace: Path,
    model: str | None = None,
    max_iterations: int = 20,
    brave_api_key: str | None = None,
    exec_config: "ExecToolConfig | None" = None,
    cron_service: "CronService | None" = None,
    restrict_to_workspace: bool = False,
):
```

**參數說明**:

| 參數 | 類型 | 說明 |
|------|------|------|
| `bus` | `MessageBus` | 訊息匯流排，用於接收和發送訊息 |
| `provider` | `LLMProvider` | LLM 提供者實例 |
| `workspace` | `Path` | 工作目錄路徑 |
| `model` | `str \| None` | 使用的模型 ID，預設使用 provider 的預設模型 |
| `max_iterations` | `int` | 最大工具呼叫迭代次數，預設 20 |
| `brave_api_key` | `str \| None` | Brave Search API 金鑰 |
| `exec_config` | `ExecToolConfig \| None` | 執行工具設定 |
| `cron_service` | `CronService \| None` | 排程服務實例 |
| `restrict_to_workspace` | `bool` | 是否限制工具只能存取工作目錄 |

**初始化流程**:
1. 儲存核心參數
2. 初始化 `ContextBuilder` (系統提示詞建構)
3. 初始化 `SessionManager` (會話管理)
4. 初始化 `ToolRegistry` (工具註冊表)
5. 初始化 `SubagentManager` (子代理管理)
6. 註冊預設工具集

### 3.3 公開方法

#### `async run() -> None`

啟動 Agent 循環，持續從 MessageBus 接收並處理訊息。

```python
async def run(self) -> None:
    self._running = True
    logger.info("Agent loop started")
    
    while self._running:
        try:
            # 等待下一條訊息（1秒超時）
            msg = await asyncio.wait_for(
                self.bus.consume_inbound(),
                timeout=1.0
            )
            
            # 處理訊息
            try:
                response = await self._process_message(msg)
                if response:
                    await self.bus.publish_outbound(response)
            except Exception as e:
                # 發送錯誤回應
                await self.bus.publish_outbound(OutboundMessage(...))
                
        except asyncio.TimeoutError:
            continue  # 無訊息，繼續循環
```

**行為**:
- 無限循環直到 `stop()` 被呼叫
- 每次循環等待 1 秒超時，允許定期檢查停止信號
- 錯誤隔離：單一訊息錯誤不會影響整個循環

#### `stop() -> None`

停止 Agent 循環。

```python
def stop(self) -> None:
    self._running = False
    logger.info("Agent loop stopping")
```

#### `async process_direct(...) -> str`

直接處理訊息（適用於 CLI 或 Cron 場景，不透過 MessageBus）。

```python
async def process_direct(
    self,
    content: str,
    session_key: str = "cli:direct",
    channel: str = "cli",
    chat_id: str = "direct",
) -> str:
```

**參數**:
- `content`: 訊息內容
- `session_key`: 會話識別鍵，預設 `cli:direct`
- `channel`: 來源頻道
- `chat_id`: 聊天 ID

**回傳**: Agent 的回應文字

### 3.4 私有方法

#### `_register_default_tools() -> None`

註冊預設工具集到 `ToolRegistry`。

**註冊的工具**:
| 工具 | 類別 | 說明 |
|------|------|------|
| `read_file` | `ReadFileTool` | 讀取檔案 |
| `write_file` | `WriteFileTool` | 寫入檔案 |
| `edit_file` | `EditFileTool` | 編輯檔案 |
| `list_dir` | `ListDirTool` | 列出目錄 |
| `exec` | `ExecTool` | 執行 shell 命令 |
| `web_search` | `WebSearchTool` | 網頁搜尋 |
| `web_fetch` | `WebFetchTool` | 擷取網頁 |
| `message` | `MessageTool` | 發送訊息 |
| `spawn` | `SpawnTool` | 生成子代理 |
| `cron` | `CronTool` | 排程任務 |

#### `async _process_message(msg: InboundMessage) -> OutboundMessage | None`

處理單一傳入訊息的完整流程。

**流程**:
1. 處理系統訊息（子代理通知）→ 轉到 `_process_system_message`
2. 獲取或創建會話 (`SessionManager.get_or_create`)
3. 更新工具上下文（設定 channel/chat_id）
4. 建構初始訊息列表 (`ContextBuilder.build_messages`)
5. **Agent Loop**: 迭代呼叫 LLM 直到無工具呼叫或達到上限
   - 呼叫 LLM (`provider.chat`)
   - 如有工具呼叫 → 執行工具 → 添加結果 → 繼續循環
   - 如無工具呼叫 → 獲得最終回應
6. 儲存對話歷史到會話
7. 回傳 `OutboundMessage`

#### `async _process_system_message(msg: InboundMessage) -> OutboundMessage | None`

處理系統訊息（例如子代理完成通知）。

**特點**:
- 從 `chat_id` 解析原始頻道和聊天 ID（格式: `channel:chat_id`）
- 使用原始會話的上下文
- 標記為系統訊息儲存

---

## 4. 使用範例

### 4.1 基本啟動

```python
from nanobot.bus.queue import MessageBus
from nanobot.providers.litellm_provider import LiteLLMProvider
from nanobot.agent.loop import AgentLoop

# 建立元件
bus = MessageBus()
provider = LiteLLMProvider(
    api_key="your-api-key",
    default_model="anthropic/claude-opus-4-5"
)

# 建立並啟動 Agent
agent = AgentLoop(
    bus=bus,
    provider=provider,
    workspace=Path.home() / ".nanobot" / "workspace",
    max_iterations=20,
)

# 啟動（阻塞）
await agent.run()
```

### 4.2 CLI 直接處理

```python
# 不經過 MessageBus，直接處理
response = await agent.process_direct(
    content="幫我查今天的天氣",
    session_key="cli:default",
)
print(response)
```

### 4.3 優雅關閉

```python
# 註冊信號處理
import signal

def signal_handler(sig, frame):
    agent.stop()
    
signal.signal(signal.SIGINT, signal_handler)

# 啟動
try:
    await agent.run()
except asyncio.CancelledError:
    pass
```

---

## 5. 設計考量

### 5.1 安全性

- **工作目錄限制**: `restrict_to_workspace=True` 時，檔案工具只能存取工作目錄
- **Shell 命令保護**: ExecTool 有危險模式偵測（`rm -rf` 等）
- **超時控制**: Shell 命令有預設 60 秒超時

### 5.2 效能

- **非同步設計**: 所有 I/O 操作均為 async，避免阻塞
- **會話快取**: `SessionManager` 記憶體快取會話，減少磁碟 I/O
- **工具結果截斷**: 過長的 shell 輸出會被截斷（10000 字元）

### 5.3 擴充性

- **工具註冊**: 可動態註冊/解除註冊工具
- **提供者抽象**: `LLMProvider` 介面支援多種 LLM 後端
- **子代理**: `SubagentManager` 支援背景任務並行處理

### 5.4 已知限制

1. **記憶體使用**: 會話歷史儲存在記憶體，長時間運行可能累積
2. **LLM 依賴**: 工具呼叫循環受 LLM 能力限制
3. **單執行緒 Agent Loop**: 一次只處理一條訊息（但子代理可並行）

---

## 6. 相關文件

- [Agent Context](./agent-context.md) - 上下文建構
- [Agent Memory](./agent-memory.md) - 記憶系統
- [Agent Skills](./agent-skills.md) - 技能載入
- [Agent Subagent](./agent-subagent.md) - 子代理管理
- [Message Bus](../03-bus/bus.md) - 訊息匯流排
- [Session Manager](../07-session/session.md) - 會話管理
- [Tools Registry](./tools/tools-registry.md) - 工具註冊表
