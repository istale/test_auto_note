
# Collaborative Development Ledger – Reference
This document defines the internal data model, action schemas,
and storage conventions used by the collaborative-development-ledger skill.

This file is intended for Copilot and Agents when performing
ledger read/write or reasoning replay operations.

## Conceptual Model (Detailed)
A Discussion Entry represents a single discussion event and includes:

- topic
- summary (neutral, non-decisional)
- multiple positions with rationale
- lifecycle status
- links to related entries
- historical notes

Entries are append-only and evolve only through status changes or linkage.

## Action Schemas

### append_discussion
**Purpose**  
Record a new discussion entry representing exploratory or unresolved reasoning.

**Notes**
- `summary` must be factual and perspective-inclusive  
- It must not contain decisions, recommendations, or trade-off conclusions

**input_schema**
```json
{
  "type": "object",
  "properties": {
    "topic": {
      "type": "string",
      "description": "討論主題標題"
    },
    "summary": {
      "type": "string",
      "description": "對討論內容的中立摘要"
    },
    "positions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "by": {
            "type": "string",
            "description": "立場提出者（user / assistant / agent 名稱）"
          },
          "stance": {
            "type": "string",
            "description": "立場簡述"
          },
          "rationale": {
            "type": "string",
            "description": "該立場的理由或假設"
          }
        },
        "required": ["by", "stance", "rationale"]
      }
    },
    "status": {
      "type": "string",
      "enum": ["exploring", "tentative", "unresolved"],
      "description": "目前討論狀態"
    },
    "related_entries": {
      "type": "array",
      "items": { "type": "string" },
      "description": "相關既有 Ledger entry ID"
    }
  },
  "required": ["topic", "summary", "positions", "status"]
}
```

**output_schema**
```json
{
  "type": "object",
  "properties": {
    "entry_id": {
      "type": "string",
      "description": "新建立的 Ledger entry ID"
    }
  }
}
```

---

### link_to_prior
**Purpose**  
Create a semantic relationship between the current discussion and prior entries.

**input_schema**
```json
{
  "type": "object",
  "properties": {
    "current_entry_id": {
      "type": "string"
    },
    "link_to": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "entry_id": { "type": "string" },
          "relation": {
            "type": "string",
            "description": "關聯描述（延伸、衝突、補充等）"
          }
        },
        "required": ["entry_id", "relation"]
      }
    }
  },
  "required": ["current_entry_id", "link_to"]
}
```

---

### mark_state

**Purpose**
Update the lifecycle state of a discussion entry without rewriting history.

**Allowed States**
- exploring：探索中
- tentative：暫定
- accepted：已採納
- deprecated：已淘汰
- revived：重新啟用

**input_schema**
```json
{
  "type": "object",
  "properties": {
    "entry_id": { "type": "string" },
    "new_status": {
      "type": "string",
      "enum": ["exploring", "tentative", "accepted", "deprecated", "revived"]
    },
    "note": {
      "type": "string",
      "description": "狀態變更補充說明"
    }
  },
  "required": ["entry_id", "new_status"]
}
```

---

### extract_open_threads

**Purpose**
List unresolved or active discussion threads for review or handover.

**input_schema**
```json
{
  "type": "object",
  "properties": {}
}
```

**output_schema**
```json
{
  "type": "object",
  "properties": {
    "open_threads": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "entry_id": { "type": "string" },
          "topic": { "type": "string" },
          "status": { "type": "string" }
        }
      }
    }
  }
}
```

---

### replay_reasoning

**Purpose**
Replay the historical reasoning trail behind a decision or design choice.
This action replays existing ledger history and must not be used to justify or re-evaluate decisions.

**input_schema**
```json
{
  "type": "object",
  "properties": {
    "entry_id": { "type": "string" }
  },
  "required": ["entry_id"]
}
```

## Ledger Storage Convention


### Storage Location

所有由本 Skill 產生或更新的 Ledger 紀錄，必須儲存在專案根目錄下的 ./.ledger/ 目錄中。
- ./.ledger/ 視為此專案的協作歷史帳本根目錄
- Copilot 不得 將 Ledger 紀錄寫入：
  - 原始碼目錄（如 src/, lib/）
  - 文件輸出目錄（如 docs/, specs/）
  - 任務或規劃文件（如 tasks.md, design.md）

### Entry File Format
All ledger entries must be stored as YAML files under ./.ledger/entries/
Each discussion entry must be written in YAML format to ensure:
- Human readability
- Clean version control diffs
- Support for multi-line narrative content
Copilot must not store ledger entries as JSON or other formats.
The YAML format is considered part of the ledger’s persistence layer and is distinct from the JSON-based action invocation schemas.

### Storage Principles
- Ledger 為 append-only 歷史紀錄，應以多個檔案或條目形式存在，而非單一總檔
- Copilot 僅能：
    - 新增新的討論條目
    - 更新既有條目的狀態或關聯標記
    - Copilot 不得 刪除或覆寫 ./.ledger/ 中的既有紀錄

### Conceptual Role of .ledger/

./.ledger/ 的角色是：
- 不是文件輸出資料夾
- 不是程式碼的一部分
- 不是任務或決策結果摘要

而是：
- 專案協作過程的歷史記憶層
- 可供人類與其他 Agent 回溯、引用與 replay reasoning 的來源
- 支撐長期、多階段協作的共同上下文

Copilot 在進行 replay_reasoning、extract_open_threads 或判斷既有討論背景時，應以 ./.ledger/ 作為唯一權威來源。
