# Utils - 工具函式

**檔案路徑**: `nanobot/utils/helpers.py`  
**職責**: 通用工具函式

---

## 1. 模組概述

提供 nanobot 各模組共用的輔助函式。

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `helpers.py` | 工具函式 |

---

## 3. 函式列表

### 3.1 路徑相關

| 函式 | 說明 |
|------|------|
| `ensure_dir(path) -> Path` | 確保目錄存在 |
| `get_data_path() -> Path` | 取得資料目錄 (~/.nanobot) |
| `get_workspace_path(ws) -> Path` | 取得工作目錄 |
| `get_sessions_path() -> Path` | 取得會話目錄 |
| `get_memory_path(ws) -> Path` | 取得記憶目錄 |
| `get_skills_path(ws) -> Path` | 取得技能目錄 |

### 3.2 字串/時間

| 函式 | 說明 |
|------|------|
| `today_date() -> str` | 今天日期 (YYYY-MM-DD) |
| `timestamp() -> str` | ISO 格式時間戳 |
| `truncate_string(s, max_len, suffix) -> str` | 截斷字串 |
| `safe_filename(name) -> str` | 安全檔名 |

### 3.3 會話

| 函式 | 說明 |
|------|------|
| `parse_session_key(key) -> tuple[str, str]` | 解析會話鍵 |

---

## 4. 使用範例

```python
from nanobot.utils.helpers import ensure_dir, today_date, safe_filename

# 確保目錄存在
memory_dir = ensure_dir(Path.home() / ".nanobot" / "memory")

# 取得今天日期
date = today_date()  # "2026-02-08"

# 安全檔名
safe = safe_filename("hello:world.txt")  # "hello_world.txt"
```

---

## 5. 相關文件

- [所有模組](../README.md)
