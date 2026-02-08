# CronService - 排程服務

**檔案路徑**: `nanobot/cron/service.py`, `nanobot/cron/types.py`  
**職責**: 管理和執行定時任務

---

## 1. 模組概述

CronService 提供類似 Unix cron 的排程功能，支援：
- 一次性任務（`at`）
- 週期性任務（`every N seconds`）
- Cron 表達式（`0 9 * * *`）

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `types.py` | 資料類別（CronJob, CronSchedule, ...） |
| `service.py` | CronService 主類別 |

---

## 3. 資料類別

### 3.1 `CronSchedule`

```python
@dataclass
class CronSchedule:
    kind: Literal["at", "every", "cron"]
    at_ms: int | None           # at: 時間戳 (ms)
    every_ms: int | None        # every: 間隔 (ms)
    expr: str | None            # cron: 表達式
    tz: str | None              # cron: 時區
```

### 3.2 `CronPayload`

```python
@dataclass
class CronPayload:
    kind: Literal["agent_turn"]
    message: str                # 發給 Agent 的訊息
    deliver: bool               # 是否傳送到頻道
    channel: str | None
    to: str | None              # 接收者 ID
```

### 3.3 `CronJob`

```python
@dataclass
class CronJob:
    id: str
    name: str
    enabled: bool
    schedule: CronSchedule
    payload: CronPayload
    state: CronJobState         # 執行狀態
    created_at_ms: int
    updated_at_ms: int
    delete_after_run: bool      # 一次性任務後刪除
```

---

## 4. 核心類別: `CronService`

### 4.1 初始化

```python
def __init__(
    self,
    store_path: Path,
    on_job: Callable[[CronJob], Coroutine[str | None]] | None = None
):
    # store_path: 儲存路徑 (~/.nanobot/cron/jobs.json)
    # on_job: 任務執行回呼（由 Agent 設定）
```

### 4.2 公開方法

| 方法 | 說明 |
|------|------|
| `async start()` | 啟動排程器 |
| `stop()` | 停止排程器 |
| `list_jobs() -> list[CronJob]` | 列出所有任務 |
| `add_job(name, schedule, message, ...) -> CronJob` | 添加任務 |
| `remove_job(job_id) -> bool` | 刪除任務 |
| `enable_job(job_id, enabled) -> CronJob \| None` | 啟用/停用 |
| `async run_job(job_id, force) -> bool` | 立即執行 |
| `status() -> dict` | 服務狀態 |

---

## 5. 使用範例

### 5.1 添加週期性任務

```python
from nanobot.cron.service import CronService
from nanobot.cron.types import CronSchedule

service = CronService(Path.home() / ".nanobot" / "cron" / "jobs.json")

# 每小時檢查郵件
job = service.add_job(
    name="check-email",
    schedule=CronSchedule(kind="every", every_ms=3600 * 1000),
    message="檢查郵件並總結重要內容",
    deliver=True,
    channel="telegram",
    to="12345"
)
```

### 5.2 添加 Cron 表達式任務

```python
# 每天早上 9 點
job = service.add_job(
    name="morning-brief",
    schedule=CronSchedule(kind="cron", expr="0 9 * * *"),
    message="生成今日摘要",
    deliver=True,
    channel="telegram",
    to="12345"
)
```

---

## 6. 相關文件

- [Cron Tool](../05-agent/tools/tools-cron.md)
- [CLI Commands](../01-cli/cli-commands.md)
