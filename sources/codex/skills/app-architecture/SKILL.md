---
name: app-architecture
description: >
  KMP + Compose Multiplatform 架構設計全棧顧問，涵蓋架構決策、後端API、多語言、無障礙。
  觸發：KMP 架構、Clean Architecture、MVVM、expect/actual、Koin DI、Voyager 導航、
  Gradle 設置、Gradle 報錯、iOS 橋接、API 設計、Ktor Client、後端選型、Firebase、
  Supabase、JWT 認證、多語言 i18n、Lyricist、strings.xml、RTL、accessibility、
  VoiceOver、TalkBack、WCAG、ADR、技術選型、架構決定、
  說「怎麼架構」「expect actual」「iOS 入口點」「後端用什麼」「多語言怎麼做」
  「加無障礙」「VoiceOver 怎麼支持」「為什麼選這個技術」。
  Do NOT use for: UI pixel design, non-KMP projects, backend-only tasks unrelated to mobile.
---

# App Architecture（KMP架構 + API後端 + 多語言 + 無障礙）

## 標準指令區塊

知識實時性：必須先聯網搜索確認最新版本號和API（版本更新快）。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用前：了解你的 App

首次使用時，先請用戶回答以下問題，讓建議更精準：

1. **你的 App 核心功能是什麼？**（例：電商、健身追蹤、筆記、社交）
2. **主要實體是什麼？**（例：商品、訓練記錄、筆記、帖子）
3. **核心動作是什麼？**（例：瀏覽商品、記錄運動、創建筆記、發帖）
4. **需要後端嗎？還是純本地 App？**
5. **目標用戶群？**（影響無障礙標準和多語言優先級）

> 如果用戶直接問技術問題，可以先回答，再在結尾補充「請告訴我你的 App 類型，讓建議更貼合」。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| KMP 分層架構 / 模組設計 | §1 KMP 架構 |
| 記錄技術選型決定 | §2 ADR |
| API / 後端設計 | §3 API & Backend |
| 多語言 i18n | §4 Localization |
| VoiceOver / TalkBack / WCAG | §5 Accessibility |

---

## §1 KMP 架構

### 當前推薦版本（需聯網確認最新）

```
Kotlin：           2.2.20
Compose Multi：    1.10.3（需要 Kotlin ≥ 2.1.0）
AGP：              9.0+（android.application 與 kmp plugin 需分開）
Koin：             4.0.x
Ktor：             3.x
SQLDelight：       2.x
Voyager：          1.1.x
Coil：             3.x（KMP）
```

### 目錄分層原則

```
shared/
├── commonMain/
│   ├── domain/          ← Entities, UseCases, Repository interfaces（純 Kotlin）
│   ├── data/
│   │   ├── remote/      ← Ktor API + DTOs
│   │   ├── local/       ← SQLDelight queries
│   │   └── repository/  ← Repository 實現
│   └── presentation/
│       ├── viewmodel/   ← 共用 ViewModel（StateFlow）
│       └── state/       ← UiState, UiEvent sealed classes
├── androidMain/         ← Android Context、系統 API
└── iosMain/             ← iOS 橋接、平台 API
androidApp/              ← 薄層，只有 MainActivity（AGP 9.0 要求獨立）
iosApp/                  ← Xcode 項目（Swift 入口點）
```

### 禁止事項

```
❌ UI 層直接調用 Repository
❌ ViewModel 直接調用 API
❌ domain 層 import android.* 或任何平台相關
❌ Dispatchers.IO 在 commonMain（iOS 不支援）→ 用 Dispatchers.Default
❌ hardcoded strings（全放 i18n 文件）
```

### iOS 入口點（必備橋接代碼）

```swift
// iosApp/ContentView.swift
struct ContentView: View {
    var body: some View { ComposeView().ignoresSafeArea(.all) }
}
struct ComposeView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        MainViewControllerKt.MainViewController()
    }
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {}
}
// iosApp/iOSApp.swift
@main struct iOSApp: App {
    init() { KoinHelperKt.doInitKoin() }
    var body: some Scene { WindowGroup { ContentView() } }
}
```

```kotlin
// shared/iosMain/MainViewController.kt
import androidx.compose.ui.window.ComposeUIViewController
fun MainViewController() = ComposeUIViewController { App() }
```

### expect/actual 模式

```kotlin
// commonMain
expect class PlatformContext
expect fun getPlatformName(): String
expect fun currentTimeMillis(): Long

// androidMain
actual class PlatformContext(val context: Context)
actual fun getPlatformName() = "Android ${Build.VERSION.RELEASE}"
actual fun currentTimeMillis() = System.currentTimeMillis()

// iosMain
actual class PlatformContext
actual fun getPlatformName() = UIDevice.currentDevice.systemName()
actual fun currentTimeMillis() = (NSDate().timeIntervalSince1970 * 1000).toLong()
```

### ViewModel 規範（以主要功能為例）

```kotlin
// 將 [YourAction] / [YourEntity] 替換為你的 App 實際動作和實體
class [YourFeature]ViewModel(
    private val [yourAction]: [YourAction]UseCase,
    private val get[YourEntity]: Get[YourEntity]UseCase
) : ViewModel() {
    private val _uiState = MutableStateFlow([YourFeature]UiState())
    val uiState: StateFlow<[YourFeature]UiState> = _uiState.asStateFlow()

    fun onEvent(event: [YourFeature]UiEvent) = when (event) {
        is [YourFeature]UiEvent.Submit -> submit(event.input)
        is [YourFeature]UiEvent.Clear -> clear()
    }
    private fun submit(input: String) {
        viewModelScope.launch {
            _uiState.update { it.copy(isLoading = true) }
            [yourAction](input)
                .onSuccess { result -> _uiState.update { it.copy(result = result, isLoading = false) } }
                .onFailure { e -> _uiState.update { it.copy(error = e.message, isLoading = false) } }
        }
    }
}
data class [YourFeature]UiState(
    val result: List<[YourEntity]> = emptyList(),
    val isLoading: Boolean = false,
    val error: String? = null
)
sealed class [YourFeature]UiEvent {
    data class Submit(val input: String) : [YourFeature]UiEvent()
    data object Clear : [YourFeature]UiEvent()
}
```

### Koin DI 設置

```kotlin
// commonMain — 將 [YourApp]/[YourFeature] 替換為實際名稱
val domainModule = module {
    factory { [YourAction]UseCase(get()) }
    factory { Get[YourEntity]DetailUseCase(get()) }
}
val dataModule = module {
    single<[YourApp]Repository> { [YourApp]RepositoryImpl(get(), get()) }
    single { [YourApp]ApiClient(get()) }
}
val presentationModule = module {
    viewModel { [YourFeature]ViewModel(get(), get()) }
    viewModel { [YourEntity]DetailViewModel(get()) }
}
// androidMain
fun initKoin(context: Context) = startKoin {
    androidContext(context)
    modules(domainModule, dataModule, presentationModule)
}
// iosMain（供 Swift 調用）
fun initKoin() = startKoin { modules(domainModule, dataModule, presentationModule) }
```

> 完整 Gradle 設置 → `references/gradle-setup.md`
> Voyager 導航詳細 → `references/navigation.md`

---

## §2 ADR（架構決策記錄）

```markdown
# ADR-[編號]: [簡短標題]
日期：YYYY-MM-DD
狀態：提議中 | 已採用 | 已棄用

## 背景
[為什麼需要這個決定？]

## 考慮的選項
1. [選項 A]  2. [選項 B]

## 決定
選擇 [選項 X]。

## 原因
- [優點 1]  - [為什麼其他選項不適合]

## 代價
- [妥協點]

## 後果
[對未來的影響]
```

### 常見 KMP 項目 ADR 模板集

```
ADR-001: KMP + Compose Multiplatform（取代分開維護 iOS/Android）
ADR-002: SQLDelight（取代 Room，支持 iOS 離線存儲）
ADR-003: Ktor Client（跨平台網絡）
ADR-004: Koin（取代 Hilt，KMP 友好）
ADR-005: Voyager（Compose 跨平台導航）
ADR-006: [後端選型]（Firebase / Supabase / 自建 Ktor）
ADR-007: 離線優先 vs 在線優先架構
```

> 根據你的 App 特性選擇填寫相關 ADR，不必全部套用。

**需要寫 ADR 的情況：** 選擇技術庫、架構模式、平台策略、數據庫、後端選型

---

## §3 API & Backend

### 後端選型

| 選項 | 適合場景 | 推薦時機 |
|------|---------|---------|
| Firebase | 快速 MVP、實時同步、Push 通知 | MVP 期 |
| Supabase | PostgreSQL、全文搜索、結構化數據 | 需要複雜查詢 |
| Ktor Server | 完全自控、Kotlin 全棧 | 成熟期 |
| PocketBase | 個人/小團隊、快速原型 | 早期驗證 |

> 選型建議：先告訴我你的 App 需要什麼數據操作（全文搜索？實時更新？離線優先？），再給出具體推薦。

### Ktor Client 設置（commonMain）

```kotlin
expect fun createHttpClient(config: HttpClientConfig<*>.() -> Unit = {}): HttpClient

// androidMain
actual fun createHttpClient(config: HttpClientConfig<*>.() -> Unit) =
    HttpClient(OkHttp) {
        install(HttpTimeout) { requestTimeoutMillis = 15_000 }
        install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
        install(Logging) { level = LogLevel.BODY }
        config()
    }

// iosMain
actual fun createHttpClient(config: HttpClientConfig<*>.() -> Unit) =
    HttpClient(Darwin) {
        install(HttpTimeout) { requestTimeoutMillis = 15_000 }
        install(ContentNegotiation) { json(Json { ignoreUnknownKeys = true }) }
        config()
    }
```

### REST API 規範（通用模板）

```
# 根據你的 App 實體替換 [resource]
GET  /v1/[resources]?q=query&filter=value   列表/搜索
GET  /v1/[resources]/{id}                   詳情
POST /v1/[resources]                        創建
PUT  /v1/[resources]/{id}                   更新
DELETE /v1/[resources]/{id}                 刪除

POST /v1/users/[favorites]                  用戶收藏/關注
GET  /v1/users/[history]                    用戶歷史

HTTP 狀態碼：200成功 / 201已創建 / 400請求錯誤 / 401未認證 / 404找不到 / 429限流 / 500服務器錯誤
```

### 錯誤處理

```kotlin
sealed class ApiError : Exception() {
    data class NetworkError(override val message: String) : ApiError()
    data class ServerError(val code: Int, override val message: String) : ApiError()
    data object Unauthorized : ApiError()
    data object RateLimited : ApiError()
}

// Repository 實現中的錯誤映射
override suspend fun fetch[YourEntity](id: String): Result<[YourEntity]> =
    apiClient.get[YourEntity](id)
        .map { dto -> dto.toDomain() }
        .mapFailure { throwable ->
            when (throwable) {
                is ClientRequestException -> when (throwable.response.status.value) {
                    401 -> ApiError.Unauthorized
                    429 -> ApiError.RateLimited
                    else -> ApiError.ServerError(throwable.response.status.value, throwable.message ?: "")
                }
                is IOException -> ApiError.NetworkError(throwable.message ?: "Network error")
                else -> throwable
            }
        }
```

---

## §4 Localization（多語言）

### 推薦方案：Lyricist

```kotlin
// build.gradle.kts
implementation("cafe.adriel.lyricist:lyricist:1.7.0")

// Strings.kt（commonMain）— 根據你的 App 替換字符串
data class Strings(
    val appName: String,
    val mainActionPlaceholder: String,
    val emptyState: (query: String) -> String,
    val resultCount: (Int) -> String
)
// 英文
val EnStrings = Strings(
    appName = "[YourApp]",
    mainActionPlaceholder = "[Action placeholder]...",
    emptyState = { query -> "No results for \"$query\"" },
    resultCount = { count -> if (count == 1) "1 result" else "$count results" }
)
// 繁體中文（香港）
val ZhHantHKStrings = Strings(
    appName = "[你的App]",
    mainActionPlaceholder = "[動作提示]...",
    emptyState = { query -> "找不到「$query」的結果" },
    resultCount = { count -> "${count} 個結果" }
)
```

### RTL 支援

```kotlin
// Compose RTL 自動支援（Row/Column 自動翻轉）
// 需要手動處理：絕對定位、圖片方向

val layoutDirection = LocalLayoutDirection.current
BasicTextField(
    textStyle = TextStyle(
        textAlign = if (layoutDirection == LayoutDirection.Rtl) TextAlign.End else TextAlign.Start
    )
)
```

---

## §5 Accessibility（無障礙）

### 目標：WCAG 2.1 Level AA

```kotlin
// ✅ 圖標按鈕必須有 contentDescription
IconButton(onClick = { onPrimaryAction() }) {
    Icon(Icons.Default.PlayArrow, contentDescription = "[描述這個動作]")
}

// ✅ 裝飾性圖片設 null
Image(painter = ..., contentDescription = null)

// ✅ 可點擊元素最小 44dp
Box(modifier = Modifier.size(44.dp).clickable { })

// ✅ 用 sp 不用 dp（支持用戶字體大小設置）
Text(text = item.description, style = TextStyle(fontSize = 16.sp))

// ✅ 不能只用顏色表達信息，同時用圖示+文字
Row {
    Icon(if (isError) Icons.Default.Error else Icons.Default.Check,
         contentDescription = if (isError) "錯誤" else "成功")
    Text(if (isError) "輸入有誤" else "輸入正確")
}
```

### 無障礙標準（根據目標用戶調整）

| 用戶群 | 字體 | 觸控目標 | 對比度 |
|--------|------|---------|--------|
| 一般用戶 | 正文 ≥ 16sp | ≥ 44dp | ≥ 4.5:1（AA） |
| 長者/視障友好 | 正文 ≥ 18sp | ≥ 56dp | ≥ 7:1（AAA） |
| 兒童 App | 正文 ≥ 18sp | ≥ 56dp | ≥ 4.5:1（AA） |

> 告訴我你的主要用戶群，我可以給出更具體的無障礙實施建議。

### 測試

```kotlin
// Compose 無障礙測試
composeTestRule.onNodeWithContentDescription("[你的動作描述]")
    .assertIsDisplayed().performClick()

// 手動：開啟 TalkBack（Android）或 VoiceOver（iOS），逐屏確認描述
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 KMP / Ktor / Compose 官方文件 | Context7 |
| 讀寫本地架構文件 / ADR | Filesystem MCP |

## 自查清單

- [ ] 已聯網搜索確認所有版本號？
- [ ] domain 層無任何平台依賴？
- [ ] 沒有使用 Dispatchers.IO 在 commonMain？
- [ ] iOS 橋接代碼完整（ContentView + initKoin）？
- [ ] 所有可點擊元素有 contentDescription 或 null（裝飾性）？
