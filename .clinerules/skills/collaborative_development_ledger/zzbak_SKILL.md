---
name: collaborative-development-ledger
description: 此 Skill 用於協助 Code Copilot 在與使用者長期協作開發專案的過程中，**持續、結構化紀錄討論歷程、思考脈絡與決策演化**。此 Skill 的目標不是產生規格文件（spec）、任務清單（tasks）或設計文件（design），而是保存「討論過哪些方案，各方當時的立場與理由，為什麼這樣想、有哪些分歧、哪些想法被保留或否決」等協作智慧，這些內容將構成一份 可被人與其他 Agent 回溯、引用與推理的協作歷史帳本（Conversation / Reasoning Ledger）。
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

### 資料模型概念（Conceptual Model）
- 核心特性：
	- 以「討論事件（Discussion Entry）」為最小單位
	- 每個 Entry 都有主題、立場、狀態與上下文
	- 可被後續討論引用或更新狀態

### Actions

#### append_discussion
新增一則討論條目至Ledger, 適用於:
- 新想法探索
- 多方立場並列
- 尚未定案的討論
- summary應為事實性與觀點並列的摘要，不應包含取捨判斷或結論性語句

#### link_to_prior
將目前討論與既有 Ledger 條目建立關聯, 適用於
- 延續先前未結案的討論
- 指出「這其實是同一條問題線」

#### mark_state
更新某一討論條目的狀態, 狀態建議值:
- exploring：探索中
- tentative：暫定
- accepted：已採納
- deprecated：已淘汰
- revived：重新啟用

#### extract_open_threads
列出目前尚未結案的重要討論主題, 適用於：
- 專案回顧
- 新 Agent 接手
- 長時間中斷後重新進入專案

#### replay_reasoning
回溯某一決策或設計的歷史推理脈絡, 適用於:
- 為什麼當初沒有選方案 B？
- 為什麼後來又改回方案 A？

## Ledger Storage Convention

### Ledger Storage Location

所有由本 Skill 產生或更新的 Ledger 紀錄，必須儲存在專案根目錄下的 ./.ledger/ 目錄中。
- ./.ledger/ 視為此專案的協作歷史帳本根目錄
- Copilot 不得 將 Ledger 紀錄寫入：
- 原始碼目錄（如 src/, lib/）
- 文件輸出目錄（如 docs/, specs/）
- 任務或規劃文件（如 tasks.md, design.md）

Storage Principles
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

## Design Principles
- 歷史不可覆寫，只能演化- 記錄「為什麼」，而不只是「是什麼」
- 結構化但不過度形式化
- 同時適合人類與 Agent 閱讀與推理
