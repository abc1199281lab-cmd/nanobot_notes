# Spawn Tool - 子代理生成工具

**檔案路徑**: `nanobot/agent/tools/spawn.py`  
**職責**: 創建背景執行的子代理

---

## 1. 工具: `spawn`

創建子代理在背景執行複雜任務。

---

## 2. 參數

```python
{
    "task": "任務描述",            # 子代理要執行的任務
    "label": "可選標籤"            # 顯示用的簡短標籤
}
```

---

## 3. 回傳

```
Subagent [研究任務] started (id: a1b2c3d4). 
I'll notify you when it completes.
```

---

## 4. 工作原理

1. Agent 呼叫 `spawn`
2. 創建背景 `asyncio.Task`
3. 子代理獨立執行（無法發送訊息或創建子代理）
4. 完成後通過系統訊息通知主 Agent
5. 主 Agent 總結結果給使用者

---

## 5. 使用場景

- 長時間網頁研究
- 大量檔案處理
- 需要並行執行的獨立任務

---

## 6. 相關文件

- [Tools Base](./tools-base.md)
- [Agent Subagent](../../agent-subagent.md)
