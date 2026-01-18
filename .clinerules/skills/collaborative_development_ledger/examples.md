# Collaborative Development Ledger — Examples

本文件示範 Code Copilot 在實際協作過程中，**何時、為何、以及如何** 使用 collaborative-development-ledger 這個 Skill。  
此檔案僅作為使用範例，並不會被視為專案的協作歷史紀錄。

---

## Example 1：新架構討論，尚未收斂

**情境**  
使用者與 Copilot 討論是否要將協作紀錄（ledger）與 spec / tasks 文件拆分，但尚未形成共識。

**判斷依據**
- 出現多種方案
- 涉及架構與 Skill 邊界
- 尚未定案

→ Copilot 應呼叫 `append_discussion`

```json
{
  "topic": "是否將協作記錄與 spec / tasks 拆分成獨立層",
  "summary": "討論是否應將協作過程紀錄（ledger）與 spec、tasks 文件分離，以避免角色與責任混淆。",
  "positions": [
    {
      "by": "user",
      "stance": "ledger 應獨立於 spec 與 tasks",
      "rationale": "協作過程與最終輸出責任不同，混在一起容易被誤用。"
    },
    {
      "by": "assistant",
      "stance": "拆分有助於長期協作與多 Agent 接手",
      "rationale": "獨立 ledger 可保存被否決想法，避免 spec 反覆改寫。"
    }
  ],
  "status": "exploring"
}
```

---

## Example 2：延續既有未結案討論

**情境**  
目前的對話實際上是在延續先前關於 Skill 邊界的討論。

→ Copilot 應呼叫 `link_to_prior`

```json
{
  "current_entry_id": "2026-01-25_skill-doc-splitting",
  "link_to": [
    {
      "entry_id": "2026-01-20_skill-boundary-definition",
      "relation": "延續先前對 Skill 邊界的討論"
    }
  ]
}
```

---

## Example 3：討論形成共識，標記狀態

**情境**  
團隊已同意採用 SKILL.md + reference.md 的拆分方式。

→ Copilot 應呼叫 `mark_state`

```json
{
  "entry_id": "2026-01-25_skill-doc-splitting",
  "new_status": "accepted",
  "note": "決定採用 SKILL.md + reference.md 的拆分結構。"
}
```

---

## Example 4：新 Agent 接手專案

**情境**  
新的 Agent 加入專案，需要快速了解目前尚未結案的重要討論。

→ Copilot 應呼叫 `extract_open_threads`

```json
{}
```

---

## Example 5：回顧歷史推理（replay_reasoning）

**情境**  
使用者詢問：  
「為什麼一開始沒有把 replay_reasoning 設計成可以指定立場？」

→ Copilot 應呼叫 `replay_reasoning`

```json
{
  "entry_id": "2026-01-25_replay-reasoning-design"
}
```

Copilot 應直接回傳**人類可讀的敘事文字**，  
而非 JSON、結論或建議。
