# BaseChannel - 頻道基礎類別

**檔案路徑**: `nanobot/channels/base.py`  
**職責**: 定義所有通訊頻道的通用介面

---

## 1. 模組概述

`BaseChannel` 是抽象基礎類別，定義所有聊天頻道必須實現的介面。具體頻道（Telegram、Discord 等）繼承此類並實現平台特定邏輯。

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `base.py` | BaseChannel 抽象類別 |

---

## 3. 核心類別: `BaseChannel`

### 3.1 抽象方法

| 方法 | 說明 |
|------|------|
| `async start()` | 啟動頻道，開始監聽訊息 |
| `async stop()` | 停止頻道，清理資源 |
| `async send(msg: OutboundMessage)` | 發送訊息到平台 |

### 3.2 通用方法

| 方法 | 說明 |
|------|------|
| `is_allowed(sender_id) -> bool` | 檢查發送者是否在白名單 |
| `async _handle_message(...)` | 處理傳入訊息，轉發到 MessageBus |

### 3.3 權限控制

通過 `config.allow_from` 設定白名單：
- 空列表 = 允許所有人
- 支援複合 ID: `"12345\|username"`

---

## 4. 實作範例

```python
class MyChannel(BaseChannel):
    name = "myplatform"
    
    async def start(self):
        # 連接到平台 API
        # 監聽訊息，呼叫 self._handle_message()
        pass
    
    async def stop(self):
        # 斷開連接
        pass
    
    async def send(self, msg):
        # 發送訊息到平台
        pass
```

---

## 5. 相關文件

- [Telegram Channel](./telegram.md)
- [Channel Manager](./channel-manager.md)
