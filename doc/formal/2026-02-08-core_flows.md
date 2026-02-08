# Nanobot 核心流程分析 (Core Flows Analysis)

**日期**: 2026-02-08
**版本**: Draft v1.0

## 1. 系統啟動流程 (Startup Flow)

當執行 `nanobot gateway` 時，系統會依序初始化各個服務。

```mermaid
sequenceDiagram
    participant CLI as CLI (commands.py)
    participant Config as Config Loader
    participant Bus as Message Bus
    participant Provider as LiteLLM Provider
    participant Cron as Cron Service
    participant Agent as Agent Loop
    participant ChannelMgr as Channel Manager

    CLI->>Config: load_config()
    Config-->>CLI: Config Object

    CLI->>Bus: Init MessageBus()
    CLI->>Provider: Init LiteLLMProvider()
    
    CLI->>Cron: Init CronService()
    
    CLI->>Agent: Init AgentLoop(bus, provider, cron...)
    activate Agent
    Agent->>Agent: Register Tools (File, Exec, Web...)
    Agent->>Agent: Init SubagentManager
    deactivate Agent

    CLI->>Cron: Set on_job callback (agent.process_direct)
    
    CLI->>ChannelMgr: Init ChannelManager(config, bus)
    activate ChannelMgr
    ChannelMgr->>ChannelMgr: Load enabled channels (Telegram, etc.)
    deactivate ChannelMgr

    CLI->>CLI: asyncio.gather(cron.start(), agent.run(), channels.start_all())
    
    par Async Tasks
        Agent->>Bus: consume_inbound() (Loop)
        ChannelMgr->>Bus: consume_outbound() (Loop)
        ChannelMgr->>Channel: start() (Telegram Polling)
    end
```

## 2. 訊息處理流程 (Message Processing Flow)

當 User 在 Telegram 發送 "Hello" 給機器人時：

```mermaid
sequenceDiagram
    participant User
    participant TG as TelegramChannel
    participant Bus
    participant Agent
    participant Context as ContextBuilder
    participant Session as SessionManager
    participant LLM as Provider (LiteLLM)

    User->>TG: "Hello"
    activate TG
    TG->>TG: _on_message()
    TG->>TG: Download Media (if any)
    TG->>Bus: publish_inbound(InboundMessage)
    deactivate TG

    Bus-->>Agent: InboundMessage
    activate Agent
    
    Agent->>Session: get_or_create(session_key)
    Session-->>Agent: Session Object
    
    Agent->>Context: build_messages(history, "Hello")
    activate Context
    Context->>Context: Load Identity, Bootstrap Files
    Context->>Context: Load Memory & Skills
    Context-->>Agent: Full Prompt (System + History + User)
    deactivate Context

    loop Thought Process (ReAct)
        Agent->>LLM: chat(messages, tools)
        LLM-->>Agent: Response (Content or ToolCall)
        
        opt is Tool Call
            Agent->>Agent: Execute Tool
            Agent->>Context: Add Tool Result
            Agent->>LLM: chat(messages + result)
        end
    end

    Agent->>Session: Save User & Assistant Message
    Agent->>Bus: publish_outbound(OutboundMessage)
    deactivate Agent

    Bus-->>TG: OutboundMessage (via ChannelManager)
    activate TG
    TG->>TG: Convert Markdown to HTML
    TG->>User: Send Reply
    deactivate TG
```

## 3. Context 建構流程 (Context Build Flow)

Agent 如何「記得」你是誰，以及它能做什麼？全靠 `ContextBuilder`。

1.  **Identity**: 載入硬編碼的 System Prompt（你是 nanobot...）。
2.  **Bootstrap**: 讀取 `AGENTS.md`, `SOUL.md`, `USER.md` 等核心設定檔。
3.  **Memory**: 讀取 `memory/MEMORY.md` (長期記憶)。
4.  **Skills**:
    - **Always-on Skills**: 直接讀取完整 `SKILL.md` 內容放入 Context。
    - **Other Skills**: 僅放入 XML 摘要 (`<skill><name>...</name></skill>`)。Agent 若需使用，需先呼叫 `read_file` 讀取詳細說明。
5.  **History**: 從 Session 讀取最近 N 則對話紀錄。
6.  **Current Message**: 加上 User 最新的一句話 (含圖片 Base64)。

## 4. 技能使用流程 (Skill Execution Flow)

Nanobot 的技能設計是「文件中定義工具」。

1.  User: "幫我查一下現在的比特幣價格"
2.  Agent:
    - 檢視 Context 中的 `<skills>` 列表，發現 `finance` skill 可能有幫助。
    - (若未載入) Agent 決定讀取 `skills/finance/SKILL.md`。
    - 讀取後，Agent 發現裡面描述了如何使用 `web_search` 工具來查詢價格。
    - Agent 呼叫 `web_search(query="bitcoin price")`。
    - 獲得結果後，Agent 整理並回覆 User。
