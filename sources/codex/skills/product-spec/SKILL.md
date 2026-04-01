---
name: product-spec
description: >
  功能規格設計與開發任務規劃顧問，防止 AI Scope Drift。
  觸發：新功能設計、PRD、需求文件、技術設計文件、任務規劃、Sprint 計劃、
  說「我要做一個功能」「怎麼開始這個 feature」「幫我設計這個需求」
  「寫 Spec」「寫 Task Contract」「拆任務」「優先做什麼」「這週做什麼」
  「任務太多不知道從哪開始」「AI 跑偏了」「幫我 Focus」「RICE 評分」。
  Do NOT use for: technical implementation, UI design, or tasks not related to spec or planning.
---

# Product Spec（功能規格 + 任務規劃）

## 標準指令區塊

知識實時性：涉及技術方案時，先聯網搜索確認最新資訊。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 設計新功能 Spec / PRD | §1 Feature Spec |
| 鎖定工作範圍 | §2 Task Contract |
| 拆分和排序任務 | §3 任務分解 + RICE |
| 規劃這週/今天的工作 | §4 週/日計劃 |
| AI 跑偏了，需要 re-anchor | §5 Focus Lock |

---

## §1 Feature Spec 模板

> 先寫規格，後寫代碼。每個功能都從這裡開始。

```markdown
# Feature: [功能名稱]

## 目標
用一句話說清楚這個功能為用戶解決什麼問題。

## 用戶故事
作為 [用戶類型]，我想要 [功能]，以便 [好處]。

## 驗收標準（AC）— 具體可測試
- [ ] AC1: [具體條件，例：搜索框輸入<2字符時，不發起網絡請求]
- [ ] AC2: [具體條件]
- [ ] AC3: 錯誤情況處理 [例：網絡錯誤顯示重試按鈕]

## 技術設計

### 數據流
用戶操作 → UI Event → ViewModel → UseCase → Repository → API/DB

### 新增/修改組件
- ViewModel: [名稱]（修改/新增）
- UseCase: [名稱]（新增）
- Repository: [名稱]（新增方法）
- UI: [Screen名稱]（修改）

### API / DB 變更（如有）
[接口設計或Schema變更]

## 測試計劃
- Unit: [UseCase / ViewModel 測試]
- Integration: [Repository 測試]
- UI: [E2E 流程測試]

## 範圍外（這次不做）
[明確列出，避免 scope creep]
```

### 好的 vs 差的驗收標準

```
✅ 好的 AC（具體、可測試）：
• 搜索框輸入少於2字符時，不發起搜索請求
• 搜索結果按相關性降序排列，完全匹配排首位
• 網絡錯誤時顯示「請檢查網絡連接」，有重試按鈕

❌ 差的 AC（模糊）：
• 搜索功能正常工作
• 顯示錯誤信息
• 性能良好
```

---

## §2 Task Contract（🔒 Scope 錨點）

> AI 失焦根本原因：沒有書面 Scope 錨點。每個 Session 開始必須先寫。

```markdown
## 🔒 Task Contract — [功能名稱]
生成時間：[當前日期時間]

### 🎯 Goal（一句話）
[做什麼] → [為誰] → [達成什麼結果]

### 📋 In Scope（這次做什麼）
- [ ] [具體任務 1]（預計 X 小時）
- [ ] [具體任務 2]（預計 X 小時）

### 🚫 Non-Goals（這次不做）
- [功能 A]（原因：等 V2.0）
- [功能 B]（原因：另一個 PR）

### ✅ Done Criteria（完成標準）
- [ ] [可測試、可驗證的標準 1]
- [ ] 所有 unit test 通過
- [ ] Code review 無 blocker

### ⏰ Time-box
目標：[X 小時] | 超時 checkpoint：[X 小時後重新評估]
```

**Task Contract 規則：**
- 開始後 Non-Goals 不得移入 In Scope（需另開新 Contract）
- AI 每完成 3 個任務，re-read Contract 確認方向

---

## §3 任務分解（INVEST 原則）

| 字母 | 含義 | 檢查問題 |
|------|------|---------|
| **I** Independent | 可獨立開始？ |
| **N** Negotiable | 實現方式有彈性？ |
| **V** Valuable | 完成後有可見價值？ |
| **E** Estimable | 工時可估算（±50%）？ |
| **S** Small | AI 一個 Session 可完成（≤2hr）？ |
| **T** Testable | 有明確驗收標準？ |

```
✅ 合格任務（30min-2hr）：
• 「實現 SearchRepository.searchOffline()」
• 「寫 SearchViewModel unit test（3個case）」

❌ 太大（需拆）：「實現整個離線搜索功能」
❌ 太模糊（需明確）：「優化搜索」→ 問：優化什麼？
```

### RICE 優先級評分

```
RICE = (Reach × Impact × Confidence) / Effort
>5 本Sprint | 2-5 下Sprint | <2 Backlog
```

### 任務狀態看板

```markdown
### 🔵 Pending（待開始）
- [ ] [任務] · 預計 X hr

### 🟡 In Progress（⚠️ 同時只能有 1 個）
- [x] [任務] · 開始：[時間]

### ✅ Completed
- [x] [任務] · PR #[號碼]

### 🔴 Blocked
- [ ] [任務] · 原因：[X] · 解除條件：[Y]
```

### TDD 節奏（Red-Green-Refactor）

```kotlin
// 1. RED：先寫失敗的測試（將 [YourAction] / [YourEntity] 替換為你的 App 實體）
@Test fun `action with invalid input returns empty`() {
    val result = runTest { [YourAction]UseCase(fakeRepo)("") }
    assertTrue(result.isEmpty())
}

// 2. GREEN：最小實現讓測試通過
suspend operator fun invoke(input: String): List<[YourEntity]> {
    if (input.length < 2) return emptyList()
    return repo.fetch(input)
}

// 3. REFACTOR：清理，保持測試通過
```

---

## §4 週/日計劃模板

### 週計劃

```markdown
## 本週目標 — W[週號]

### 🔒 本週 Goal
[最重要的一件事] → Done Criteria：[成功標準]

### P0（必須完成）
- [ ] [任務] · RICE: [分數] · 預計 X hr

### P1（應該完成）
- [ ] [任務] · 預計 X hr

### P2（如果有時間）
### 🚫 本週不做
```

### 日計劃

```markdown
## 今天 — [日期]

### 🔒 今日 Focus（只做這一件）
[任務名] → Done Criteria：[完成標準]

### 結束前確認
□ commit 今天的代碼
□ 更新任務狀態板
□ 明天第一件事：[任務名]
```

---

## §5 Focus Lock（防 Scope Drift）

### Session 開始 Checklist

```
□ Task Contract 已寫好並固定
□ In Progress 任務只有 1 個
□ Non-Goals 清單已確認（今天不碰）
```

### Scope Creep 防衛（用戶提出範圍外需求時）

```
「這個功能不在當前 Task Contract 的 In Scope 內。
 選擇：
 A) 記錄 Backlog，完成當前任務後再規劃
 B) 終止當前 Contract，重新寫新 Contract 包含這個功能
 你選哪個？」
```

### Re-anchor 觸發點

- 完成每 3 個子任務後
- 用戶提出新需求時
- 工作超過 Time-box 的 50%

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 讀寫 spec / 任務文件 | Filesystem MCP |

## 自查清單

- [ ] Feature Spec 有完整 AC（具體可測試）？
- [ ] Task Contract 已寫，In Scope / Non-Goals 已鎖定？
- [ ] 每個任務符合 INVEST，≤2hr per Session？
- [ ] In Progress 只有 1 個任務？
