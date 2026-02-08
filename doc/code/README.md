# nanobot 程式碼文件索引

這是 nanobot 原始碼的詳細中文文件集合。

---

## 文件結構

```
doc/draft/code/
├── 01-cli/
│   └── cli-commands.md          # CLI 命令
├── 02-config/
│   └── config-schema.md         # 設定結構
├── 03-bus/
│   └── bus.md                   # 訊息匯流排
├── 04-channels/
│   ├── base-channel.md          # 頻道基礎類別
│   ├── channel-manager.md       # 頻道管理器
│   └── telegram.md              # Telegram 頻道
├── 05-agent/                    # 核心 Agent 模組
│   ├── agent-core.md            # Agent Loop 核心
│   ├── agent-context.md         # 上下文建構器
│   ├── agent-memory.md          # 記憶系統
│   ├── agent-skills.md          # 技能載入器
│   ├── agent-subagent.md        # 子代理管理器
│   └── tools/                   # 工具實作
│       ├── tools-base.md        # 工具基礎類別
│       ├── tools-registry.md    # 工具註冊表
│       ├── tools-filesystem.md  # 檔案系統工具
│       ├── tools-shell.md       # Shell 執行工具
│       ├── tools-web.md         # 網頁工具
│       ├── tools-message.md     # 訊息發送工具
│       ├── tools-spawn.md       # 子代理生成工具
│       └── tools-cron.md        # 排程工具
├── 06-providers/
│   └── llm-provider.md          # LLM 提供者
├── 07-session/
│   └── session.md               # 會話管理
├── 08-cron/
│   └── cron.md                  # 排程服務
├── 09-heartbeat/
│   └── heartbeat.md             # 心跳服務
└── 10-utils/
    └── utils.md                 # 工具函式
```

---

## Phase 1: 核心流程 (Core Flow)

這些模組構成 nanobot 的核心處理引擎：

| 文件 | 原始碼 | 職責 |
|------|--------|------|
| [agent-core.md](./05-agent/agent-core.md) | `agent/loop.py` | Agent 主循環 |
| [agent-context.md](./05-agent/agent-context.md) | `agent/context.py` | 上下文建構 |
| [bus.md](./03-bus/bus.md) | `bus/*.py` | 訊息匯流排 |
| [session.md](./07-session/session.md) | `session/manager.py` | 會話管理 |

---

## Phase 2: 輸入輸出層 (I/O Layer)

處理與外部世界的通訊：

| 文件 | 原始碼 | 職責 |
|------|--------|------|
| [base-channel.md](./04-channels/base-channel.md) | `channels/base.py` | 頻道介面 |
| [channel-manager.md](./04-channels/channel-manager.md) | `channels/manager.py` | 頻道管理 |
| [telegram.md](./04-channels/telegram.md) | `channels/telegram.py` | Telegram 實作 |
| [llm-provider.md](./06-providers/llm-provider.md) | `providers/*.py` | LLM 介面 |

---

## Phase 3: 工具與擴充 (Tools & Extensions)

提供 Agent 能力的工具：

| 文件 | 原始碼 | 職責 |
|------|--------|------|
| [tools-base.md](./05-agent/tools/tools-base.md) | `agent/tools/base.py` | 工具基礎 |
| [tools-registry.md](./05-agent/tools/tools-registry.md) | `agent/tools/registry.py` | 工具註冊 |
| [tools-filesystem.md](./05-agent/tools/tools-filesystem.md) | `agent/tools/filesystem.py` | 檔案操作 |
| [tools-shell.md](./05-agent/tools/tools-shell.md) | `agent/tools/shell.py` | Shell 執行 |
| [tools-web.md](./05-agent/tools/tools-web.md) | `agent/tools/web.py` | 網頁搜尋 |
| [tools-message.md](./05-agent/tools/tools-message.md) | `agent/tools/message.py` | 訊息發送 |
| [tools-spawn.md](./05-agent/tools/tools-spawn.md) | `agent/tools/spawn.py` | 子代理生成 |
| [tools-cron.md](./05-agent/tools/tools-cron.md) | `agent/tools/cron.py` | 排程操作 |
| [agent-memory.md](./05-agent/agent-memory.md) | `agent/memory.py` | 記憶系統 |
| [agent-skills.md](./05-agent/agent-skills.md) | `agent/skills.py` | 技能系統 |
| [agent-subagent.md](./05-agent/agent-subagent.md) | `agent/subagent.py` | 子代理管理 |
| [cron.md](./08-cron/cron.md) | `cron/*.py` | 排程服務 |

---

## Phase 4: 基礎設施 (Infrastructure)

支援整個系統的基礎模組：

| 文件 | 原始碼 | 職責 |
|------|--------|------|
| [cli-commands.md](./01-cli/cli-commands.md) | `cli/commands.py` | CLI 命令 |
| [config-schema.md](./02-config/config-schema.md) | `config/*.py` | 設定系統 |
| [heartbeat.md](./09-heartbeat/heartbeat.md) | `heartbeat/service.py` | 心跳服務 |
| [utils.md](./10-utils/utils.md) | `utils/helpers.py` | 工具函式 |

---

## 快速參考

### 核心資料流

```
User Message
    ↓
Channel (Telegram/Discord/...)
    ↓
MessageBus.publish_inbound()
    ↓
AgentLoop.consume_inbound()
    ↓
AgentLoop._process_message()
    ↓ (循環)
LLMProvider.chat()
    ↓
ToolRegistry.execute()
    ↓
MessageBus.publish_outbound()
    ↓
Channel.send()
    ↓
User Response
```

### 工具類別對照

| 工具名稱 | 用途 |
|---------|------|
| `read_file` | 讀取檔案 |
| `write_file` | 寫入檔案 |
| `edit_file` | 編輯檔案 |
| `list_dir` | 列出目錄 |
| `exec` | 執行 shell |
| `web_search` | 網頁搜尋 |
| `web_fetch` | 擷取網頁 |
| `message` | 發送訊息 |
| `spawn` | 生成子代理 |
| `cron` | 管理排程 |

---

## 貢獻

這些文件根據 nanobot 原始碼自動生成，隨著程式碼更新而更新。
