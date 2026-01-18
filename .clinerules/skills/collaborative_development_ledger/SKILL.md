---
name: collaborative-development-ledger
description: 此 Skill 用於協助 Code Copilot 在與使用者長期協作開發專案的過程中，持續、結構化紀錄討論歷程、思考脈絡與決策演化。此 Skill 的目標不是產生規格文件（spec）、任務清單（tasks）或設計文件（design），而是保存「討論過哪些方案，各方當時的立場與理由，為什麼這樣想、有哪些分歧、哪些想法被保留或否決」等協作智慧，這些內容將構成一份可被人與其他 Agent 回溯、引用與推理的協作歷史帳本（Conversation / Reasoning Ledger）。
---
# Collaborative Development Ledger

## What This Skill Is NOT (Out of Scope / Guardrails)
- 不作為逐字聊天紀錄工具
- 不負責自動產生 spec / design / task 文件
- 不自行決定技術結論
- 不覆寫或刪除既有歷史紀錄（僅能新增或標記狀態）

## When to Use This Skill
本 Skill 不應在每一次對話中自動呼叫，而應在以下情境下由 Copilot 主動判斷使用：
- 一段討論涉及架構、流程、設計取捨、角色分工、技能邊界等關鍵決策
- 僅在出現「高價值討論事件」時使用（涉及架構、流程、設計取捨、角色分工、技能邊界等關鍵決策）。
- 出現多種方案、分歧觀點或尚未收斂的構想或當前討論與先前已存在的討論主題形成延續或衝突
- 不應逐句對話呼叫
- 使用者明確表達「先記下來」「之後可能會再用到」
- 所有操作皆為新增或狀態標記，不可刪除既有紀錄
- Skill 不直接與使用者互動，由 Copilot 內部呼叫

此 Skill 的定位是：慢速、稀疏、但高價值的記憶寫入

## What This Skill Does
此 Skill 維護一個長期存在、append-only 的協作歷史帳本（Ledger）。

### 資料模型概念（Conceptual Model）
- 以「討論事件（Discussion Entry）」為最小單位
- 每個 Entry 都有主題、多方立場、上下文與狀態的演化
- 討論可被後續 Entry 引用、延續或標記狀態

### Actions

#### append_discussion
新增一則新想法探索, 多方立場並列、尚未定案的討論紀錄

#### link_to_prior
將目前討論與既有 Ledger 條目建立關聯

#### mark_state
標記討論條目的生命週期狀態變化

#### extract_open_threads
列出目前尚未結案的重要討論主題

#### replay_reasoning
回溯某一決策或設計的歷史推理脈絡

**詳細的資料模型、Action schema 與 Ledger 儲存結構，請參考 `reference.md`**

## Design Principles
- 歷史不可覆寫，只能演化- 記錄「為什麼」，而不只是「是什麼」
- 結構化但不過度形式化
- 同時適合人類與 Agent 閱讀與推理
