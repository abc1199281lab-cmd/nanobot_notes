# SessionManager - 會話管理

**檔案路徑**: `nanobot/session/manager.py`  
**職責**: 管理對話歷史和會話狀態

---

## 1. 模組概述

SessionManager 負責：
- 儲存和載入對話歷史
- 會話快取（記憶體 + 磁碟）
- 會話識別（`channel:chat_id`）

儲存格式：JSONL（每行一條訊息）
儲存位置：`~/.nanobot/sessions/{channel}_{chat_id}.jsonl`

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `manager.py` | Session 和 SessionManager 類別 |

---

## 3. 核心類別

### 3.1 `Session` (會話資料)

```python
@dataclass
class Session:
    key: str                      # 會話鍵 (channel:chat_id)
    messages: list[dict]          # 訊息列表
    created_at: datetime
    updated_at: datetime
    metadata: dict                # 額外資料
```

**方法**:
- `add_message(role, content, **kwargs)`: 添加訊息
- `get_history(max_messages=50) -> list[dict]`: 取得歷史（LLM 格式）
- `clear()`: 清空會話

### 3.2 `SessionManager` (會話管理)

#### 初始化

```python
def __init__(self, workspace: Path):
    self.sessions_dir = Path.home() / ".nanobot" / "sessions"
    self._cache: dict[str, Session] = {}  # 記憶體快取
```

#### 公開方法

| 方法 | 說明 |
|------|------|
| `get_or_create(key: str) -> Session` | 獲取或創建會話 |
| `save(session: Session)` | 儲存會話到磁碟 |
| `delete(key: str) -> bool` | 刪除會話 |
| `list_sessions() -> list[dict]` | 列出所有會話 |

---

## 4. 儲存格式 (JSONL)

```jsonl
{"_type": "metadata", "created_at": "2026-02-08T10:00:00", ...}
{"role": "user", "content": "Hello", "timestamp": "..."}
{"role": "assistant", "content": "Hi!", "timestamp": "..."}
```

---

## 5. 使用範例

```python
from nanobot.session.manager import SessionManager

sessions = SessionManager(Path.home() / ".nanobot" / "workspace")

# 獲取會話
session = sessions.get_or_create("telegram:12345")

# 添加訊息
session.add_message("user", "What's the weather?")
session.add_message("assistant", "It's sunny!")

# 儲存
sessions.save(session)

# 取得歷史（給 LLM）
history = session.get_history(max_messages=10)
```

---

## 6. 設計考量

### 6.1 快取策略

- 記憶體快取常用會話
- 懶加載：首次使用時從磁碟讀取
- 顯式儲存：呼叫 `save()` 才寫入磁碟

### 6.2 檔案命名

- 使用 `safe_filename()` 轉換特殊字元
- `:` 轉為 `_`

### 6.3 已知限制

- 無加密：會話檔案為明文 JSONL
- 無壓縮：大會話佔用較多磁碟空間
- 單機限制：無法跨機器同步

---

## 7. 相關文件

- [Agent Core](../05-agent/agent-core.md)
