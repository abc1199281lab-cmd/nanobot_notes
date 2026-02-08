# Web Tools - 網頁工具

**檔案路徑**: `nanobot/agent/tools/web.py`  
**職責**: 網頁搜尋和內容擷取

---

## 1. 工具列表

| 工具 | 類別 | 說明 |
|------|------|------|
| `web_search` | `WebSearchTool` | 網頁搜尋（Brave Search API） |
| `web_fetch` | `WebFetchTool` | 擷取網頁內容 |

---

## 2. `web_search`

使用 Brave Search API 搜尋網頁。

### 2.1 參數

```python
{
    "query": "search terms",    # 搜尋關鍵字
    "count": 5                  # 結果數量 (1-10，預設 5)
}
```

### 2.2 回傳格式

```
Results for: search terms

1. Title
   https://url.com
   Description snippet

2. Title
   ...
```

### 2.3 API 金鑰

需要 `BRAVE_API_KEY` 環境變數或在設定中設定。

---

## 3. `web_fetch`

擷取 URL 內容並轉換為可讀格式。

### 3.1 參數

```python
{
    "url": "https://example.com",      # 目標 URL
    "extractMode": "markdown",          # "markdown" 或 "text"
    "maxChars": 50000                   # 最大字元數
}
```

### 3.2 回傳格式 (JSON)

```json
{
  "url": "原始 URL",
  "finalUrl": "最終 URL（重定向後）",
  "status": 200,
  "extractor": "readability|json|raw",
  "truncated": false,
  "length": 1234,
  "text": "擷取的內容"
}
```

### 3.3 處理流程

1. 驗證 URL（必須 http/https）
2. 下載內容（最多 5 次重定向）
3. 根據 Content-Type 處理：
   - `application/json` → 格式化 JSON
   - `text/html` → Readability 提取正文
   - 其他 → 原始文字
4. 轉換為 Markdown（可選）

---

## 4. 安全機制

- URL 驗證（僅允許 http/https）
- 重定向限制（5 次）
- 超時控制（10-30 秒）

---

## 5. 相關文件

- [Tools Base](./tools-base.md)
- [Config Schema](../../02-config/config-schema.md)
