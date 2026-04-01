---
name: commit-code
description: >
  Git 工作流、代碼文件生成與 CI/CD 自動化一站式顧問。
  觸發：git、commit、branch、merge、pull request、rebase、衝突解決、
  commit message、Conventional Commits、PR 描述、changelog、What's New、
  KDoc、README、API 文件、架構說明、CI/CD、GitHub Actions、Fastlane、
  自動打包、TestFlight、Play Console、自動發布、
  說「怎麼提交」「commit 怎麼寫」「PR 流程」「git 衝突」「branch 策略」
  「寫文件」「加 KDoc」「README 怎麼寫」「怎麼自動化發布」「CI 設置」。
  Do NOT use for: feature development, bug fixing, code review, or tasks unrelated to git/docs/CI.
---

# Commit & Code（Git + 文件 + CI/CD）

## 標準指令區塊

知識實時性：必須聯網搜索確認 GitHub Actions / Fastlane 最新語法和版本。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| Git 操作 / Commit / Branch / PR | §1 Git 工作流 |
| 寫 Changelog / What's New | §2 更新說明 |
| 寫 KDoc / README | §3 代碼文件 |
| 設置 CI/CD 自動化 | §4 CI/CD |

---

## §1 Git 工作流

### Branch 策略

```
main          ← 永遠可發布，對應 App Store 版本
feature/*     ← 每個功能一個 branch
fix/*         ← bug fix
release/*     ← 發版準備
hotfix/*      ← 緊急修復（直接從 main 開）
```

```bash
# 開始新功能
git checkout main && git pull
git checkout -b feature/offline-search

# 每天至少一個有意義的 commit
git add [具體文件]
git commit -m "feat(search): add offline query support"

# 完成後 rebase（保持乾淨歷史）
git fetch origin
git rebase origin/main
git push origin feature/offline-search
```

### Commit Message（Conventional Commits）

```
格式：<type>(<scope>): <subject>

type：
  feat      新功能（觸發 minor 版本號遞增）
  fix       Bug 修復（觸發 patch 版本號遞增）
  refactor  重構（不改行為）
  test      測試
  docs      文件
  perf      性能改善
  chore     構建/工具/依賴
  style     格式（不影響邏輯）

scope（KMP 項目，根據你的 App 功能定義）：
  [main-feature] / [secondary-feature] / auth / ui / db / api

示例（將功能名稱替換為你的 App）：
  feat([feature]): add offline support with SQLDelight cache
  fix(ui): resolve input bar focus lost on Android 12
  refactor(viewmodel): extract [YourAction]UseCase from [YourFeature]ViewModel
  test(usecase): add unit tests for [YourAction]UseCase edge cases
  perf(lazycolumn): add stable keys to prevent unnecessary recomposition
  chore(deps): update Kotlin to 2.2.20 and Compose to 1.10.3

❌ 不好的：
  "fix bug"
  "update"
  "wip"
```

### 衝突解決

```bash
# rebase 遇到衝突
git status                     # 看哪些文件有衝突
# 打開文件，找 <<<<<<< HEAD 手動解決
git add [解決了的文件]
git rebase --continue
git rebase --abort             # 如果搞不定，放棄

# Squash WIP commits（merge 前整理）
git rebase -i main
# 把 "wip" 和 "fix typo" 的 commit squash 成有意義的 commit
```

### PR 前 Checklist

```
□ 沒有 println / NSLog（改用 Timber / 條件日誌）
□ 沒有 hardcoded 字串
□ 沒有 unused imports / dead code
□ 新代碼有 unit test
□ 所有測試通過（./gradlew test）
□ Android + iOS 模擬器測試通過
```

### PR 描述模板

```markdown
## 這個 PR 做了什麼？
[一句話說明這個 PR 的目的]

## 變更內容
- [ ] [具體變更 1]
- [ ] [具體變更 2]

## 如何測試
1. [測試步驟 1]
2. [測試步驟 2]

## 截圖（UI 改動時）
[Before/After 截圖]

## 相關 Issue
Closes #[Issue 號碼]
```

---

## §2 更新說明（Changelog / What's New）

### App Store What's New 寫法原則

```
✅ 用戶能理解的語言（不用技術術語）
✅ 關注用戶受益（「你現在可以...」「更快...」）
✅ 具體描述（「搜索速度提升 30%」比「改善性能」好）
✅ iOS：≤ 4000 字符，優先寫最重要的 1-3 點
✅ Android What's New：≤ 500 字符（極度精簡）

示例（根據你的 App 功能填寫）：
iOS：
「新增功能：
• [核心新功能]：[用戶能看懂的一句話說明]
• [次要新功能]：[說明]
• 更快的 [主要操作]：[操作] 速度提升 X%，現在離線也可以即時使用

修復：
• 修復在 [設備] 上 [問題] 的問題」

Android（極簡，≤500 字符）：
「新增 [功能1]、[功能2]，[操作] 速度大幅提升，修復多個問題。」
```

### CHANGELOG.md 格式

```markdown
# Changelog

## [1.2.0] — YYYY-MM-DD
### Added
- [新功能 1]（描述用戶受益，非技術細節）
- [新功能 2]
- [新功能 3]

### Changed
- [改進 1]，速度提升 X%
- [改進 2]

### Fixed
- 修復 [設備/場景] 上 [問題] 的問題（#42）
- 修復 Android 12 首次安裝 [問題]（#38）

## [1.1.0] — YYYY-MM-DD
...
```

---

## §3 代碼文件（KDoc + README）

### KDoc 規範

```kotlin
/**
 * [說明這個 UseCase 的用途，一句話]。
 *
 * [輸入驗證規則或邊界條件說明]。
 * [緩存/網絡策略說明]。
 *
 * @param input [輸入參數說明]
 * @param filter [可選過濾參數說明]（默認 [默認值]）
 * @param limit 最大返回數量（默認 20）
 * @return [Result] 成功包含 [[YourEntity]] 列表
 *
 * @see Get[YourEntity]DetailUseCase
 * @since 1.0.0
 */
class [YourAction]UseCase(private val repository: [YourApp]Repository) {
    suspend operator fun invoke(
        input: String,
        filter: String = "all",
        limit: Int = 20
    ): Result<List<[YourEntity]>>
}
```

**需要 KDoc 的：** 所有 public class/interface、所有 public function（非 override）、複雜業務邏輯、expect/actual

**不需要：** 私有方法（除非複雜）、自解釋代碼、override 方法

### README 模板

````markdown
# [YourApp]

[一句話描述你的 App 是什麼、為誰做、解決什麼問題]。
KMP + Compose Multiplatform，支持 iOS + Android。

## 技術棧
- Kotlin Multiplatform（共用業務邏輯）
- Compose Multiplatform 1.10.3（跨平台 UI）
- Ktor 3.x（網絡）
- SQLDelight 2.x（本地數據庫）
- Koin 4.x（依賴注入）

## 環境要求
- Kotlin 2.2.20+
- Android Studio Meerkat+
- Xcode 15+（iOS 開發需要 macOS）
- JDK 17

## 運行

```bash
# Android
./gradlew :androidApp:installDebug

# iOS（需要 macOS）
cd iosApp && xed .
```

## 架構
Clean Architecture + MVVM，詳見 `docs/adr/`
````

---

## §4 CI/CD 自動化

### GitHub Actions（KMP）

```yaml
# .github/workflows/ci.yml
name: CI
on:
  push: { branches: [main, develop] }
  pull_request: { branches: [main] }

jobs:
  test:
    runs-on: macos-latest  # KMP iOS 需要 macOS
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with: { java-version: '17', distribution: 'zulu' }
      - name: Cache Gradle
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle.kts') }}
      - name: Run Tests
        run: ./gradlew test --continue
      - name: Build Android
        run: ./gradlew :androidApp:assembleRelease
      - name: Build iOS (KMP framework)
        run: ./gradlew :shared:linkReleaseFrameworkIosArm64
```

### Fastlane iOS 發布

```ruby
# fastlane/Fastfile
platform :ios do
  lane :beta do
    increment_build_number(xcodeproj: "iosApp/iosApp.xcodeproj")
    build_app(
      scheme: "iosApp",
      workspace: "iosApp/iosApp.xcworkspace",
      export_method: "app-store"
    )
    upload_to_testflight(
      api_key_path: "fastlane/api_key.json",
      skip_waiting_for_build_processing: true
    )
  end
end
```

### GitHub Secrets 清單

```
Android：
  KEYSTORE_BASE64     ← base64 編碼的 keystore
  KEYSTORE_PASSWORD   ← keystore 密碼
  KEY_ALIAS / KEY_PASSWORD
  PLAY_STORE_JSON     ← Play Console service account JSON

iOS：
  APP_STORE_CONNECT_API_KEY_ID
  APP_STORE_CONNECT_ISSUER_ID
  APP_STORE_CONNECT_API_KEY   ← .p8 文件內容
  APPLE_TEAM_ID
```

### 版本號自動化

```kotlin
// build.gradle.kts
val versionCode = System.getenv("GITHUB_RUN_NUMBER")?.toInt() ?: 1
val versionName = "1.2.${System.getenv("GITHUB_RUN_NUMBER") ?: "0"}"
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 GitHub Actions / Fastlane 文件 | Context7 |
| 讀寫本地配置文件 / Fastfile | Filesystem MCP |

## 自查清單

- [ ] Commit message 符合 Conventional Commits 格式？
- [ ] PR 描述有測試步驟？
- [ ] What's New 用戶能看懂（無技術術語）？
- [ ] KDoc 覆蓋所有 public API？
- [ ] GitHub Secrets 已設置（無明文 key）？
- [ ] CI 在 PR 上通過才能 merge？
