# ContextBuilder - 上下文建構器

**檔案路徑**: `nanobot/agent/context.py`  
**職責**: 組裝系統提示詞和對話訊息，為 LLM 提供完整上下文

---

## 1. 模組概述

ContextBuilder 負責將各種資訊來源（啟動檔案、記憶、技能、對話歷史）整合成連貫的 LLM 提示詞。它實現了「漸進式載入」策略：

- **總是載入**: 核心身份、長期記憶、always-技能
- **按需載入**: 一般技能僅顯示摘要，Agent 用 `read_file` 讀取詳細內容

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `context.py` | ContextBuilder 類別 |

---

## 3. 核心類別: `ContextBuilder`

### 3.1 用途

建構完整的 LLM 提示詞，包括系統提示詞和對話訊息列表。

### 3.2 初始化 (`__init__`)

```python
def __init__(self, workspace: Path):
    self.workspace = workspace
    self.memory = MemoryStore(workspace)      # 記憶存取
    self.skills = SkillsLoader(workspace)     # 技能載入
```

**參數**:
- `workspace`: 工作目錄路徑，用於定位啟動檔案和記憶檔案

**啟動檔案清單** (`BOOTSTRAP_FILES`):
| 檔案 | 用途 |
|------|------|
| `AGENTS.md` | Agent 指令與指南 |
| `SOUL.md` | 個性與價值觀 |
| `USER.md` | 使用者偏好 |
| `TOOLS.md` | 工具使用說明 |
| `IDENTITY.md` | 身份設定 |

### 3.3 公開方法

#### `build_system_prompt(skill_names: list[str] | None = None) -> str`

建構完整的系統提示詞。

**組成部分**（依序）:
1. **核心身份** (`_get_identity`): nanobot 的基本介紹、當前時間、執行環境
2. **啟動檔案** (`_load_bootstrap_files`): 工作目錄下的 .md 檔案
3. **記憶上下文** (`memory.get_memory_context`): 長期記憶 + 今日筆記
4. **總是啟用的技能** (`skills.load_skills_for_context`): 標記 `always=true` 的技能
5. **可用技能摘要** (`skills.build_skills_summary`): XML 格式的技能列表

**回傳**: 完整的系統提示詞字串

#### `build_messages(...) -> list[dict[str, Any]]`

建構完整的訊息列表供 LLM 呼叫。

```python
def build_messages(
    self,
    history: list[dict[str, Any]],      # 歷史訊息
    current_message: str,               # 當前使用者訊息
    skill_names: list[str] | None = None,  # 要包含的技能
    media: list[str] | None = None,     # 圖片/媒體檔案路徑
    channel: str | None = None,         # 當前頻道
    chat_id: str | None = None,         # 當前聊天 ID
) -> list[dict[str, Any]]:
```

**回傳格式** (OpenAI 格式):
```python
[
    {"role": "system", "content": "..."},    # 系統提示詞
    {"role": "user", "content": "..."},      # 歷史訊息 1
    {"role": "assistant", "content": "..."}, # 歷史回應 1
    {"role": "user", "content": [...]},      # 當前訊息（含圖片）
]
```

**圖片處理**:
- 支援本地圖片檔案（jpg, png, gif）
- 自動轉換為 base64 編碼
- 以多模態格式傳給 LLM

#### `add_tool_result(...) -> list[dict[str, Any]]`

添加工具執行結果到訊息列表。

```python
def add_tool_result(
    self,
    messages: list[dict[str, Any]],
    tool_call_id: str,      # 工具呼叫 ID
    tool_name: str,         # 工具名稱
    result: str            # 執行結果
) -> list[dict[str, Any]]:
```

**回傳**: 更新後的訊息列表（添加 tool role 訊息）

#### `add_assistant_message(...) -> list[dict[str, Any]]`

添加助手回應到訊息列表（含工具呼叫）。

```python
def add_assistant_message(
    self,
    messages: list[dict[str, Any]],
    content: str | None,              # 回應內容
    tool_calls: list[dict] | None,   # 工具呼叫列表
) -> list[dict[str, Any]]:
```

### 3.4 私有方法

#### `_get_identity() -> str`

建構核心身份區塊，包含：
- nanobot 介紹
- 可用工具清單
- 當前時間
- 執行環境資訊
- 工作目錄路徑

#### `_load_bootstrap_files() -> str`

從工作目錄載入所有啟動檔案。

#### `_build_user_content(...) -> str | list[dict]`

建構使用者訊息內容，支援多模態（文字 + 圖片）。

---

## 4. 使用範例

### 4.1 基本使用

```python
from nanobot.agent.context import ContextBuilder

context = ContextBuilder(Path.home() / ".nanobot" / "workspace")

# 建構系統提示詞
system_prompt = context.build_system_prompt()

# 建構完整訊息
messages = context.build_messages(
    history=[{"role": "user", "content": "你好"}],
    current_message="幫我寫一個 Python 函式",
    channel="cli",
    chat_id="direct",
)
```

### 4.2 多模態訊息

```python
# 傳送圖片
messages = context.build_messages(
    history=[],
    current_message="描述這張圖片",
    media=["/path/to/image.jpg"],
)
```

### 4.3 Agent Loop 中的使用

```python
# AgentLoop._process_message 中的典型流程
messages = self.context.build_messages(
    history=session.get_history(),
    current_message=msg.content,
    media=msg.media if msg.media else None,
    channel=msg.channel,
    chat_id=msg.chat_id,
)

# 呼叫 LLM
response = await self.provider.chat(
    messages=messages,
    tools=self.tools.get_definitions(),
)

# 如有工具呼叫
if response.has_tool_calls:
    messages = self.context.add_assistant_message(
        messages, response.content, tool_call_dicts
    )
    # ... 執行工具 ...
    messages = self.context.add_tool_result(
        messages, tool_call_id, tool_name, result
    )
```

---

## 5. 設計考量

### 5.1 安全性

- **路徑解析**: 所有檔案路徑經過 `expanduser().resolve()` 處理
- **無外部 URL**: 不直接存取遠端 URL，僅處理本地檔案

### 5.2 效能

- **漸進式載入**: 大技能檔案不預設載入，減少 Token 消耗
- **Base64 快取**: 圖片編碼無快取，每次重新編碼

### 5.3 擴充性

- **啟動檔案可擴展**: 修改 `BOOTSTRAP_FILES` 常數即可
- **技能系統**: 與 `SkillsLoader` 整合，支援動態技能

### 5.4 已知限制

1. **圖片大小**: 大圖片會增加 Token 消耗
2. **歷史長度**: 不會自動截斷，需由呼叫方控制
3. **YAML 解析**: 技能 Frontmatter 使用簡易解析，不支援複雜 YAML

---

## 6. 相關文件

- [Agent Core](./agent-core.md) - Agent 主循環
- [Agent Memory](./agent-memory.md) - MemoryStore 記憶系統
- [Agent Skills](./agent-skills.md) - SkillsLoader 技能載入
- [Session Manager](../07-session/session.md) - 會話歷史
