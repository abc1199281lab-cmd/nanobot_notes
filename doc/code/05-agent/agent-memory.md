# MemoryStore - 記憶系統

**檔案路徑**: `nanobot/agent/memory.py`  
**職責**: 持久化儲存 Agent 的長期記憶和每日筆記

---

## 1. 模組概述

MemoryStore 提供簡單但有效的記憶系統，支援兩種記憶類型：

- **長期記憶** (`MEMORY.md`): 跨會話持久的重要資訊
- **每日筆記** (`YYYY-MM-DD.md`): 當日發生的事件和筆記

儲存位置：`~/.nanobot/workspace/memory/`

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `memory.py` | MemoryStore 類別 |

---

## 3. 核心類別: `MemoryStore`

### 3.1 用途

管理記憶檔案的讀寫，為 Agent 提供持久化上下文。

### 3.2 初始化 (`__init__`)

```python
def __init__(self, workspace: Path):
    self.workspace = workspace
    self.memory_dir = ensure_dir(workspace / "memory")  # 確保目錄存在
    self.memory_file = self.memory_dir / "MEMORY.md"      # 長期記憶檔案
```

**參數**:
- `workspace`: 工作目錄路徑

### 3.3 公開方法

#### `get_today_file() -> Path`

取得今天的記憶檔案路徑（格式：`YYYY-MM-DD.md`）。

#### `read_today() -> str`

讀取今天的記憶筆記。

**回傳**: 檔案內容，如不存在則回傳空字串

#### `append_today(content: str) -> None`

追加內容到今天的記憶筆記。

**行為**:
- 如檔案不存在，自動添加日期標題
- 如檔案存在，追加到現有內容

#### `read_long_term() -> str`

讀取長期記憶（`MEMORY.md`）。

#### `write_long_term(content: str) -> None`

寫入長期記憶（覆寫整個檔案）。

#### `get_recent_memories(days: int = 7) -> str`

取得最近 N 天的記憶。

```python
def get_recent_memories(self, days: int = 7) -> str:
    # 回傳最近 7 天的日記內容，以 --- 分隔
```

**參數**:
- `days`: 回顧天數，預設 7 天

**回傳**: 組合的記憶內容

#### `list_memory_files() -> list[Path]`

列出所有記憶檔案，按日期排序（最新在前）。

#### `get_memory_context() -> str`

取得記憶上下文（供 ContextBuilder 使用）。

**回傳格式**:
```markdown
## Long-term Memory
[MEMORY.md 內容]

## Today's Notes
[今日日記內容]
```

---

## 4. 使用範例

### 4.1 基本操作

```python
from nanobot.agent.memory import MemoryStore

memory = MemoryStore(Path.home() / ".nanobot" / "workspace")

# 記錄今天的發現
memory.append_today("學習了新的 Python 語法：walrus operator")

# 儲存長期偏好
memory.write_long_term("""# Long-term Memory

## User Information
- Name: Alice
- Job: Software Engineer

## Preferences
- Dark mode preferred
- Python over JavaScript
""")
```

### 4.2 在 Agent 中使用

```python
# ContextBuilder 自動整合記憶
system_prompt = context.build_system_prompt()
# 記憶內容會出現在系統提示詞中
```

---

## 5. 設計考量

### 5.1 簡單性

- **純 Markdown**: 人類可讀，易於手動編輯
- **無資料庫**: 檔案系統即儲存層
- **無結構化**: 自由格式文字，Agent 自行理解

### 5.2 自動化

- **日期處理**: 自動處理檔案命名和標題
- **目錄管理**: 自動建立所需目錄

### 5.3 已知限制

1. **無搜尋功能**: 只能按日期或全檔案讀取
2. **無去重**: 可能重複記錄相同資訊
3. **容量限制**: 長期記憶檔案可能無限增長

---

## 6. 相關文件

- [Agent Context](./agent-context.md) - 使用記憶的上下文建構
- [Agent Core](./agent-core.md) - Agent 主循環
