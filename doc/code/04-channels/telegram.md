# TelegramChannel - Telegram 頻道實作

**檔案路徑**: `nanobot/channels/telegram.py`  
**職責**: Telegram Bot API 整合

---

## 1. 模組概述

使用 `python-telegram-bot` 庫實現 Telegram 頻道，支援：
- 長輪詢（polling）模式
- 文字、圖片、語音、檔案訊息
- Markdown 轉 HTML 格式
- 語音轉文字（使用 Groq）

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `telegram.py` | TelegramChannel 類別 |

---

## 3. 核心類別: `TelegramChannel`

### 3.1 初始化

```python
def __init__(
    self,
    config: TelegramConfig,
    bus: MessageBus,
    groq_api_key: str = ""
):
    self.config = config
    self.groq_api_key = groq_api_key
    self._app: Application | None = None
    self._chat_ids: dict[str, int] = {}  # sender_id -> chat_id 映射
```

### 3.2 訊息處理

| 訊息類型 | 處理方式 |
|---------|---------|
| 文字 | 直接轉發 |
| 圖片 | 下載到 media/，添加 [image: path] |
| 語音 | 下載 + Groq 轉錄 + [transcription: text] |
| 檔案 | 下載到 media/，添加 [file: path] |

### 3.3 Markdown 轉 HTML

`_markdown_to_telegram_html()` 函式處理：
- `**bold**` → `<b>bold</b>`
- `_italic_` → `<i>italic</i>`
- `` `code` `` → `<code>code</code>`
- ` ```code``` ` → `<pre><code>code</code></pre>`
- `[text](url)` → `<a href="url">text</a>`

---

## 4. 設定範例

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["12345", "username"],
      "proxy": "http://127.0.0.1:7890"
    }
  }
}
```

---

## 5. 相關文件

- [Base Channel](./base-channel.md)
- [Channel Manager](./channel-manager.md)
