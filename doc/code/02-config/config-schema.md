# Config Schema - 設定結構

**檔案路徑**: `nanobot/config/schema.py`, `nanobot/config/loader.py`  
**職責**: 定義和載入 nanobot 設定

---

## 1. 模組概述

使用 `Pydantic` 定義型別安全的設定結構，支援：
- JSON 設定檔
- 環境變數
- 多層級設定（Agents, Channels, Providers, Tools）

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `schema.py` | Pydantic 模型定義 |
| `loader.py` | 設定載入/儲存 |

---

## 3. 設定結構

### 3.1 根設定 (`Config`)

```python
class Config(BaseSettings):
    agents: AgentsConfig
    channels: ChannelsConfig
    providers: ProvidersConfig
    gateway: GatewayConfig
    tools: ToolsConfig
```

### 3.2 Agents

```python
class AgentDefaults:
    workspace: str = "~/.nanobot/workspace"
    model: str = "anthropic/claude-opus-4-5"
    max_tokens: int = 8192
    temperature: float = 0.7
    max_tool_iterations: int = 20
```

### 3.3 Channels

| 頻道 | 設定 |
|------|------|
| Telegram | `token`, `allow_from`, `proxy` |
| WhatsApp | `bridge_url`, `allow_from` |
| Discord | `token`, `allow_from` |
| Feishu | `app_id`, `app_secret`, `allow_from` |

### 3.4 Providers

支援的提供者：
- `openrouter`, `anthropic`, `openai`
- `deepseek`, `groq`, `gemini`
- `zhipu`, `dashscope`, `moonshot`
- `ollama`, `vllm`, `aihubmix`

### 3.5 Tools

```python
class ToolsConfig:
    web: WebToolsConfig        # Brave Search API
    exec: ExecToolConfig       # Shell 超時設定
    restrict_to_workspace: bool  # 目錄限制
```

---

## 4. 設定檔範例

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.nanobot/workspace",
      "model": "anthropic/claude-opus-4-5",
      "maxTokens": 8192,
      "temperature": 0.7
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["12345"]
    }
  },
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-..."
    }
  },
  "tools": {
    "web": {
      "search": {
        "apiKey": "BS...",
        "maxResults": 5
      }
    },
    "exec": {
      "timeout": 60
    },
    "restrictToWorkspace": false
  }
}
```

---

## 5. 環境變數

使用 `NANOBOT_` 前綴：

```bash
NANOBOT_PROVIDERS__OPENROUTER__API_KEY=sk-or-...
NANOBOT_AGENTS__DEFAULTS__MODEL=anthropic/claude-opus-4-5
```

---

## 6. 相關文件

- [CLI Commands](../01-cli/cli-commands.md)
