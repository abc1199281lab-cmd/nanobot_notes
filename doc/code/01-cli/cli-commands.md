# CLI Commands - 命令列介面

**檔案路徑**: `nanobot/cli/commands.py`  
**職責**: 實現 nanobot 的所有 CLI 命令

---

## 1. 模組概述

使用 `typer` 庫實現命令列介面，提供：
- 初始設定（onboard）
- 互動式 Agent（agent）
- 閘道伺服器（gateway）
- 頻道管理（channels）
- 排程管理（cron）
- 狀態檢查（status）

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `commands.py` | 所有 CLI 命令實作 |
| `__main__.py` | 入口點 |

---

## 3. 主要命令

### 3.1 `nanobot onboard`

初始化 nanobot 設定和工作目錄。

**建立檔案**:
- `~/.nanobot/config.json`
- `~/.nanobot/workspace/`
- `AGENTS.md`, `SOUL.md`, `USER.md`
- `memory/MEMORY.md`

### 3.2 `nanobot agent`

與 Agent 互動。

**選項**:
- `-m, --message`: 單次訊息模式
- `-s, --session`: 會話 ID
- `--model`: 覆寫預設模型

**範例**:
```bash
nanobot agent -m "你好"
nanobot agent -m "總結這篇論文" --model "gpt-4"
```

### 3.3 `nanobot gateway`

啟動閘道伺服器，處理所有頻道和排程。

**選項**:
- `-p, --port`: 閘道端口（預設 18790）
- `-v, --verbose`: 詳細輸出

### 3.4 `nanobot channels`

頻道管理子命令。

| 子命令 | 說明 |
|--------|------|
| `status` | 顯示頻道狀態 |
| `login` | WhatsApp QR 碼登入 |

### 3.5 `nanobot cron`

排程管理子命令。

| 子命令 | 說明 |
|--------|------|
| `list` | 列出所有任務 |
| `add` | 添加新任務 |
| `remove` | 刪除任務 |
| `enable` | 啟用/停用任務 |
| `run` | 立即執行任務 |

**添加任務範例**:
```bash
# 每 5 分鐘執行
nanobot cron add -n "check" -m "檢查郵件" --every 300

# 每天早上 9 點
nanobot cron add -n "morning" -m "早安問候" --cron "0 9 * * *"
```

### 3.6 `nanobot status`

顯示 nanobot 狀態和設定。

---

## 4. 使用範例

```bash
# 初始化
nanobot onboard

# 編輯設定
nano ~/.nanobot/config.json

# 啟動閘道
nanobot gateway

# 單次查詢
nanobot agent -m "今天日期是什麼？"

# 查看狀態
nanobot status
```

---

## 5. 相關文件

- [Config Schema](../02-config/config-schema.md)
- [Cron Service](../08-cron/cron.md)
