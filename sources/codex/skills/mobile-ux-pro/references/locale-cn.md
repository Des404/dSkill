# 中國大陸本地化規範（CN Locale）

載入條件：「中國大陸」「CN」「簡體中文」「國內市場」「大陸用戶」

## 語言規範
- 簡體中文（不是繁體）
- 字體：PingFang SC（iOS）· Source Han Sans CN / Noto Sans SC（Android）
- 正文 ≥ 16px，行距 1.7-1.8

## 主流平台（2025）
微信（WeChat，超級 App）· 微博 · 抖音（TikTok 國內版）· 小紅書 · B站 · QQ（年輕人）

## 支付方式（缺一不可）
- 微信支付（必須）
- 支付寶（必須）
- Apple Pay（次要）
- 雲閃付（銀聯）

## 分享渠道優先
① 微信朋友圈 ② 微信私信/群 ③ 微博 ④ 小紅書 ⑤ QQ

## 合規要求（非常重要）
- ICP 備案（必須）
- 用戶數據必須存儲於境內服務器（等保合規）
- 《個人信息保護法》（PIPL）
- 未成年人保護：實名制、使用時長限制
- Google 服務在大陸不可用（Maps/Fonts/Firebase/Analytics 全部需替換）

## Google 服務替換方案
| Google 服務 | 替代方案 |
|------------|---------|
| Google Maps | 高德地圖 SDK / 百度地圖 SDK |
| Google Fonts | 本地字體 / jsDelivr CDN |
| Firebase | 騰訊雲 / 阿里雲 |
| Google Analytics | 神策數據 / GrowingIO |
| Google Pay | 微信支付 / 支付寶 |
| FCM 推送 | 小米推送 / 華為推送 / 個推 |

## 視覺偏好
- 超級 App 思維：功能多合一，用戶習慣複雜 UI
- 紅色 = 吉祥，大量用於促銷/按鈕/重要元素
- 資訊密度高（用戶接受比西方 App 更密集的排版）
- 直播帶貨 UI 元素（彈幕/購物車浮動按鈕）

## 特有功能需求
- 手機使用時間 7.8h/天，廣告機會極多
- 小程式（Mini Program）生態：優先考慮微信小程式版本
- 掃碼支付場景（線下融合）
- 人臉識別登入（用戶接受度高）
