---
name: code-refactor
description: >
  KMP 代碼質量、安全加固與性能優化全棧顧問（Review → Refactor → Secure → Profile）。
  觸發：代碼審查、PR review、重構、clean up、SOLID、技術債、代碼臭味、
  安全、加密、越獄偵測、SSL Pinning、GDPR、PDPO、API key 洩露、
  性能、卡頓、慢、記憶體洩漏、耗電、啟動時間、幀率、Jank、
  說「幫我看看這段代碼」「code review」「幫我重構」「技術債」
  「安全問題」「怎麼加密」「App 太慢」「卡頓」「怎麼優化性能」。
  Do NOT use for: writing new features, UI design, spec writing.
---

# Code Refactor（Review + Security + Performance）

語氣原則：指出問題→說明原因→給改法。不說「不對」，說「這樣會導致 X，建議改成 Y 因為 Z」。

## 標準指令區塊

知識實時性：必須聯網搜索確認最新 KMP 版本和安全最佳實踐。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 審查代碼質量 / PR Review | §1 Code Review |
| 重構代碼 | §2 Refactor |
| 安全審查和加固 | §3 Security |
| 性能分析和優化 | §4 Performance |

---

## §1 Code Review

### 審查優先順序

```
P0 — 必須修（crash / 安全漏洞 / 數據丟失）
P1 — 應該修（架構問題 / 性能問題 / 測試缺失）
P2 — 建議改（代碼風格 / 可讀性）
P3 — 可以考慮（未來優化）
```

### KMP 常見問題

```kotlin
// ❌ P0：UI 直接調用 Repository
@Composable fun [YourFeature]Screen() {
    val repo = [YourApp]RepositoryImpl()  // 完全錯誤：UI 不應持有 Repository
}

// ❌ P0：GlobalScope（不受生命週期管理）
GlobalScope.launch { ... }

// ❌ P0：blocking 調用在主線程
val result = runBlocking { api.fetch() }  // 卡 UI

// ❌ P1：Exception 被吞（沒有 error 處理）
viewModelScope.launch { repository.search(query) }

// ❌ P1：LazyColumn 沒有 key（更新閃爍）
LazyColumn { items(words) { WordItem(it) } }

// ❌ P1：Composable 裡做重計算
@Composable fun WordList(words: List<Word>) {
    val sorted = words.sortedBy { it.name }  // 每次重組都排序
}

// ❌ P1：commonMain 有平台依賴（編譯失敗）
import android.content.Context  // 在 commonMain！

// ✅ 正確的 P0 問題修法
viewModelScope.launch {
    _uiState.update { it.copy(isLoading = true) }
    runCatching { repository.search(query) }
        .onSuccess { words -> _uiState.update { it.copy(words = words, isLoading = false) } }
        .onFailure { e -> _uiState.update { it.copy(error = e.message, isLoading = false) } }
}
LazyColumn { items(words, key = { it.id }) { WordItem(it) } }
```

### Review 輸出格式

```
## 代碼審查報告

### 🔴 P0 — 必須修（N個）
[問題] [行號] [原因] → [改法]

### 🟠 P1 — 應該修
### 🟡 P2 — 建議改
### ✅ 做得好的地方
### 結論：可 merge / 修完 P0+P1 後 merge / 需要重新設計
```

---

## §2 Refactor

### SOLID 原則（快速參考）

```kotlin
// S — 單一職責：ViewModel 不做格式化，UseCase 不做 UI
// ❌ ViewModel 做太多
class SearchViewModel : ViewModel() {
    fun formatResult(item: [YourEntity]): String { ... }  // 應在 Formatter
}

// D — 依賴倒置：依賴 interface，不依賴具體實現
// ❌
class [YourAction]UseCase(private val repo: [YourApp]RepositoryImpl)
// ✅
class [YourAction]UseCase(private val repo: [YourApp]Repository)
```

### 最常用重構模式

**Extract UseCase（ViewModel 邏輯太重）**

```kotlin
// ❌ ViewModel 包含業務邏輯（將 [YourFeature] / [YourEntity] 替換為你的 App）
class [YourFeature]ViewModel : ViewModel() {
    fun submit(input: String) {
        val cleaned = input.trim().lowercase()
        val results = if (cleaned.length < 2) emptyList()
                      else repository.fetch(cleaned).filter { it.isActive }
    }
}

// ✅ 提取到 UseCase
class [YourAction]UseCase(private val repo: [YourApp]Repository) {
    suspend operator fun invoke(input: String): List<[YourEntity]> {
        val cleaned = input.trim().lowercase()
        if (cleaned.length < 2) return emptyList()
        return repo.fetch(cleaned).filter { it.isActive }
    }
}
```

**Sealed Class 取代 Boolean 地獄**

```kotlin
// ❌ Boolean 地獄
data class UiState(val isLoading: Boolean, val isError: Boolean, val data: List<[YourEntity]>?)

// ✅ 清晰的狀態
sealed class [YourFeature]UiState {
    object Loading : [YourFeature]UiState()
    data class Success(val items: List<[YourEntity]>) : [YourFeature]UiState()
    data class Error(val message: String) : [YourFeature]UiState()
    object Empty : [YourFeature]UiState()
}
```

**Extract Composable（> 100 行拆分）**

```kotlin
// ❌ 一個 Composable 超過 100 行
@Composable fun SearchScreen(...) { /* 130行 */ }

// ✅ 拆成小 Composable
@Composable fun SearchScreen(...) {
    SearchBar(...)
    WordResultList(...)
    ErrorState(...)
}
```

### 技術債追蹤

```
分類：
A — 架構債：ViewModel 直接調 API、無 UseCase 層
B — 代碼債：函數 >50行、魔法數字、重複代碼
C — 測試債：核心功能無 unit test
D — 依賴債：庫超過 6 個月沒更新

優先級：P0立即修 → P1本Sprint → P2下版本 → P3技術債日

Boy Scout Rule：每次碰到一個文件，順手修一個小問題。
```

---

## §3 Security（OWASP Mobile Top 10）

### 安全快速檢查清單

```
□ M1：API key / token 不在源代碼中
□ M2：敏感數據用 EncryptedSharedPreferences / Keychain（不用明文）
□ M3：SSL Pinning（如果處理敏感數據）
□ M4：JWT token 有刷新機制
□ M5：本地 DB 有加密（SQLCipher for SQLDelight）
□ M8/M9：Release build 有 ProGuard / R8 混淆
```

### API Key 保護

```kotlin
// ❌ 禁止：源代碼中的 API key（會被逆向！）
const val API_KEY = "sk-1234567890"

// ✅ Android：local.properties + BuildConfig
// local.properties（加入 .gitignore）：API_KEY=sk-1234...
android {
    buildFeatures { buildConfig = true }
    defaultConfig {
        buildConfigField("String", "API_KEY", "\"${properties["API_KEY"]}\"")
    }
}
// 最佳實踐：API key 放後端，App 不直接持有

// ✅ iOS：Secrets.xcconfig（加入 .gitignore）
// CLAUDE_API_KEY = sk-1234...
// Info.plist: CLAUDE_API_KEY = $(CLAUDE_API_KEY)
```

### 安全數據存儲

```kotlin
// ❌ 禁止：SharedPreferences 明文存 token
sharedPrefs.edit().putString("token", authToken).apply()

// ✅ Android：EncryptedSharedPreferences
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM).build()
val securePrefs = EncryptedSharedPreferences.create(
    context, "secure_prefs", masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)

// ✅ iOS：Keychain（通過 expect/actual）
expect class SecureStorage {
    fun save(key: String, value: String)
    fun get(key: String): String?
}
```

### 合規（GDPR / PDPO）

```
香港 PDPO + GDPR 基本要求：
□ 隱私政策頁面（App Store 必須）
□ 只收集必要數據（搜索記錄不記錄個人身份）
□ 用戶可刪除自己的數據（搜索歷史清除功能）
□ Firebase Analytics 匿名化（不記錄設備 ID）
□ 廣告 ID：讓用戶可以退出追蹤
```

---

## §4 Performance

### 性能目標（根據你的 App 類型調整）

```
啟動時間：Cold start < 2 秒（Android < 1.5s，iOS < 1s）
主要操作延遲：本地操作 < 200ms，網絡操作 < 1s
幀率：60 fps（無 Jank，< 16ms/frame）
記憶體：< 150MB（一般使用）
耗電：不在後台消耗 CPU

> 告訴我你的 App 類型（工具、電商、社交等），我可以給出更具體的性能基準。
```

### Compose 性能優化

```kotlin
// ✅ 用 key 避免不必要的重組
LazyColumn {
    items(words, key = { it.id }) { WordItem(it) }
}

// ✅ 用 remember 緩存計算
val sortedWords = remember(words) { words.sortedBy { it.frequency } }

// ✅ derivedStateOf 避免不必要的重組
val showScrollToTop by remember {
    derivedStateOf { listState.firstVisibleItemIndex > 5 }
}

// ✅ 圖片用 Coil 異步加載（不阻塞 UI）
AsyncImage(model = word.imageUrl, contentDescription = word.word)

// ❌ Composable 中做網絡請求（嚴重問題）
@Composable fun WordItem(wordId: String) {
    val word = runBlocking { api.getWord(wordId) }  // 卡死 UI！
}
```

### 啟動優化

```kotlin
// ✅ 延遲加載非核心功能
class App : Application() {
    override fun onCreate() {
        super.onCreate()
        // 只初始化啟動必須的東西
        initKoin()
        // 延遲初始化 Analytics（用戶不需要等這個）
        lifecycleScope.launch { initAnalytics() }
    }
}

// ✅ Splash Screen API（Android 12+）
// res/values/themes.xml：windowSplashScreenAnimatedIcon
```

### 記憶體洩漏防範

```kotlin
// ✅ 在 ViewModel 中管理 Coroutine 生命週期
class SearchViewModel : ViewModel() {
    // viewModelScope 自動在 onCleared() 時取消
    private fun search(query: String) {
        viewModelScope.launch { ... }
    }
    // ❌ 不要在 ViewModel 外 launch 非 viewModelScope 的 Coroutine
}

// ✅ Compose 中用 DisposableEffect 清理
DisposableEffect(key) {
    val listener = ...
    onDispose { listener.unregister() }
}
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查最新 KMP / Compose 性能最佳實踐 | Context7 |
| 讀寫本地代碼文件 | Filesystem MCP |

## 自查清單

- [ ] Review：每個問題有 P0-P3 優先級 + 改法？
- [ ] Refactor：重構前有測試（無測試不重構）？
- [ ] Security：API key 不在源代碼？敏感數據用加密存儲？
- [ ] Performance：Compose 用了 key / remember / derivedStateOf？
- [ ] 重構和新功能分開 PR？
