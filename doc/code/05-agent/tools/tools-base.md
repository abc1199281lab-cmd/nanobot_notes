# Tool Base - 工具基礎類別

**檔案路徑**: `nanobot/agent/tools/base.py`  
**職責**: 定義所有工具的通用介面和驗證邏輯

---

## 1. 模組概述

`Tool` 是抽象基礎類別，定義工具必須實現的介面和參數驗證邏輯。

---

## 2. 核心類別: `Tool`

### 2.1 抽象屬性

| 屬性 | 說明 |
|------|------|
| `name: str` | 工具名稱（用於函式呼叫） |
| `description: str` | 工具描述 |
| `parameters: dict` | JSON Schema 參數定義 |

### 2.2 抽象方法

| 方法 | 說明 |
|------|------|
| `async execute(**kwargs) -> str` | 執行工具 |

### 2.3 參數驗證

```python
def validate_params(params: dict) -> list[str]:
    # 回傳錯誤列表，空列表表示驗證通過
```

支援的驗證：
- 類型檢查（string, integer, number, boolean, array, object）
- 範圍檢查（minimum, maximum）
- 長度檢查（minLength, maxLength）
- 枚舉檢查（enum）
- 必需欄位（required）

### 2.4 格式轉換

```python
def to_schema() -> dict:
    # 轉換為 OpenAI 函式格式
```

---

## 3. 實作範例

```python
from nanobot.agent.tools.base import Tool

class MyTool(Tool):
    @property
    def name(self) -> str:
        return "my_tool"
    
    @property
    def description(self) -> str:
        return "Does something useful"
    
    @property
    def parameters(self) -> dict:
        return {
            "type": "object",
            "properties": {
                "input": {"type": "string"}
            },
            "required": ["input"]
        }
    
    async def execute(self, input: str, **kwargs) -> str:
        return f"Processed: {input}"
```

---

## 4. 相關文件

- [Tools Registry](./tools-registry.md)
