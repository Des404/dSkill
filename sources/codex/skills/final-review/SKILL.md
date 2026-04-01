---
name: final-review
description: >
  發布前最終審查顧問，涵蓋代碼 Review 與部署檢查清單。
  觸發：PR review、發布前檢查、deploy checklist、上線前確認、
  merge 前審查、版本發布、TestFlight、Play Console Internal Track、
  說「準備發布了」「merge 前看看」「review 這個 PR」「發布 checklist」
  「上線前要做什麼」「這個改動安全嗎」「migration 有問題嗎」。
  Do NOT use for: writing new features, ongoing code quality improvement, daily development tasks.
---

# Final Review（PR審查 + 發布前Checklist）

## 標準指令區塊

知識實時性：必須聯網搜索確認最新 KMP 版本和 Store 審核要求。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 審查 PR / merge 前確認 | §1 Code Review |
| 版本發布前 Checklist | §2 Deploy Checklist |

---

## §1 Code Review（PR 審查）

### 審查框架（SCTQ）

```
S — Safety（安全）：有沒有安全漏洞、數據洩露風險？
C — Correctness（正確）：邏輯正確嗎？邊界案例有處理嗎？
T — Tests（測試）：有沒有測試？覆蓋主要場景了嗎？
Q — Quality（質量）：架構合理嗎？代碼可讀嗎？
```

### 優先等級

```
P0 — 必須修（不能 merge）
  • Crash / 數據丟失風險
  • 安全漏洞（API key 暴露、敏感數據明文存儲）
  • commonMain 有平台依賴（會導致 iOS/Android 編譯失敗）
  • GlobalScope / runBlocking（生命週期問題）

P1 — 應該修（merge 前修，有例外可協商）
  • 缺少 error handling
  • 架構違規（UI 直接調 Repository）
  • 新功能無 unit test
  • 重要性能問題（LazyColumn 無 key）

P2 — 建議改（不阻塞 merge）
  • 代碼可讀性問題
  • 命名不清晰
  • 可以更簡潔的寫法

P3 — 未來考慮
  • 技術債（記錄到 tech-debt tracker）
```

### KMP 特有審查點

```kotlin
// ❌ P0：UI 直接實例化 Repository（完全違反架構）
@Composable fun [YourFeature]Screen() {
    val repo = [YourApp]RepositoryImpl()  // 應通過 ViewModel + DI 注入
}

// ❌ P0：沒有 error handling 的 Coroutine
viewModelScope.launch { repository.search(query) }

// ❌ P0：Dispatchers.IO 在 commonMain（iOS 不支援）
viewModelScope.launch(Dispatchers.IO) { ... }

// ❌ P1：LazyColumn 無 key（重排序時 UI 閃爍）
LazyColumn { items(words) { WordItem(it) } }

// ❌ P1：Boolean 地獄
data class UiState(val isLoading: Boolean, val isError: Boolean, val isEmpty: Boolean)

// ✅ 正確
viewModelScope.launch {
    runCatching { repository.search(query) }
        .onSuccess { _state.update { it.copy(words = it) } }
        .onFailure { e -> _state.update { it.copy(error = e.message) } }
}
LazyColumn { items(words, key = { it.id }) { WordItem(it) } }
sealed class UiState { object Loading; data class Success(...); data class Error(...) }
```

### 與 Database Migration 相關的特別審查

```kotlin
// 審查 Migration 時必須確認：
// □ 版本號正確遞增
// □ 不刪除已有列（向後兼容）
// □ 新列有 DEFAULT 值
// □ FOREIGN KEY 關係正確
// □ 在舊數據庫上測試過（不只是空庫）

// SQLDelight Migration 示例（正確）
-- migrations/2.sqm（將 YourTable 替換為你的實際表名）
ALTER TABLE YourTable ADD COLUMN new_column INTEGER NOT NULL DEFAULT 0;  ✅ 有 DEFAULT
-- 不允許：DROP COLUMN（SQLite 不支援）
```

### PR 審查輸出格式

```
## PR 審查報告：[PR 標題]

### 🔴 P0 — 必須修（N個）
1. [問題描述] [文件:行號]
   原因：[為什麼這樣不行]
   改法：[具體代碼]

### 🟠 P1 — 應該修
### 🟡 P2 — 建議改
### ✅ 做得好的地方

### 結論
☐ 可以直接 merge
☑ 修完 P0 可以 merge
☐ 修完 P0 + P1 可以 merge
☐ 需要重新設計
```

---

## §2 Deploy Checklist（發布前）

### 代碼質量

```
□ 沒有 TODO / FIXME（或已記錄到 tech-debt）
□ 沒有 println / NSLog / Log.d（改用 Timber / 條件日誌）
□ 沒有 hardcoded 字串（全在 strings.xml / i18n 文件）
□ 沒有 unused imports / dead code
□ KDoc 注釋完整（所有 public API）
□ 沒有 API key / secret 在源代碼中
```

### 測試

```
□ 新代碼有 unit test（核心業務邏輯 > 80% 覆蓋）
□ Bug fix 有 regression test
□ 所有測試通過（./gradlew test）
□ Android 模擬器測試通過（API 30 + API 34）
□ iOS 模擬器測試通過（iOS 16 + iOS 18）
□ 實機測試（如果有影響 UI 的變更）
```

### 數據庫 Migration（有 Schema 變更時）

```
□ Migration 文件版本號正確（N.sqm 對應版本 N）
□ 在含有舊數據的設備上測試（不只是新安裝）
□ Migration 只用 DDL（ALTER TABLE / CREATE TABLE）
□ 新列有 DEFAULT 值（向後兼容）
□ FOREIGN KEY 關係沒有破壞
□ 測試從版本 1 升到最新版本的完整路徑
```

### 版本號

```kotlin
// build.gradle.kts
val versionCode = System.getenv("GITHUB_RUN_NUMBER")?.toInt() ?: localVersionCode
val versionName = "1.2.3"  // 語義化版本

// 確認：
// □ versionCode 比現有版本大（+1 或用 CI build number）
// □ versionName 按語義版本更新（major.minor.patch）
// □ iOS Build Number 也已遞增
```

### 發布前冒煙測試

```
□ 核心流程測試（你的 App 最重要的用戶動作，從頭到尾走一遍）
□ 網絡錯誤處理（關掉網絡，確認 offline 模式）
□ 首次安裝流程（模擬全新用戶）
□ 從上一個版本升級（確認 Migration 正常）
□ Push 通知測試（如果有通知相關改動）
□ 廣告顯示正常（如果有廣告相關改動）
```

### Android 發布檢查

```
□ Release build 開啟 ProGuard / R8 混淆
□ Signing Config 正確（用 release keystore）
□ Play Store 截圖尺寸正確
□ What's New 文字 ≤ 500 字符
□ targetSdkVersion 是最新（Google 要求）
□ 隱私政策 URL 有效
```

### iOS 發布檢查

```
□ 版本號和 Build Number 已更新
□ App Store Connect 截圖上傳（iPhone 6.7" 必須）
□ 所有 entitlements 正確（Associated Domains、Push 等）
□ Info.plist 的隱私使用說明完整（麥克風、相機等）
□ TestFlight 先測試 24-48 小時
□ 年齡分級沒有變化（或已更新）
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 讀本地代碼文件審查 | Filesystem MCP |
| 查 App Store 最新審核指南 | Context7 |

## 自查清單

- [ ] 所有 P0 問題已解決？
- [ ] Migration 在舊數據上測試過？
- [ ] Release build 已測試（不是 debug）？
- [ ] 版本號已正確遞增？
- [ ] TestFlight / Internal Track 已驗證通過？
