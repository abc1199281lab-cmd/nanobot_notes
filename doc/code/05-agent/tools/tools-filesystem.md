# Filesystem Tools - 檔案系統工具

**檔案路徑**: `nanobot/agent/tools/filesystem.py`  
**職責**: 檔案讀寫、編輯、目錄列出

---

## 1. 工具列表

| 工具 | 類別 | 說明 |
|------|------|------|
| `read_file` | `ReadFileTool` | 讀取檔案 |
| `write_file` | `WriteFileTool` | 寫入檔案 |
| `edit_file` | `EditFileTool` | 編輯檔案（文字替換） |
| `list_dir` | `ListDirTool` | 列出目錄內容 |

---

## 2. 目錄限制

所有工具支援 `allowed_dir` 參數：
- `None`: 允許任何路徑
- `Path`: 限制只能在該目錄下操作

---

## 3. 工具詳細

### 3.1 `read_file`

```python
{
    "path": "/path/to/file"  # 檔案路徑
}
```

回傳: 檔案內容或錯誤訊息

### 3.2 `write_file`

```python
{
    "path": "/path/to/file",
    "content": "file content"
}
```

回傳: 成功訊息含位元組數

### 3.3 `edit_file`

```python
{
    "path": "/path/to/file",
    "old_text": "text to find",
    "new_text": "replacement text"
}
```

**限制**:
- `old_text` 必須完全匹配
- 如出現多次，會警告要求更多上下文

### 3.4 `list_dir`

```python
{
    "path": "/path/to/dir"
}
```

回傳: 格式化的目錄列表（📁 資料夾, 📄 檔案）

---

## 4. 使用範例

```python
from nanobot.agent.tools.filesystem import ReadFileTool, WriteFileTool

# 讀取
read_tool = ReadFileTool(allowed_dir=Path("/safe/path"))
content = await read_tool.execute("/safe/path/file.txt")

# 寫入
write_tool = WriteFileTool()
result = await write_tool.execute("/tmp/output.txt", "Hello World")
```

---

## 5. 相關文件

- [Tools Base](./tools-base.md)
