# MessageBus - 訊息匯流排

**檔案路徑**: `nanobot/bus/queue.py`, `nanobot/bus/events.py`  
**職責**: 非同步訊息佇列，解耦頻道與 Agent

---

## 1. 模組概述

MessageBus 是 nanobot 的核心通訊基礎設施，實現生產者-消費者模式：

- **生產者**: Chat Channels (Telegram, Discord 等)
- **消費者**: Agent Loop

這種設計讓頻道和 Agent 可以獨立運行，通過非同步佇列溝通。

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `queue.py` | `MessageBus` 類別 |
| `events.py` | 訊息資料類別 |

---

## 3. 訊息類型

### 3.1 `InboundMessage` (傳入訊息)

```python
@dataclass
class InboundMessage:
    channel: str           # telegram, discord, cli
    sender_id: str         # 使用者 ID
    chat_id: str           # 聊天 ID
    content: str           # 訊息內容
    timestamp: datetime    # 時間戳
    media: list[str]       # 媒體檔案路徑
    metadata: dict         # 頻道特定資料
```

**屬性**:
- `session_key`: 自動生成 `f"{channel}:{chat_id}"`

### 3.2 `OutboundMessage` (傳出訊息)

```python
@dataclass
class OutboundMessage:
    channel: str
    chat_id: str
    content: str
    reply_to: str | None   # 回覆訊息 ID
    media: list[str]
    metadata: dict
```

---

## 4. 核心類別: `MessageBus`

### 4.1 用途

管理傳入和傳出訊息的佇列，支援訂閱分發。

### 4.2 初始化

```python
def __init__(self):
    self.inbound: asyncio.Queue[InboundMessage]
    self.outbound: asyncio.Queue[OutboundMessage]
    self._outbound_subscribers: dict  # 頻道 → 回呼列表
    self._running = False
```

### 4.3 公開方法

#### 傳入佇列

| 方法 | 說明 |
|------|------|
| `async publish_inbound(msg)` | 發布傳入訊息 |
| `async consume_inbound() -> InboundMessage` | 消費傳入訊息（阻塞） |
| `inbound_size` | 待處理傳入訊息數 |

#### 傳出佇列

| 方法 | 說明 |
|------|------|
| `async publish_outbound(msg)` | 發布傳出訊息 |
| `async consume_outbound() -> OutboundMessage` | 消費傳出訊息 |
| `subscribe_outbound(channel, callback)` | 訂閱特定頻道 |
| `outbound_size` | 待處理傳出訊息數 |

#### 分發

| 方法 | 說明 |
|------|------|
| `async dispatch_outbound()` | 背景分發任務 |
| `stop()` | 停止分發 |

---

## 5. 使用範例

### 5.1 基本流程

```python
bus = MessageBus()

# Channel 發送訊息
await bus.publish_inbound(InboundMessage(
    channel="telegram",
    sender_id="12345",
    chat_id="67890",
    content="Hello!"
))

# Agent 消費
msg = await bus.consume_inbound()

# Agent 回覆
await bus.publish_outbound(OutboundMessage(
    channel=msg.channel,
    chat_id=msg.chat_id,
    content="Hi there!"
))
```

### 5.2 分發模式

```python
# 註冊回呼
async def telegram_handler(msg: OutboundMessage):
    await telegram_bot.send(msg.chat_id, msg.content)

bus.subscribe_outbound("telegram", telegram_handler)

# 啟動分發（背景任務）
asyncio.create_task(bus.dispatch_outbound())
```

---

## 6. 設計考量

### 6.1 解耦

- Channels 和 Agent 完全分離
- 一方崩潰不影響另一方

### 6.2 擴展性

- 支援多個頻道並行
- 訂閱模式允許多個處理器

### 6.3 已知限制

- 無持久化：訊息僅儲存在記憶體
- 無重試機制：失敗的傳出不會自動重試

---

## 7. 相關文件

- [Agent Core](../05-agent/agent-core.md)
- [Channel Manager](../04-channels/channel-manager.md)
