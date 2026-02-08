# ChannelManager - 頻道管理器

**檔案路徑**: `nanobot/channels/manager.py`  
**職責**: 協調多個聊天頻道的生命周期和訊息路由

---

## 1. 模組概述

ChannelManager 負責：
- 根據設定初始化啟用的頻道
- 統一啟動/停止所有頻道
- 路由傳出訊息到正確的頻道

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `manager.py` | ChannelManager 類別 |

---

## 3. 核心類別: `ChannelManager`

### 3.1 初始化

```python
def __init__(self, config: Config, bus: MessageBus):
    self.config = config
    self.bus = bus
    self.channels: dict[str, BaseChannel] = {}
    self._init_channels()  # 根據設定初始化
```

**支援的頻道**:
- Telegram
- WhatsApp
- Discord
- Feishu (飛書)

### 3.2 公開方法

| 方法 | 說明 |
|------|------|
| `async start_all()` | 啟動所有頻道和分發器 |
| `async stop_all()` | 停止所有頻道 |
| `get_channel(name) -> BaseChannel \| None` | 獲取指定頻道 |
| `get_status() -> dict` | 獲取所有頻道狀態 |
| `enabled_channels -> list[str]` | 啟用的頻道名稱列表 |

### 3.3 訊息路由

```python
async def _dispatch_outbound(self):
    while True:
        msg = await self.bus.consume_outbound()
        channel = self.channels.get(msg.channel)
        if channel:
            await channel.send(msg)
```

---

## 4. 使用範例

```python
from nanobot.channels.manager import ChannelManager
from nanobot.config.loader import load_config
from nanobot.bus.queue import MessageBus

config = load_config()
bus = MessageBus()

manager = ChannelManager(config, bus)

# 啟動所有頻道（阻塞）
await manager.start_all()
```

---

## 5. 相關文件

- [Base Channel](./base-channel.md)
- [Telegram Channel](./telegram.md)
