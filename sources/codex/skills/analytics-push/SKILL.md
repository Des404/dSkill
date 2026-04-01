---
name: analytics-push
description: >
  App 數據分析與 Push 通知設計實現顧問（Firebase Analytics + FCM + APNs），適用任何 Mobile App。
  觸發：Firebase Analytics、Mixpanel、數據追蹤、漏斗分析、留存率、DAU、MAU、
  Cohort 分析、A/B 測試、北極星指標、Push 通知、FCM、APNs、通知設計、
  Deep Link、個人化通知、通知時段、
  說「怎麼追蹤數據」「分析用戶行為」「留存率數據」「設置 Analytics」
  「推送通知怎麼做」「FCM 設置」「通知不到達」「通知文案怎麼寫」「Deep Link」。
  Do NOT use for: feature implementation, UI design, or tasks unrelated to analytics or push.
---

# Analytics + Push（數據分析 + 推送通知）

## 標準指令區塊

知識實時性：必須聯網搜索確認 Firebase SDK 和 FCM/APNs 最新版本。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用前：定義你的指標框架

> 在設置追蹤之前，先定義：
> - **北極星指標**：什麼行為代表用戶獲得了核心價值？
> - **留存目標**：你的 App 類型的合理 D1/D7/D30 目標是多少？
> - **關鍵漏斗**：用戶從安裝到留存的關鍵步驟是什麼？

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 設置 Firebase Analytics | §1 Analytics 設置 |
| 定義要追蹤的事件 | §2 事件追蹤框架 |
| 分析留存率 / 漏斗 | §3 指標分析 |
| 設置 FCM / APNs | §4 Push 設置 |
| 設計通知文案和策略 | §5 通知策略 |

---

## §1 Analytics 設置（KMP）

```kotlin
// commonMain — 定義事件接口
expect object Analytics {
    fun logEvent(name: String, params: Map<String, Any> = emptyMap())
    fun setUserId(userId: String?)
    fun setUserProperty(name: String, value: String?)
}

// androidMain
actual object Analytics {
    actual fun logEvent(name: String, params: Map<String, Any>) {
        val bundle = Bundle().apply {
            params.forEach { (key, value) ->
                when (value) {
                    is String -> putString(key, value)
                    is Int -> putInt(key, value)
                    is Long -> putLong(key, value)
                    is Double -> putDouble(key, value)
                }
            }
        }
        FirebaseAnalytics.getInstance(context).logEvent(name, bundle)
    }
    actual fun setUserId(userId: String?) = FirebaseAnalytics.getInstance(context).setUserId(userId)
    actual fun setUserProperty(name: String, value: String?) =
        FirebaseAnalytics.getInstance(context).setUserProperty(name, value)
}

// iosMain
actual object Analytics {
    actual fun logEvent(name: String, params: Map<String, Any>) {
        FIRAnalytics.logEventWithName(name, parameters = params as Map<Any?, *>)
    }
    actual fun setUserId(userId: String?) = FIRAnalytics.setUserID(userId)
    actual fun setUserProperty(name: String, value: String?) =
        FIRAnalytics.setUserPropertyString(value, forName = name)
}
```

---

## §2 事件追蹤框架

### 事件分類（根據你的 App 填入）

```kotlin
// 事件命名規範：snake_case，動詞_名詞
object AnalyticsEvents {

    // ── 核心功能事件（根據你的 App 核心行為定義）──
    // const val [CORE_ACTION] = "[core_action]"    // 每次完成核心操作
    // const val [VIEW_CONTENT] = "[view_content]"  // 查看主要內容
    // const val [SAVE_ITEM] = "[save_item]"        // 收藏/保存
    // const val [SHARE_ITEM] = "[share_item]"      // 分享

    // ── 用戶旅程事件（通用）──
    const val ONBOARDING_COMPLETED = "onboarding_completed"
    const val AHA_MOMENT = "aha_moment"            // 你定義的 Aha Moment
    const val APP_OPENED = "app_opened"

    // ── 增長事件 ──
    const val CONTENT_SHARED = "content_shared"    // K-factor 追蹤
    const val NOTIFICATION_OPENED = "notification_opened"
    const val STREAK_BROKEN = "streak_broken"      // 流失風險信號（如有 Streak）
    const val UPGRADE_VIEWED = "upgrade_viewed"    // 付費轉化追蹤
}
```

### 事件使用示例

```kotlin
// 核心動作追蹤
Analytics.logEvent(AnalyticsEvents.AHA_MOMENT, mapOf(
    "step" to "first_core_action",
    "session_number" to sessionNumber,
    "time_to_aha_seconds" to timeElapsed
))

// 增長事件追蹤
Analytics.logEvent(AnalyticsEvents.CONTENT_SHARED, mapOf(
    "content_type" to contentType,
    "channel" to "whatsapp"  // instagram / twitter / copy_link / etc.
))

// 用戶屬性（用於分群）
Analytics.setUserProperty("user_segment", "power_user")  // casual / regular / power
Analytics.setUserProperty("subscription_status", "free")   // free / trial / paid
```

---

## §3 指標分析

### 北極星指標定義

```
[你的北極星] = 每日完成[你的核心動作]的次數

為什麼選這個指標：
• 代表用戶獲得了核心價值
• 與 DAU 和收入直接相關
• 可分解為更小的指標

目標：每月增長 [X]%
```

### 留存率目標（行業基準）

```
不同 App 類型的 D30 留存基準：
• 工具類（實用工具）：10-15%
• 社交類：15-25%
• 遊戲類：10-20%
• 媒體/娛樂：15-25%
• 健康/健身：15-20%

設定你的目標（基於行業基準調整）：
D1 > [X]%  D7 > [Y]%  D30 > [Z]%
```

### 留存率診斷

```
D1 差（< 目標 × 0.7）
→ Onboarding 有問題，Aha Moment 沒有發生
→ 改善：降低首次使用門檻、加快到達 Aha Moment

D1 好但 D7 差
→ 沒有理由回來，沒有建立習慣
→ 改善：Push 通知、Streak 機制、每日新內容

D7 好但 D30 差
→ 新鮮感消退
→ 改善：個人化、社交功能、進度系統
```

### Firebase Console 必看報告

```
Engagement → Events：確認事件觸發正確
Retention → Cohort Analysis：追蹤 D1/D7/D30
Analytics → Funnels：Onboarding 漏斗
Crashlytics → Crash-free users：> 99.5% 目標
```

### A/B 測試（Firebase Remote Config）

```kotlin
val remoteConfig = FirebaseRemoteConfig.getInstance()
remoteConfig.fetchAndActivate().addOnCompleteListener {
    val variant = remoteConfig.getString("onboarding_variant")
    // "A": [版本A描述]
    // "B": [版本B描述]
    showOnboarding(variant)
}
```

---

## §4 Push 通知設置（FCM + APNs）

### KMP 設置

```kotlin
// commonMain — 接口
expect class PushNotificationManager {
    fun requestPermission(onResult: (Boolean) -> Unit)
    fun getToken(onToken: (String) -> Unit)
}

// androidMain — FCM
actual class PushNotificationManager(private val context: Context) {
    actual fun requestPermission(onResult: (Boolean) -> Unit) {
        if (Build.VERSION.SDK_INT >= 33) {
            ActivityCompat.requestPermissions(activity,
                arrayOf(Manifest.permission.POST_NOTIFICATIONS), REQUEST_CODE)
        } else { onResult(true) }
    }
    actual fun getToken(onToken: (String) -> Unit) {
        FirebaseMessaging.getInstance().token.addOnSuccessListener { onToken(it) }
    }
}

// iosMain — APNs
actual class PushNotificationManager {
    actual fun requestPermission(onResult: (Boolean) -> Unit) {
        UNUserNotificationCenter.currentNotificationCenter()
            .requestAuthorizationWithOptions(
                UNAuthorizationOptionAlert or UNAuthorizationOptionSound or UNAuthorizationOptionBadge
            ) { granted, _ -> onResult(granted) }
    }
}
```

### Firebase Console 設置步驟

```
1. Firebase Console → 你的 App → Cloud Messaging
2. Android：google-services.json 放 androidApp/
3. iOS：GoogleService-Info.plist 放 iosApp/ + APNs 憑證上傳
4. 測試：Console → 發送測試通知 → 輸入設備 FCM token
```

---

## §5 通知策略

### 通知類型設計（根據你的 App 調整）

| 通知目的 | 文案風格 | 最佳時段 | 觸發條件 |
|---------|---------|---------|---------|
| 習慣建立 | 「[用戶動作提醒]」 | 用戶最活躍時段 | 每日 |
| 進度提醒 | 「你的[進度]快要[狀態]了！」 | 離截止前幾小時 | 條件觸發 |
| 里程碑 | 「你達成了[成就]！🎉」 | 即時 | 達成時 |
| 重新激活 | 「好久不見！[召回原因]」 | 下午 2-4 時 | N天未回訪 |
| 新內容 | 「[新內容名稱]已上架」 | 發布時 | 新內容發布 |

### 通知文案原則

```
✅ 具體：告訴用戶通知裡有什麼
✅ 個人化：用用戶的數據（「你的第 12 天」）
✅ 可操作：點擊後直接到相關頁面（Deep Link）
✅ 有節制：每天 ≤ 1 條（工具類）

❌ 模糊：「今天有新內容等你」
❌ 頻繁：多於用戶期望的頻率
❌ 錯誤時段：22:00-8:00（睡眠時段）
```

### 通知權限請求時機

```kotlin
// ✅ 在 Aha Moment 之後才請求（不是第一次打開）
if (ahaMomentReached && !permissionRequested) {
    // 先解釋通知的好處，再請求
    showNotificationBenefitDialog {
        pushManager.requestPermission { granted ->
            Analytics.logEvent("notification_permission",
                mapOf("granted" to granted, "session" to sessionNumber))
        }
    }
}
```

### Deep Link 設置

```kotlin
// Android — AndroidManifest.xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW"/>
    <category android:name="android.intent.category.DEFAULT"/>
    <data android:scheme="https" android:host="yourapp.com" android:pathPrefix="/[feature]/"/>
</intent-filter>

// 通知 payload
{
    "notification": { "title": "[標題]", "body": "[內容]" },
    "data": { "deepLink": "https://yourapp.com/[feature]/[id]" }
}
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 Firebase Analytics / FCM 官方文件 | Context7 |

## 自查清單

- [ ] 北極星指標已定義（什麼行為=用戶獲得價值）？
- [ ] 核心事件已追蹤（Aha Moment、核心動作、分享）？
- [ ] D1/D7/D30 目標已設定，在 Firebase 可追蹤？
- [ ] Push 通知在 Aha Moment 後才請求權限？
- [ ] 通知文案具體 + 有 Deep Link？
