# Tools Registry - 工具註冊表

**檔案路徑**: `nanobot/agent/tools/registry.py`  
**職責**: 動態工具管理和執行

---

## 1. 模組概述

ToolRegistry 管理所有 Agent 工具的註冊、查詢和執行。它是一個中央註冊表，允許動態添加或移除工具。

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `registry.py` | ToolRegistry 類別 |
| `base.py` | Tool 抽象基礎類別 |

---

## 3. 核心類別: `ToolRegistry`

### 3.1 初始化

```python
def __init__(self):
    self._tools: dict[str, Tool] = {}
```

### 3.2 公開方法

| 方法 | 說明 |
|------|------|
| `register(tool: Tool)` | 註冊工具 |
| `unregister(name: str)` | 解除註冊 |
| `get(name) -> Tool \| None` | 獲取工具 |
| `has(name) -> bool` | 檢查是否存在 |
| `get_definitions() -> list[dict]` | 取得所有工具定義（OpenAI 格式） |
| `async execute(name, params) -> str` | 執行工具 |
| `tool_names -> list[str]` | 工具名稱列表 |

---

## 4. 使用範例

```python
from nanobot.agent.tools.registry import ToolRegistry
from nanobot.agent.tools.filesystem import ReadFileTool

registry = ToolRegistry()

# 註冊工具
registry.register(ReadFileTool())

# 取得定義給 LLM
tools = registry.get_definitions()

# 執行工具
result = await registry.execute("read_file", {"path": "README.md"})
```

---

## 5. 相關文件

- [Tool Base](./tools-base.md)
- [Filesystem Tools](./tools-filesystem.md)
