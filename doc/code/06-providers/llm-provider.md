# LLMProvider - LLM 提供者基礎介面

**檔案路徑**: `nanobot/providers/base.py`, `nanobot/providers/litellm_provider.py`  
**職責**: 抽象 LLM 呼叫，支援多種後端

---

## 1. 模組概述

提供者系統實現統一的 LLM 介面，支援：
- OpenRouter、Anthropic、OpenAI
- Gemini、DeepSeek、Groq
- 本地模型（Ollama、vLLM）

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `base.py` | 抽象介面和資料類別 |
| `litellm_provider.py` | LiteLLM 實作（主要提供者） |
| `transcription.py` | 語音轉錄（Groq） |

---

## 3. 資料類別

### 3.1 `ToolCallRequest`

```python
@dataclass
class ToolCallRequest:
    id: str
    name: str           # 工具名稱
    arguments: dict     # 工具參數
```

### 3.2 `LLMResponse`

```python
@dataclass
class LLMResponse:
    content: str | None
    tool_calls: list[ToolCallRequest]
    finish_reason: str
    usage: dict         # token 使用統計
    
    @property
    def has_tool_calls(self) -> bool
```

---

## 4. 抽象類別: `LLMProvider`

### 4.1 抽象方法

| 方法 | 說明 |
|------|------|
| `async chat(messages, tools, model, ...) -> LLMResponse` | 發送聊天請求 |
| `get_default_model() -> str` | 獲取預設模型 |

---

## 5. 主要實作: `LiteLLMProvider`

### 5.1 支援的提供者

自動偵測並設定：
- **OpenRouter**: `sk-or-*` API key
- **AiHubMix**: `api_base` 包含 aihubmix
- **Ollama**: `api_base` 包含 :11434
- **其他**: 根據模型名稱關鍵字

### 5.2 自動前綴

根據偵測結果自動添加提供者前綴：
- `openrouter/{model}`
- `ollama/{model}`
- `hosted_vllm/{model}`
- `dashscope/{model}`（通義千問）

### 5.3 使用範例

```python
from nanobot.providers.litellm_provider import LiteLLMProvider

provider = LiteLLMProvider(
    api_key="sk-or-...",
    default_model="anthropic/claude-opus-4-5"
)

response = await provider.chat(
    messages=[
        {"role": "system", "content": "You are helpful."},
        {"role": "user", "content": "Hello!"}
    ],
    tools=tool_definitions,
    model="anthropic/claude-opus-4-5"
)

if response.has_tool_calls:
    for tc in response.tool_calls:
        print(f"Tool: {tc.name}, Args: {tc.arguments}")
else:
    print(response.content)
```

---

## 6. 相關文件

- [Agent Core](../05-agent/agent-core.md)
- [Config Schema](../02-config/config-schema.md)
