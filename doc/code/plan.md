Nanobot 詳細程式碼文件規劃 (Detailed Code Documentation Plan)
日期: 2026-02-08 目標: 為 
nanobot
 的每個模組、檔案、類別與函式建立詳細的中文說明文件

1. 文件化策略 (Documentation Strategy)
1.1 分層文件結構
我們將採用「由上而下」的文件結構：
```
doc/draft/code/
├── 01-cli/                    # CLI 與入口點
├── 02-config/                 # 設定管理
├── 03-bus/                    # 訊息匯流排
├── 04-channels/               # 通訊層
├── 05-agent/                  # 智能核心
│   ├── agent-core.md          # Agent Loop 核心
│   ├── agent-context.md       # Context Builder
│   ├── agent-skills.md        # Skills Loader
│   ├── agent-memory.md        # Memory Store
│   ├── agent-subagent.md      # Subagent Manager
│   └── tools/                 # 工具實作
│       ├── tools-filesystem.md
│       ├── tools-shell.md
│       ├── tools-web.md
│       └── tools-other.md
├── 06-providers/              # LLM 供應者
├── 07-session/                # 對話管理
├── 08-cron/                   # 排程服務
├── 09-heartbeat/              # 心跳服務
└── 10-utils/                  # 工具函式
```

1.2 每個文件的內容結構
每個 .md 文件將包含：

模組概述: 該模組的職責與設計目的
檔案清單: 列出該模組下的所有檔案
類別與函式說明:
類別名稱與用途
初始化參數 (
init
)
公開方法 (Public Methods)
重要私有方法 (Private Methods)
資料結構 (Dataclasses/Models)
使用範例: 簡單的程式碼範例
注意事項: 特殊設計考量或陷阱
2. 模組優先順序 (Priority Order)
根據重要性與複雜度，我們將按以下順序撰寫文件：

Phase 1: 核心流程 (Core Flow) - 最高優先
05-agent/agent-core.md - 
agent/loop.py
 (AgentLoop)
05-agent/agent-context.md - 
agent/context.py
 (ContextBuilder)
03-bus/ - 
bus/queue.py
, bus/events.py
07-session/ - 
session/manager.py
Phase 2: 輸入輸出層 (I/O Layer)
04-channels/ - 
channels/base.py
, 
channels/telegram.py
, 
channels/manager.py
06-providers/ - 
providers/base.py
, 
providers/litellm_provider.py
Phase 3: 工具與擴充 (Tools & Extensions)
05-agent/tools/ - 所有 Tool 實作
05-agent/agent-skills.md - 
agent/skills.py
08-cron/ - cron/service.py, cron/types.py
Phase 4: 基礎設施 (Infrastructure)
01-cli/ - 
cli/commands.py
02-config/ - 
config/schema.py
, 
config/loader.py
10-utils/ - utils/helpers.py
3. 文件範本 (Documentation Template)
每個文件將遵循以下範本：

```markdown

[模組名稱] - [簡短描述]
檔案路徑: `nanobot/[path]` 職責: [一句話說明這個模組的核心職責]

1. 模組概述
[詳細說明這個模組在整體架構中的角色]

2. 檔案結構
`file1.py`: [說明]
`file2.py`: [說明]
3. 核心類別與函式
3.1 ClassName
用途: [類別的用途]

初始化 (`init`)
```python def init(self, param1: Type1, param2: Type2): ... ```

參數:

`param1`: [說明]
`param2`: [說明]
方法: `method_name`
```python def method_name(self, arg: Type) -> ReturnType: ... ```

用途: [方法的用途] 參數: [參數說明] 回傳: [回傳值說明]

4. 使用範例
```python

範例程式碼
```

5. 設計考量
[重要的設計決策]
[已知限制或陷阱] ```
4. 執行計劃 (Execution Plan)
Step 1: 建立目錄結構
在 `doc/draft/code/` 下建立所有子目錄。

Step 2: 撰寫 Phase 1 文件 (核心流程)
優先完成最核心的 4 個文件：

`05-agent/agent-core.md`
`05-agent/agent-context.md`
`03-bus/bus.md`
`07-session/session.md`
Step 3: 依序完成 Phase 2-4
按照優先順序逐步完成其他模組的文件。

Step 4: 建立索引文件
在 `doc/draft/code/README.md` 建立總索引，列出所有文件的連結與簡介。

5. 預期產出 (Expected Deliverables)
總文件數: 約 15-20 個 Markdown 文件
總字數: 預估 20,000-30,000 字
涵蓋範圍:
所有核心類別 (約 30+ 個)
所有公開方法 (約 150+ 個)
重要私有方法 (約 50+ 個)
6. Next Steps
請確認此規劃是否符合您的需求。若無異議，我將開始執行：

建立目錄結構
撰寫 Phase 1 的 4 個核心文件