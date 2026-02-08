# Nanobot 模組深究 (Module Deep Dive)

**日期**: 2026-02-08
**版本**: Draft v1.0

## 1. Agent 核心 (Agent Loop)

`nanobot/agent/loop.py` 是整個系統的心臟。
- **職責**: 接收訊息 -> 思考 -> 行動 -> 回覆。
- **關鍵類別**: `AgentLoop`
- **主要流程 (`run`)**:
  - `while self._running`:
    - `msg = await bus.consume_inbound()`: 等待訊息。
    - `self._process_message(msg)`: 處理訊息。
      - 取得 Session。
      - 設定 Context (工具上下文)。
      - `ContextBuilder.build_messages()`: 建立 Prompt。
      - **LLM Loop (`max_iterations`)**:
        - `response = await provider.chat(...)`
        - 若有 `tool_calls` -> 執行 Tool -> 將結果加入 Context -> 繼續 Loop。
        - 若無 `tool_calls` -> 視為最終回應 -> 跳出 Loop。
      - 更新 Session (儲存對話)。
      - `bus.publish_outbound(response)`: 發送回覆。

## 2. 供應者整合 (Provider Integration)

`nanobot/providers/litellm_provider.py` 負責與 LLM 溝通。
- **特點**: 使用 `LiteLLM` 函式庫，支援多種 Provider (OpenAI, Anthropic, Ollama 等)。
- **自動前綴 (Auto-prefix)**: 根據 model name 自動加上 provider prefix (例如 `gpt-4` -> `openai/gpt-4`)，簡化 Config 設定。
- **OpenRouter 支援**: 若偵測到 API Key 為 `sk-or-` 開頭，自動設定為 OpenRouter 模式。
- **本地模型**: 支援 `ollama` 與 `vllm` 作為本地後端。

## 3. 技能載入 (Skills Loader)

`nanobot/agent/skills.py` 負責管理 Agent 的額外能力。
- **技能定義**: 每個 Skill 是一個目錄，內含 `SKILL.md`。透過 Markdown 文件定義技能的使用方式與範例。
- **動態載入**:
  - `list_skills()`: 列出所有可用技能。
  - `build_skills_summary()`: 產生 XML 格式的技能摘要清單，放入 System Prompt 供 Agent 參考。
  - `load_skill(name)`: 讀取完整 `SKILL.md` 內容。
- **依賴檢查**: 檢查 Skill 是否需要特定的 CLI 工具 (如 `git`, `ffmpeg`) 或環境變數。

## 4. Session 管理 (Session Manager)

`nanobot/session/manager.py` 負責對話狀態的持久化。
- **儲存格式**: JSON Lines (`.jsonl`)。每一行是一個 JSON 物件 (Message)。
- **Metadata**: 檔案第一行通常是 Metadata (建立時間、標籤等)。
- **Cache**: 簡單的記憶體快取，避免頻繁讀取同一 Session。

## 5. 資料結構 (Data Structures)

- **InboundMessage**: 
  - `channel`: 來源通道 (telegram, cli)。
  - `sender_id`: 發送者 ID。
  - `chat_id`: 聊天室 ID。
  - `content`: 文字內容。
  - `media`: 媒體檔案路徑列表。

- **OutboundMessage**:
  - `channel`: 目標通道。
  - `chat_id`: 目標聊天室 ID。
  - `content`: 回覆內容 (支援 Markdown)。
