# 99 UX 規則 + 香港用家規範

格式：`規則ID | 規則名 | 說明 | 來源 | 違反後果`

---

## TOUCH 觸控互動（T01–T12）

T01 | touch-target-ios | 所有可點擊元素 ≥ 44×44pt | HIG | 用戶點擊失敗，操作挫敗感
T02 | touch-target-android | 所有可點擊元素 ≥ 48×48dp | MD | 點擊不中，特別在小螢幕上
T03 | thumb-zone | 核心操作放底部 60% 螢幕（拇指熱區）| UX Research | 用戶需要換手才能操作
T04 | swipe-natural | 滑動方向符合直覺：左右換頁，上下滾動 | HIG/MD | 用戶誤觸，反應混亂
T05 | long-press-menu | 長按顯示 Context Menu（香港用戶已習慣 WhatsApp 模式）| HIG | 功能不可見，用戶放棄
T06 | haptic-feedback | 重要操作（刪除、確認、切換）配合觸感反饋 | HIG | 操作感空洞，缺乏確認感
T07 | drag-handle | 可拖動元素需要明確的視覺把手 | MD | 用戶不知道可以拖動
T08 | gesture-discoverability | 手勢操作必須有視覺暗示（邊緣陰影/箭頭）| HIG | 功能隱藏，用戶找不到
T09 | tap-feedback | 每個可點擊元素需要視覺反饋（0.1s 內）| MD | 用戶不確定有沒有點到
T10 | double-tap | 雙擊功能需要 Tooltip 或首次使用提示 | UX | 用戶不知道雙擊可以做什麼
T11 | swipe-to-delete | 滑動刪除需要確認步驟（防誤觸）| HIG | 用戶意外刪除內容
T12 | pull-to-refresh | 下拉刷新需要明確的動畫和觸感反饋 | MD | 用戶不知道有沒有刷新

---

## NAVIGATION 導航（N01–N12）

N01 | tab-bar-max | iOS Tab Bar 最多 5 個 Tab | HIG | 圖示擁擠，標籤不可見
N02 | bottom-nav-android | Android Bottom Navigation ≤ 5 個 | MD | 導航混亂，分不清層級
N03 | back-nav | iOS 必須支援左滑返回；Android 必須支援系統返回鍵 | HIG/MD | 用戶卡住，無法返回
N04 | active-state | 當前頁面的 Tab 必須有清楚的選中狀態 | HIG/MD | 用戶不知道自己在哪裡
N05 | deep-link | 通知和分享連結必須 Deep Link 到對應頁面 | UX | 用戶跳轉後迷失在主頁
N06 | breadcrumb | 超過 3 層的層級結構需要導航提示 | UX | 用戶不知道怎麼回去
N07 | empty-state | 空頁面必須說明原因 + 引導動作 | HIG/MD | 用戶以為 App 壞了
N08 | loading-state | 所有異步操作需要 Loading 動畫（≤200ms 可省略）| HIG/MD | 用戶不確定 App 是否在運作
N09 | error-state | 網絡錯誤需要具體說明 + 重試按鈕 | HIG/MD | 用戶看到空白不知所措
N10 | nav-hierarchy | 功能層級 ≤ 3 層，超過需要重新架構 | UX | 用戶迷路，找不到功能
N11 | search-prominence | 常用搜索功能放在頂部（香港用戶習慣搜索優先）| HK UX | 用戶滾動找功能，效率低
N12 | onboarding-progress | Onboarding 必須顯示進度（「第 2/4 步」）| UX | 用戶不知道還要多久，中途放棄

---

## TYPOGRAPHY 排版（Ty01–Ty10）

Ty01 | body-min-size | 正文最小 16px（香港長者用戶多，建議 17-18px）| HK UX | 閱讀困難，用戶放大字體破版
Ty02 | cjk-line-height | 繁體中文行距 ≥ 1.7（英文 1.5 不夠）| HK UX | 中文行距太密，閱讀吃力
Ty03 | contrast-aa | 文字對背景對比度 ≥ 4.5:1（WCAG AA）| WCAG | 視力較差用戶無法閱讀
Ty04 | font-hierarchy | 標題/正文/說明至少 3 個字重層級（700/400/300）| HIG | 頁面無視覺重點，一片混亂
Ty05 | cjk-no-truncate | 繁體中文盡量避免截斷（字符截斷比英文難看）| HK UX | 截斷位置怪，文字意思不完整
Ty06 | line-max-width | 正文每行最大 70 字符（中文約 35 字）| UX | 行太長，眼睛跟蹤困難
Ty07 | dynamic-type | iOS 必須支援 Dynamic Type（用戶調字大小）| HIG | 調大字體後 UI 破版
Ty08 | numeric-tabular | 數字用等寬字體（金額、計時器）| HIG/MD | 數字跳動，排版不整齊
Ty09 | weight-semantic | 不同字重有語義意義（粗=重要，細=說明）| UX | 用戶無法快速掃描重點
Ty10 | font-system | CJK 字體必須指定 Noto Sans HK / Noto Sans TC 作備選 | HK UX | Android 回退字體難看

---

## COLOR 色彩（C01–C10）

C01 | color-semantic | 定義語義色彩 Token（不直接用 hex 寫在組件）| MD | 改主題色要改幾十個地方
C02 | dark-mode | 暗色模式用降飽和色，不是直接反色 | HIG/MD | 暗色下顏色刺眼
C03 | color-blindness | 功能性色彩（成功/錯誤）不能只靠顏色，需配圖示 | WCAG | 色盲用戶無法分辨
C04 | brand-3-max | 品牌色最多 3 種，避免視覺雜亂 | UX | 頁面色彩混亂，無品牌感
C05 | red-hk | 香港用戶：紅色兩義（警告 + 優惠/好彩），需要情境配合 | HK UX | 用戶看到紅色以為是錯誤，其實是促銷
C06 | surface-shadow | 卡片/表面用 elevation（陰影/色差）而非邊框 | MD | 邊框太多，頁面擁擠
C07 | state-colors | 按鈕需要 Default/Hover/Pressed/Disabled 四個狀態色 | HIG/MD | 用戶不確定按鈕有沒有點到
C08 | gradient-use | Gradient 只用於 Hero/強調元素，不用於功能性 UI | UX | 漸變過多，頁面廉價感
C09 | white-bg-hk | 香港用戶習慣資訊密集，純白背景顯得空洞；用 #F5F5F5 或 #FAFAFA | HK UX | 頁面感覺不完整
C10 | festive-color | 農曆新年/節日期間啟用紅金配色（香港文化）| HK UX | 錯失本地化連結機會

---

## LAYOUT 佈局（L01–L10）

L01 | safe-area | iOS 必須處理 Dynamic Island / Notch / Home Indicator 安全區域 | HIG | 內容被 notch 遮擋
L02 | status-bar | 內容不能延伸到 Status Bar 下（除非全屏沉浸）| HIG | 時間/電池圖示被覆蓋
L03 | keyboard-avoid | 輸入框獲得焦點時，鍵盤不能遮擋輸入框 | HIG/MD | 用戶看不到自己在打什麼
L04 | scroll-performance | 列表必須用 FlatList/LazyColumn，不能用 ScrollView 包 100+ 項 | Performance | 滾動卡頓，用戶流失
L05 | card-consistency | 同類卡片尺寸/間距/圓角必須統一 | HIG/MD | 頁面看起來很業餘
L06 | density-hk | 香港用戶接受更高信息密度（比西方 App 可以塞更多）| HK UX | 內容太空白，用戶覺得不夠用
L07 | grid-base | 所有間距基於 4px 或 8px 網格 | Material/HIG | 間距不一致，看起來不專業
L08 | fold-support | 如支持折疊屏，需要 Large Screen 佈局 | Android | 折疊後 UI 破版
L09 | landscape-graceful | 橫屏時 UI 不能完全壞掉（至少可用）| HIG/MD | 橫屏用戶體驗崩潰
L10 | iphone-se | iPhone SE（375px 寬）必須測試，香港仍有大量用戶 | HK UX | 小螢幕按鈕重疊，文字溢出

---

## FORMS 表單（Fo01–Fo08）

Fo01 | input-label | 輸入框必須有 Label，不能只靠 Placeholder | HIG/MD | 用戶填寫後 Placeholder 消失，不知道填了什麼
Fo02 | error-inline | 錯誤提示顯示在對應輸入框旁邊，不是頂部 | HIG/MD | 用戶找不到哪個欄位有錯
Fo03 | keyboard-type | 根據輸入類型設置鍵盤：數字/郵件/電話/URL | HIG/MD | 用戶需要手動切換鍵盤類型
Fo04 | autocomplete | 表單支援自動填充（姓名/地址/信用卡）| HIG/MD | 用戶需要手動輸入所有資料
Fo05 | submit-state | 提交按鈕需要 Loading 狀態，防止重複提交 | UX | 用戶連按多次，重複提交
Fo06 | form-progress | 長表單需要分步驟（≤4個欄位/步）| UX | 用戶看到長表單直接放棄
Fo07 | hk-phone | 香港手機號 8 位，需要正確的輸入格式和驗證 | HK UX | 輸入驗證錯誤，用戶無法提交
Fo08 | payment-hk | 支持 AlipayHK、WeChat Pay HK、Apple Pay、FPS（快速支付）| HK UX | 少了本地支付方式，轉化率降低

---

## PERFORMANCE 性能（P01–P08）

P01 | first-paint | 首屏有意義內容 < 2 秒（LTE）| Google | 用戶流失率直線上升
P02 | interaction-response | 用戶操作後 < 100ms 有視覺反饋 | HIG | 用戶以為沒有點到，重複點擊
P03 | image-optimization | 圖片用 WebP/HEIC，啟用 CDN，懶加載 | Performance | 頁面卡，流量消耗大
P04 | offline-state | 核心功能離線時需要清楚提示（而非崩潰）| HIG/MD | 用戶在地鐵/隧道斷網，App 假死
P05 | list-virtualization | 長列表（>50項）必須虛擬化渲染 | Performance | 記憶體溢出，App 崩潰
P06 | animation-fps | 動畫保持 60fps（< 16ms/frame）| HIG/MD | 動畫卡頓，App 顯得廉價
P07 | battery-conscious | 避免持續後台 GPS/感應器（用戶電量意識強）| HK UX | 用戶發現耗電，立刻卸載
P08 | app-size | App 大小 < 100MB（香港用戶存儲緊）| Store | 用戶猶豫下載，特別是舊機型

---

## ACCESSIBILITY 無障礙（A01–A08）

A01 | voiceover-label | 所有圖示/圖片需要 accessibilityLabel | WCAG | 視障用戶無法使用
A02 | color-contrast-aa | 所有文字對比度 ≥ 4.5:1 | WCAG AA | 弱視用戶閱讀困難
A03 | touch-target-access | 無障礙模式下觸控目標 ≥ 60×60pt | WCAG | 手部障礙用戶無法操作
A04 | dynamic-type-support | 所有文字支援系統字體大小調整 | HIG | 長者/弱視用戶調大字後 UI 破版
A05 | reduce-motion | 支援「減少動態效果」設定 | HIG | 前庭障礙用戶動畫引起不適
A06 | alt-text | 功能性圖片必須有文字替代 | WCAG | 圖片載入失敗用戶不知道是什麼
A07 | focus-indicator | 鍵盤/遙控操作時焦點指示器清晰可見 | WCAG | 鍵盤用戶無法知道焦點在哪
A08 | hk-senior | 香港長者 App：字體 ≥ 18px，觸控目標 ≥ 56pt，操作 ≤ 3 步 | HK UX | 長者無法使用，流失最大用戶群

---

## SHARING 分享功能（S01–S08）

S01 | share-whatsapp-first | 香港用家分享優先 WhatsApp，按鈕放第一位 | HK UX | 分享轉化率低 50%+
S02 | share-card-auto | 完成任務後自動生成分享卡（無需用戶操作）| Viral UX | 用戶懶得手動截圖
S03 | share-2-steps | 分享流程最多 2 步（選平台+發送）| UX | 每多一步流失 30% 用戶
S04 | prefill-message | 分享文案預填，用戶只需點發送 | UX | 空白文案讓用戶不知道寫什麼
S05 | share-timing | 分享按鈕只在情緒高點出現（完成/達標後）| UX | 錯誤時機的分享提示讓用戶反感
S06 | deep-link-referral | 邀請連結需要 Deep Link + 歸因（Branch.io）| Engineering | 安裝後跳主頁，新用戶流失
S07 | referral-dual | 邀請需要雙向獎勵（邀請方和被邀請方都得）| Growth | 單向獎勵轉化率低 3x
S08 | share-card-size | 分享卡需要 9:16（IG Story）和 1:1（WhatsApp）兩個版本 | HK UX | 尺寸不對，IG Story 有黑邊

---

## HK SPECIFIC 香港專屬（HK01–HK13）

HK01 | lang-traditional | 必須使用繁體中文，不是簡體 | HK Culture | 香港用戶極度反感簡體中文
HK02 | bilingual-ui | 界面支援中英切換（部分用戶偏好全英）| HK UX | 忽略英文用戶群
HK03 | ios-android-parity | Android 50.6% vs iOS 46.3%，兩個平台都必須優化 | HK Data | 忽略 Android 就失去一半用戶
HK04 | octopus-payment | 有實體支付場景需考慮八達通整合 | HK Culture | 失去交通/便利店用戶場景
HK05 | fps-payment | 支持 FPS 轉數快（香港主流即時支付）| HK Finance | 用戶付款不方便，放棄
HK06 | active-hours | 優先優化週五六晚 9-11pm（香港最活躍時段）| HK Data | Push 通知時間錯，無人回應
HK07 | lihkg-culture | 港式幽默和流行語可以適度使用（吸引本地用戶）| HK Culture | App 感覺不「貼地」
HK08 | cny-design | 農曆新年期間需要紅金主題版本 | HK Culture | 錯過最大節日營銷機會
HK09 | privacy-serious | 香港用戶對私隱敏感，權限請求需要解釋原因 | HK UX | 用戶拒絕授權，功能失效
HK10 | commute-design | 設計需適合地鐵使用：單手操作、耳機場景、低頭角度 | HK Context | 通勤場景體驗差，用戶流失
HK11 | info-density-high | 香港用戶可接受高密度信息，比西方 App 更緊湊 | HK UX | 太空白的設計讓用戶覺得沒料
HK12 | wechat-mainland | 如覆蓋大灣區，需要微信分享 + 簡繁體切換 | GBA UX | 錯過大灣區用戶
HK13 | se-test | iPhone SE（375px）仍然普及，必須測試 | HK Device | 按鈕重疊，文字溢出

---

## MONETIZATION UX 變現體驗（M01–M06）

M01 | value-before-paywall | 用戶先體驗核心價值，才展示 Paywall（Aha Moment 後）| Growth | 過早展示 Paywall，用戶還沒理解價值
M02 | rewarded-opt-in | Rewarded 廣告必須用戶主動選擇，不能強制 | Store Policy | 被強迫看廣告，用戶憤怒卸載
M03 | ad-natural-break | 廣告只在自然停頓點出現（任務完成/頁面切換）| UX | 核心流程中插廣告，用戶暴怒
M04 | trial-friction-low | 免費試用 ≤ 3 步開始，不需要信用卡 | Growth | 過多步驟開始試用，用戶放棄
M05 | cancel-respect | 取消訂閱流程清晰，不刻意隱藏取消選項 | HIG/Store | 被投訴，App Store 差評
M06 | price-3-tier | 定價用三層（基礎/標準/高級），用戶傾向選中間 | Psychology | 只有 2 個選項，轉化率低
