# 東南亞本地化規範（SEA Locale）

載入條件：「東南亞」「SEA」「Southeast Asia」「新加坡」「馬來西亞」「泰國」「菲律賓」「越南」「印尼」

## 語言規範（多語言市場）
| 國家 | 主要語言 | 字體推薦 |
|------|---------|---------|
| 新加坡 | 英文（主）+ 中文/馬來文/泰米爾文 | Inter + Noto Sans SC |
| 馬來西亞 | 英文 + 馬來文 + 中文 | Inter + Noto Sans |
| 泰國 | 泰文 | Noto Sans Thai |
| 菲律賓 | 英文 + 他加祿語 | Inter |
| 越南 | 越南文 | Noto Sans / Be Vietnam Pro |
| 印尼 | 印尼文 | Inter + Noto Sans |

## 主流平台（2025）
Facebook（主導，尤其菲律賓/印尼）· TikTok（極高增長，整個 SEA）· Instagram · WhatsApp · Telegram · LINE（泰國/台灣）

## 支付方式（按國家）
| 國家 | 主要支付 |
|------|---------|
| 新加坡 | PayNow · GrabPay · PayLah! · Apple Pay |
| 馬來西亞 | Touch 'n Go eWallet · GrabPay · FPX |
| 泰國 | PromptPay · TrueMoney · GrabPay |
| 菲律賓 | GCash · Maya · GrabPay |
| 越南 | MoMo · ZaloPay · VNPay |
| 印尼 | GoPay · OVO · DANA · ShopeePay |
| 全區通用 | COD（貨到付款）— 非常重要 |

## 分享渠道優先
① WhatsApp/LINE（因國家而異）② Facebook ③ TikTok ④ Instagram

## 市場特點
- 手機使用時間極高（菲律賓 5.5h/天，全球前列）
- 低端 Android 設備多（需優化 APK < 50MB）
- 數據流量成本敏感（圖片/影片需壓縮，支援低速網絡）
- Facebook 在菲律賓/印尼仍是主導社交平台
- 電商直播非常流行（尤其 TikTok Shop）
- COD 是重要支付選項（信用卡普及率低）

## UX 差異
- 本地化文案比機器翻譯重要（用母語說話更親切）
- 節日行銷機會多（齋戒月 Ramadan、宋干節、聖誕節等）
- App 大小是下載決策關鍵（老舊設備存儲空間緊）
- 用戶接受廣告密度高
- 遊戲化元素效果好（Points/Badges/Leaderboard）

## 技術注意
- 右到左（RTL）語言：馬來西亞/印尼有阿拉伯字符場景
- 泰文/越南文字符：需要正確的 Unicode 字體支援
- 低速網絡優化：圖片 WebP 格式，Progressive Loading

## 法規
PDPA（泰國/新加坡）· PDPL（印尼）· 各國數據本地化要求不一
