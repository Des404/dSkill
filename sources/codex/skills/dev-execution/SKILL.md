---
name: dev-execution
description: >
  KMP 開發執行顧問，涵蓋 Debug、測試自動化與 AI 子代理並行開發。
  觸發：bug、crash、崩潰、異常行為、測試失敗、Firebase Crashlytics、ANR、
  unit test、integration test、UI test、TDD、Turbine、MockK、測試策略、
  大任務拆分、並行開發、子代理、subagent、任務拆分、多任務、
  說「這裡有問題」「為什麼不工作」「crash 了」「找不到原因」
  「怎麼測試」「寫 test」「測試覆蓋率」「這個任務太大」「怎麼更快完成」
  「並行處理」「委派任務」「Android 正常但 iOS 有問題」。
  Do NOT use for: new feature design, spec writing, UI design, product planning.
---

# Dev Execution（Debug + 測試 + 子代理並行開發）

## 標準指令區塊

知識實時性：必須聯網搜索確認 KMP / Compose Multiplatform 最新版本行為。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| Debug bug / crash | §1 Debug |
| 寫測試 / 制定測試策略 | §2 Testing |
| 大任務拆分 + 並行加速 | §3 子代理開發 |

---

## §1 Debug（開發期 + 生產崩潰）

### 5步 Debug 流程

```
Step 1: 重現（Reproduce）
→ 構造最小重現場景，確認 bug 是穩定的

Step 2: 隔離（Isolate）
→ 二分法：把問題範圍縮小到一個函數/組件

Step 3: 診斷（Diagnose）
→ 讀 Logcat / Console，找到 Exception 和 Stack Trace

Step 4: 修復（Fix）
→ 針對根本原因修復，不要 patch symptom

Step 5: 驗證（Verify）
→ 跑測試，確認修復有效且沒有回歸
```

### KMP 常見問題快查

| 症狀 | 根本原因 | 解法 |
|------|---------|------|
| iOS 黑屏 | ComposeUIViewController 沒有 `ignoresSafeArea` | `ComposeView().ignoresSafeArea(.all)` |
| Koin 注入失敗 | Module 沒在 iOS `initKoin()` 中加載 | 確認 iOS `KoinHelperKt.doInitKoin()` 包含所有 module |
| Coroutine 不執行 | 用了 `Dispatchers.IO` 在 iOS | 改用 `Dispatchers.Default` |
| Flow 不更新 | `StateFlow` 在 Compose 沒用 `collectAsStateWithLifecycle()` | 改用 `collectAsStateWithLifecycle()` |
| Compose Preview 崩潰 | Preview 的 ViewModel 有真實依賴 | 用 fake state 預覽，不要實例化 ViewModel |
| Gradle Sync 失敗 | AGP 9.0+ 要求 android.application 分離 | `androidApp/` 只保留 MainActivity，shared/ 放邏輯 |
| SQLDelight iOS crash | Driver 未初始化 | 確認 `DatabaseDriverFactory` 在 Koin module 中 |
| sealed class 編譯警告 | 用 `object` 而非 `data object` | Kotlin 1.9+ 改 `data object Clear` |

### 生產崩潰監控（Firebase Crashlytics）

```kotlin
// 設置（androidMain）
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        if (!BuildConfig.DEBUG) {
            FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(true)
        }
    }
}

// 記錄非致命錯誤
FirebaseCrashlytics.getInstance().recordException(exception)

// 自定義 key（根據你的 App 特性添加診斷上下文）
FirebaseCrashlytics.getInstance().apply {
    setCustomKey("user_action", action)       // 用戶正在做什麼
    setCustomKey("feature", featureName)      // 哪個功能
    setCustomKey("offline_mode", isOffline)   // 是否離線
}

// 崩潰率目標：> 99.5% crash-free users
// Firebase Console → Crashlytics → 按 Issues 排序
```

### Logcat 常用命令

```bash
# 只看你的 App log（Android）
adb logcat -s "YourApp" "KMP" "Koin"

# 看 KMP 相關崩潰
adb logcat | grep -i "exception\|fatal\|crash"

# iOS Simulator log
xcrun simctl spawn booted log stream --predicate 'subsystem == "com.yourapp"'
```

---

## §2 Testing（測試策略 + 實現）

### 測試金字塔

```
         [E2E / UI Tests]     少量，慢，高信心
        [Integration Tests]   中量
   [Unit Tests] ← 主力         大量，快，低維護成本
```

### 技術棧（2026 推薦）

```
Unit：      kotlin.test（commonMain 跨平台）
Flow 測試：  Turbine（測試 Flow 發射）
Mock：       MockK（KMP 兼容）
Coroutines： kotlinx-coroutines-test
Compose UI： compose-ui-test（Android）
```

### Unit Test 規範

```kotlin
// 將 [YourAction] / [YourEntity] / [YourApp] 替換為你的 App 實際名稱
class [YourAction]UseCaseTest {
    private val fakeRepository = Fake[YourApp]Repository()
    private val useCase = [YourAction]UseCase(fakeRepository)

    @Test
    fun `returns empty list when input is invalid`() = runTest {
        val result = useCase("")
        assertTrue(result.isSuccess)
        assertTrue(result.getOrThrow().isEmpty())
    }

    @Test
    fun `returns results when input matches`() = runTest {
        fakeRepository.setItems(listOf([YourEntity].fixture()))
        val result = useCase("valid_input")
        assertEquals(1, result.getOrThrow().size)
    }

    @Test
    fun `returns error when repository throws`() = runTest {
        fakeRepository.setError(Exception("Network error"))
        val result = useCase("valid_input")
        assertTrue(result.isFailure)
    }
}
```

### ViewModel + Flow 測試（Turbine）

```kotlin
class [YourFeature]ViewModelTest {
    @get:Rule val mainDispatcherRule = MainDispatcherRule()
    private val fakeUseCase = Fake[YourAction]UseCase()
    private lateinit var viewModel: [YourFeature]ViewModel

    @Before fun setup() { viewModel = [YourFeature]ViewModel(fakeUseCase) }

    @Test
    fun `action emits loading then results`() = runTest {
        fakeUseCase.setResult(listOf([YourEntity].fixture()))
        viewModel.uiState.test {
            val initial = awaitItem()
            assertFalse(initial.isLoading)
            viewModel.onEvent([YourFeature]UiEvent.Submit("input"))
            val loading = awaitItem()
            assertTrue(loading.isLoading)
            val result = awaitItem()
            assertFalse(result.isLoading)
            assertEquals(1, result.items.size)
        }
    }
}
```

### Compose UI 測試

```kotlin
@RunWith(AndroidJUnit4::class)
class [YourFeature]ScreenTest {
    @get:Rule val composeTestRule = createComposeRule()

    @Test fun `main input is focused on launch`() {
        composeTestRule.setContent { [YourFeature]Screen() }
        composeTestRule.onNodeWithTag("main_input")
            .assertIsFocused()
    }

    @Test fun `results list shows items after action`() {
        composeTestRule.setContent {
            [YourFeature]Content(
                state = [YourFeature]UiState(items = listOf([YourEntity].fixture())),
                onEvent = {}
            )
        }
        composeTestRule.onNodeWithText("[expected text]").assertIsDisplayed()
    }
}
```

### Fake Repository 模式

```kotlin
// 將 [YourApp] / [YourEntity] 替換為你的 App 實際名稱
class Fake[YourApp]Repository : [YourApp]Repository {
    private var items = emptyList<[YourEntity]>()
    private var error: Exception? = null

    fun setItems(list: List<[YourEntity]>) { items = list; error = null }
    fun setError(e: Exception) { error = e; items = emptyList() }

    override suspend fun fetch(query: String): Result<List<[YourEntity]>> =
        error?.let { Result.failure(it) } ?: Result.success(
            items.filter { it.title.contains(query, ignoreCase = true) }
        )
}
```

### 測試覆蓋率目標

```
核心業務邏輯（UseCase）：> 90%
ViewModel 狀態管理：> 80%
Repository：> 70%
UI 層：主要用戶流程的 E2E 測試（你的 App 核心動作）
```

---

## §3 子代理並行開發

### 核心思想

把一個大任務拆成多個獨立子任務，並行完成，比串行快 3-5x。

```
大任務：「完成 [YourFeature] 功能」
↓
Sub-1：[YourFeature]Screen UI（可獨立）
Sub-2：[YourFeature]ViewModel（可獨立）
Sub-3：[YourAction]UseCase（可獨立）
Sub-4：Repository 實現（依賴 Sub-3 接口，但接口可先定義）
Sub-5：Unit Tests（依賴 Sub-1-4 完成後）

Sub-1 + Sub-2 + Sub-3 並行執行 → Sub-4 → Sub-5
```

### 適合 / 不適合並行的任務

```
✅ 適合並行（互相獨立）：
• 同時寫 3 個不同 Screen 的 UI
• 同時寫 5 個 UseCase 的 unit test
• 同時分析 5 個競品的 UX
• 同時生成多種語言的翻譯文案

❌ 不適合並行（有依賴）：
• A 的輸出是 B 的輸入
• 需要共享狀態修改
• 順序很重要的步驟（如遷移）
```

### 高效 Prompt 模板（子任務）

```
✅ 高效（明確輸入/輸出/邊界）：
「用 KMP + Compose Multiplatform 實現 [YourFeature]Screen：
- [主要 UI 組件 1]（例：頂部輸入欄，自動聚焦）
- [主要 UI 組件 2]（例：結果列表，key = it.id）
- Empty / Loading / Error state

輸入接口：
  state: [YourFeature]UiState
  onEvent: ([YourFeature]UiEvent) -> Unit

不包含：ViewModel、業務邏輯（另一個任務負責）」

❌ 低效：「幫我做 [功能]」
```

### 批量任務分配

```markdown
## 並行任務包

任務 1（UI）：
  輸入：UiState 定義、設計稿描述
  輸出：[YourFeature]Screen.kt 完整 Compose 代碼

任務 2（ViewModel）：
  輸入：UiState / UiEvent 接口、UseCase 接口
  輸出：[YourFeature]ViewModel.kt

任務 3（UseCase + 測試）：
  輸入：Repository 接口、業務規則
  輸出：[YourAction]UseCase.kt + [YourAction]UseCaseTest.kt

依賴關係：任務 1 + 2 + 3 並行 → 任務 4（整合 + E2E 測試）
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 KMP / Coroutines / Turbine 文件 | Context7 |
| 讀寫本地代碼 / 測試文件 | Filesystem MCP |

## 自查清單

- [ ] Bug 已找到根本原因（不是只 patch symptom）？
- [ ] 修復有對應的 regression test？
- [ ] 每個 Unit Test 覆蓋 3 個場景（正常、邊界、失敗）？
- [ ] ViewModel 測試用了 Turbine？
- [ ] 並行任務已確認互相獨立（無共享狀態）？
