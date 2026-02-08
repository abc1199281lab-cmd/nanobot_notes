# Shell Tool - Shell 執行工具

**檔案路徑**: `nanobot/agent/tools/shell.py`  
**職責**: 安全地執行 shell 命令

---

## 1. 工具: `exec`

執行 shell 命令並回傳輸出。

---

## 2. 初始化參數

```python
def __init__(
    self,
    timeout: int = 60,              # 超時秒數
    working_dir: str | None = None,  # 預設工作目錄
    deny_patterns: list[str],       # 危險模式正則
    allow_patterns: list[str],     # 允許模式（如設定則白名單）
    restrict_to_workspace: bool,   # 是否限制路徑
):
```

---

## 3. 安全機制

### 3.1 預設封鎖模式

```python
deny_patterns = [
    r"\brm\s+-[rf]{1,2}\b",      # rm -r, rm -rf
    r"\bdel\s+/[fq]\b",          # del /f, del /q
    r"\b(format|mkfs|diskpart)\b",  # 磁碟操作
    r"\bdd\s+if=",               # dd
    r">\s*/dev/sd",              # 寫入磁碟
    r"\b(shutdown|reboot|poweroff)\b",
    r":\(\)\s*\{.*\};\s*:",      # fork bomb
]
```

### 3.2 路徑限制

當 `restrict_to_workspace=True`:
- 封鎖 `../` 路徑遍歷
- 驗證所有絕對路徑在工作目錄內

---

## 4. 工具參數

```python
{
    "command": "ls -la",           # 要執行的命令
    "working_dir": "/optional/path"  # 可選工作目錄
}
```

---

## 5. 使用範例

```python
from nanobot.agent.tools.shell import ExecTool

tool = ExecTool(
    timeout=30,
    working_dir="/project",
    restrict_to_workspace=True
)

result = await tool.execute("git status")
```

---

## 6. 相關文件

- [Tools Base](./tools-base.md)
