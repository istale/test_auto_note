# SKILL.md
## Collaborative Development Ledger Skill

---

### name
collaborative_development_ledger

---

### description
此 Skill 用於在與使用者長期協作開發專案時，**結構化紀錄討論歷程、思考脈絡與決策演化**。  
它不產生規格文件，而是保存「為什麼這樣想、有哪些分歧、哪些想法被保留或否決」等協作智慧，供未來人類或 Agent 回溯與推理。

---

### usage_guidelines
- 僅在出現「高價值討論事件」時使用（架構、流程、技能邊界、設計取捨）。
- 不應逐句對話呼叫。
- 所有操作皆為新增或狀態標記，不可刪除既有紀錄。
- Skill 不直接與使用者互動，由 Copilot 內部呼叫。

---

### actions

#### 1. append_discussion

**description**  
新增一則討論條目至協作帳本（Ledger）。

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

#### 2. link_to_prior

**description**  
將目前討論與既有 Ledger 條目建立關聯。

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

#### 3. mark_state

**description**  
更新某一討論條目的狀態。

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

#### 4. extract_open_threads

**description**  
列出目前尚未結案的重要討論主題。

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

#### 5. replay_reasoning

**description**  
回溯某一決策或設計的歷史推理脈絡。

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

---

### design_principles
- 歷史不可覆寫，只能演化
- 記錄「為什麼」，而不只是「是什麼」
- 結構化但不過度形式化
- 同時適合人類與 Agent 閱讀與推理

---

### non_goals
- 不作為逐字聊天紀錄
- 不自動產生 spec / task / design 文件
- 不自行決定技術結論

---

### future_extensions
- 與 spec / design 文件雙向連結
- 決策脈絡自動摘要
- 多 Agent 協作回放
- 作為高品質 reasoning 資料集來源

