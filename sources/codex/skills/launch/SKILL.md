---
name: launch
description: >
  App Store + Play Store 提交審核與 ASO 優化全棧顧問。
  觸發：提交 App、App Store 審核、Play Console、被拒原因、截圖要求、隱私政策、
  ASO、App Store 排名、關鍵字優化、截圖設計、下載率、Apple Search Ads、
  評分請求、What's New 寫法、上架、Store 元數據、
  說「怎麼提交 App」「審核被拒」「App Store 截圖」「上架」「元數據」
  「怎麼提高下載量」「ASO」「關鍵字」「搜索排名」「截圖怎麼設計」。
  Do NOT use for: feature development, analytics setup, CI/CD automation, push notifications.
---

# Launch（App Store 提交 + ASO 優化）

## 標準指令區塊

知識實時性：App Store 審核規則經常更新，必須先聯網搜索確認最新要求。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 第一次上架 / 提交審核 | §1 提交 Checklist |
| 被拒了怎麼辦 | §2 被拒原因和解法 |
| 提高搜索排名和下載量 | §3 ASO 優化 |
| 截圖設計 | §4 截圖策略 |

---

## §1 提交 Checklist

### App Store Connect（iOS）

```
App 資訊：
□ App 名稱：≤ 30 字符（含最重要的關鍵字）
□ 副標題：≤ 30 字符（補充關鍵字）
□ 關鍵字：≤ 100 字符，逗號分隔，不重複標題已有詞
□ 描述：≤ 4000 字符（前 3 行最重要，展開前的文字）
□ 宣傳文字：≤ 170 字符（可隨時更新，不需重新審核）

視覺資產：
□ App 圖標：1024×1024px，無透明、無圓角、無透明度
□ 截圖：iPhone 6.7"（必須）、6.5"（可選）、5.5"（舊設備）
□ iPad 截圖：12.9"（如果支持 iPad）
□ 預覽影片（強烈推薦，轉化率 +20-30%）

合規：
□ 隱私政策 URL（必須，且頁面必須可以訪問）
□ 支持 URL
□ 年齡分級問卷已完成
□ 加密聲明（使用 HTTPS 需要申報）
□ 版本號和 Build Number 已遞增
□ 測試賬號（如果有登入功能）
```

### Play Console（Android）

```
商店資訊：
□ 標題：≤ 30 字符
□ 簡短說明：≤ 80 字符
□ 完整說明：≤ 4000 字符
□ 應用圖標：512×512px
□ 宣傳圖片：1024×500px（Feature Graphic）
□ 截圖：最少 2 張，最多 8 張（手機）

合規：
□ 隱私政策 URL
□ 內容分級問卷已完成
□ 數據安全表格已填（說明收集什麼數據）
□ targetSdkVersion 是 Google 要求的最新版本
□ 64位 APK / AAB
□ 上傳 AAB（不是 APK，Google 要求）
```

---

## §2 被拒原因和解法

### iOS 最常見被拒

| 被拒原因 | 解法 |
|---------|------|
| 4.3 Spam（功能太少）| 加更多核心功能，確認 App 有獨特價值 |
| 2.1 信息不足 | 提供詳細說明，加截圖演示功能 |
| 5.1.1 隱私政策缺失 | 添加隱私政策頁面和 URL |
| 1.0 App Crash | 修復崩潰，提供測試賬號，說明復現步驟 |
| 3.1.1 未使用 IAP | 如果有付費內容，必須用 Apple IAP |
| 2.3.3 功能不符描述 | 截圖和描述要準確反映 App 實際功能 |
| 5.1.2 數據收集 | 在 App 內說明收集用途，更新隱私政策 |

### Android 最常見被拒

| 被拒原因 | 解法 |
|---------|------|
| 欺騙性行為 | 確認截圖和描述真實 |
| 版權問題 | 確認 App 使用的第三方內容授權符合分發條款 |
| 數據安全表格不完整 | 補充填寫所有收集的數據類型 |
| targetSdkVersion 過低 | 更新到 Google 要求的最低版本 |

### 被拒後的回應策略

```
1. 仔細閱讀被拒原因（往往有具體 guideline 編號）
2. 查看 App Store Review Guidelines 對應條款
3. 修改 App，說明修改了什麼
4. 在回應中引用 guideline，說明你如何符合
5. 如果有誤解，可以 Appeal（但要有理有據）

回應模板：
「感謝您的審查。我們已按照 [Guideline X.X] 的要求進行了以下修改：
1. [具體修改 1]
2. [具體修改 2]

[如果需要]，以下是我們的測試賬號：
Email: test@example.com / Password: Test1234」
```

---

## §3 ASO 優化

### ASO 核心要素（影響力排序）

```
1. App 名稱（最重要，含核心關鍵字）
2. 截圖（影響 70% 的下載決定）
3. 副標題 / 簡短說明
4. 關鍵字欄位（iOS，100 字符）
5. 評分和評論數量
6. 預覽影片
7. 更新頻率
```

### 關鍵字策略

```
目標關鍵字類型：
• 高意圖詞：你的類別最核心的詞（搜索量高，競爭大）
• 長尾詞：加上場景/特性修飾的詞（搜索量低，競爭小，轉化率高）
• 競品名稱：⚠️ Apple 不允許直接用競品名稱

iOS 關鍵字（100 字符，逗號分隔，不要空格）：
[根據你的 App 類型填寫：功能詞、使用場景詞、目標用戶詞、差異化特性詞]
例：「離線,快速,[核心功能],[目標用戶類型],[特色功能]」

監控：App Store Connect → Analytics → 搜索詞
      每月更新關鍵字（根據哪些詞帶來下載）
```

### 評分策略

```kotlin
// iOS — 在 Aha Moment 後請求評分（用戶完成核心動作一定次數後）
// 將 [actionCount] 替換為你的 App 核心動作計數器
if (actionCount == 10 || actionCount == 50 || actionCount == 100) {
    SKStoreReviewController.requestReview()  // 系統 API，不能手動觸發彈窗
}

// Android — 用 Google Play In-App Review API
val reviewManager = ReviewManagerFactory.create(context)
reviewManager.requestReviewFlow().addOnCompleteListener { request ->
    if (request.isSuccessful) {
        val reviewInfo = request.result
        reviewManager.launchReviewFlow(activity, reviewInfo)
    }
}

// 最佳時機：
// • 完成一個核心動作後（你的 App 最重要的用戶動作）
// • 連續使用 7 天後
// • 更新後首次開啟
// 避免：App 啟動時立即請求、錯誤發生後請求
```

### Apple Search Ads（付費獲客）

```
入門策略（有限預算）：
1. 開通 ASA Basic（自動關鍵字，適合入門）
2. 目標：品牌詞（自己 App 名稱）→ 防止競品截流
3. 月預算：根據你的 LTV 估算（建議起步 CPA ≤ 30天留存用戶的 LTV）

指標監控：
• CPT（每次點擊成本）：參考同類 App 業界基準
• CR（點擊轉化率）：目標 > 50%
• CPA（每次安裝成本）：目標 < 你的 30 天用戶 LTV

Advanced 策略（有一定規模後）：
• 自定義關鍵字組
• Broad Match + Exact Match 組合
• 排除無效關鍵字
```

---

## §4 截圖策略

### 截圖設計原則

```
第一張截圖：最重要（決定 70% 的下載）
• 3 秒內傳達核心價值
• 大字標題（字體 ≥ 40pt）
• 配合 App 主色調
• 展示最吸引人的核心功能

後續截圖：
• 每張展示一個功能
• 文字說明用戶受益（不是功能描述）
• 順序：核心功能 → 差異化功能 → 次要功能

✅ 截圖文字原則：
「[速度/便利性的用戶受益]」（展示核心功能）
「[感官體驗的受益]」（展示差異化功能）
「[場景受益]」（展示離線/隨時隨地）
「[習慣養成受益]」（展示每日功能/通知）

✅ 文字格式：動詞開頭、用戶視角、聚焦受益而非功能名稱
❌ 不好的文字：直接說功能名稱（「搜索功能」「離線模式」）
```

### 尺寸規格

```
iOS：
  iPhone 6.7"（iPhone 15 Pro Max）：1320×2868px 必須
  iPhone 6.5"（iPhone 14 Plus）：1242×2688px（可選）
  iPad Pro 12.9"（M4）：2048×2732px（如果支持 iPad）

Android：
  手機：最小 320×320px，最大 3840×3840px，長寬比 2:1
  平板：與手機分開上傳
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查最新 App Store Review Guidelines | Context7 |

## 自查清單

- [ ] 已聯網確認最新 App Store 審核要求？
- [ ] 截圖第一張 3 秒內傳達核心價值？
- [ ] 關鍵字欄位 ≤ 100 字符，無重複？
- [ ] 隱私政策 URL 有效可訪問？
- [ ] Android What's New ≤ 500 字符？
- [ ] 版本號已遞增？
- [ ] 有測試賬號（如有登入功能）？
