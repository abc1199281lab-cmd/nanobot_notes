# SkillsLoader - 技能載入器

**檔案路徑**: `nanobot/agent/skills.py`  
**職責**: 載入和管理 Agent 的技能 (SKILL.md)

---

## 1. 模組概述

SkillsLoader 管理 nanobot 的技能系統。技能是 Markdown 檔案，教導 Agent 如何使用特定工具或執行特定任務。

**技能位置**:
- 工作目錄: `~/.nanobot/workspace/skills/{skill-name}/SKILL.md`
- 內建: `nanobot/skills/{skill-name}/SKILL.md`

---

## 2. 檔案結構

| 檔案 | 說明 |
|------|------|
| `skills.py` | SkillsLoader 類別 |

---

## 3. 核心類別: `SkillsLoader`

### 3.1 用途

發現、載入和管理技能，支援依賴檢查和漸進式載入。

### 3.2 初始化 (`__init__`)

```python
def __init__(self, workspace: Path, builtin_skills_dir: Path | None = None):
    self.workspace = workspace
    self.workspace_skills = workspace / "skills"           # 使用者技能
    self.builtin_skills = builtin_skills_dir or BUILTIN_SKILLS_DIR  # 內建技能
```

### 3.3 公開方法

#### `list_skills(filter_unavailable: bool = True) -> list[dict]`

列出所有可用技能。

**回傳**:
```python
[
    {"name": "github", "path": "/path/to/SKILL.md", "source": "workspace"},
    {"name": "weather", "path": "/path/to/SKILL.md", "source": "builtin"},
]
```

#### `load_skill(name: str) -> str | None`

載入指定技能內容。

**搜尋順序**: 工作目錄 → 內建

#### `load_skills_for_context(skill_names: list[str]) -> str`

載入指定技能供上下文使用。

**行為**:
- 移除 YAML frontmatter
- 格式化為 Agent 可讀格式

#### `build_skills_summary() -> str`

建構技能摘要（XML 格式）。

**範例輸出**:
```xml
<skills>
  <skill available="true">
    <name>github</name>
    <description>GitHub CLI 技能</description>
    <location>/path/to/SKILL.md</location>
  </skill>
  <skill available="false">
    <name>tmux</name>
    <description>Tmux 遠端控制</description>
    <location>/path/to/SKILL.md</location>
    <requires>CLI: tmux</requires>
  </skill>
</skills>
```

#### `get_always_skills() -> list[str]`

取得標記為 `always=true` 的技能。

#### `get_skill_metadata(name: str) -> dict | None`

解析技能的 YAML frontmatter。

---

## 4. SKILL.md 格式

```yaml
---
name: skill-name
description: Short description of the skill
always: false  # Whether to always load
metadata: |
  {"nanobot": {"requires": {"bins": ["gh"], "env": ["GITHUB_TOKEN"]}}}
---

# Skill Content

詳細的技能說明...
```

---

## 5. 設計考量

### 5.1 漸進式載入

- **摘要優先**: 僅傳技能列表給 LLM
- **按需讀取**: Agent 用 `read_file` 讀取詳細內容
- **降低 Token**: 避免上下文膨脹

### 5.2 依賴檢查

- **執行檔**: 檢查 `PATH` 中的命令
- **環境變數**: 檢查必要環境變數
- **標記 unavailable**: 未滿足依賴的技能標記為不可用

### 5.3 覆蓋機制

- 使用者技能優先於內建技能
- 允許使用者自定義和擴展

---

## 6. 相關文件

- [Agent Context](./agent-context.md) - 技能整合到上下文
- [Agent Core](./agent-core.md) - Agent 主循環
