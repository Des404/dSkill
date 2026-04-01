# 日本本地化規範（JP Locale）

載入條件：「日本」「JP」「日本用戶」「Japanese market」

## 語言規範
- 日文（ひらがな + 漢字 + カタカナ）
- 字體：Hiragino Sans（iOS 系統字體）· Noto Sans JP（Android）
- Google Fonts：`@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@400;700&display=swap');`
- 正文 ≥ 15px，行距 1.8（日文行距需更大）
- 注意：日文換行規則（禁則処理）—— 標點不能在行首

## 主流平台（2025）
LINE（最重要，日常通訊）· Twitter/X（日本是全球 X 最活躍市場之一）· Instagram · YouTube · TikTok

## 支付方式
PayPay（最普及，~60% 普及率）· LINE Pay · d払い · 楽天ペイ · Apple Pay · 信用卡 · コンビニ支払い（便利店）

## 分享渠道優先
① LINE ② Twitter/X（日本 X 使用率特別高）③ Instagram ④ 複製連結

## 視覺偏好
- 精緻、細節感強（日式設計美學——余白、整齊）
- 可愛（Kawaii）風格有廣泛接受度
- 粉色/柔和色調受歡迎
- 信息密度中等（比中國大陸少，比美國多）
- 動畫精緻（日本用戶對動畫質量要求高）

## UX 差異
- 用戶容忍度極低——有 bug 等於 1 星評分
- 文字說明詳細（用戶習慣閱讀完整說明書）
- 敬語 vs 常體：需根據 App 性質選擇（銀行/醫療用敬語）
- 用戶重視隱私，不喜歡過多廣告
- 年長用戶比例高，需要較大字體選項

## 技術注意
- 日文換行：`word-break: keep-all` 或使用 CSS `overflow-wrap`
- 直排文字（縦書き）：電子書/新聞 App 可能需要
- 字元計數：日文一字 = 兩個英文字元寬度（排版計算要調整）

## 法規
個人情報保護法（APPI）
