# Nanobot Repo 理解計劃書

**日期**: 2026-02-08
**作者**: Antigravity Agent

## 1. 目標 (Goal)
針對 `ref/nanobot` 進行完整的程式碼分析，深入理解 Agent 的架構、運作原理、資料流向以及各模組的職責，並為「笨蛋博士」產出簡單好懂的技術文件。

## 2. 執行策略 (Execution Strategy)
採取「由宏觀到微觀 (Top-Down)」與「資料流追蹤 (Data Flow Tracing)」相結合的方式進行分析。

- **Layer 1: 全局架構 (Macro Architecture)**
  - 分析依賴 (`pyproject.toml`) 與入口 (`cli`, `main`)。
  - 理解核心服務啟動流程。
  - 產出：系統架構概觀圖與說明。

- **Layer 2: 核心邏輯 (Core Logic)**
  - **Agent & Brain**: 分析 `agent` 目錄，理解決策迴圈 (Loop)。
  - **Memory & Session**: 分析 `session` 與 Context 管理。
  - **Providers**: 分析 LLM 串接方式 (`providers` via `litellm`)。

- **Layer 3: 輸出入與能力 (IO & Capabilities)**
  - **Channels**: 分析 Telegram, Lark, WebSocket 實作。
  - **Bus**: 理解事件驅動機制。
  - **Skills**: 工具 (Tools) 的註冊與呼叫機制。

## 3. 預計產出文件 (Expected Deliverables)
根據 `AGENTS.md` 規範，產出以下文件至 `doc/draft/`：

1.  **架構總覽 (Architecture Overview)**
    - 系統邊界、核心元件互動。
2.  **核心流程分析 (Core Flows)**
    - 啟動流程 (Startup).
    - 訊息處理流程 (Message Handling: Channel -> Agent -> Reply).
    - Session 生命周期與狀態管理。
3.  **模組深究 (Deep Dive)**
    - `Agent` 決策邏輯。
    - `Provider` 適配層。
    - `Skills` 擴充機制。

## 4. 詳細執行步驟 (Steps)

### Phase 1: 基礎設施與啟動 (Infrastructure & Startup)
- [ ] 分析 `nanobot/cli` 與 `nanobot/__main__.py`。
- [ ] 分析 `nanobot/config` 設定檔結構。
- [ ] 分析 `nanobot/bus` 事件匯流排機制。

### Phase 2: 訊息處理與路由 (Messaging & Routing)
- [ ] 分析 `nanobot/channels` (Telegram/Lark) 如何接收訊息。
- [ ] 追蹤訊息如何進入 `nanobot/agent`。
- [ ] 分析 `nanobot/session` 如何維護對話狀態。

### Phase 3: Agent 智能核心 (Intelligence)
- [ ] 分析 `nanobot/agent` 的核心 Class。
- [ ] 分析 `nanobot/providers` 如何呼叫 LLM (LiteLLM)。
- [ ] 分析 User Prompt 組裝與 Context 處理。

### Phase 4: 技能與擴充 (Skills & Extensions)
- [ ] 分析 `nanobot/skills` 載入機制。
- [ ] 分析 `nanobot/cron` 排程機制。

## 5. Next Step
請確認此計劃是否符合您的需求。若無異議，我將從 **Phase 1: 基礎設施與啟動** 開始進行分析並撰寫文件。
