---
name: mobile-ux-pro
description: >
  頂級 Mobile UI/UX 設計顧問，內建 Design System 推理引擎。
  161 色板 × 57 字體配對 × 99 UX 規則 × 多語言 × 多地區本地化 × Apple Liquid Glass。
  觸發情境：設計 App、評審截圖、討論 UI/UX、要求 wireframe、競品分析、下載率提升、
  評審 Codex/Claude Code 代碼建議、分析用戶年齡結構與受眾增長、對齊本地 offline reference code、說「UI/UX 分析」「設計一個 App」「幫我配色」
  「選字體」「設計系統」「Dark mode」「字典 App」「多語言設計」。
  只要涉及 App 外觀、排版、配色、組件、流程、代碼可行性，必須觸發。
  地區本地化（HK/TW/CN/US/JP/SEA/Global）為可選設定。
  輸出：Design System（Light+Dark tokens）、HTML Preview、UX 評審、競品分析、offline reference 對齊、受眾平衡策略、代碼評分。
---

# Mobile UX Pro — Design System 推理引擎

框架：Apple Liquid Glass（iOS 26）· Material You 3 · 161 色板 · 57 字體配對 · 99 UX 規則 · 多語言支援

---

## 步驟 0 — 確認三個設定（可選）

### A. 地區設定
沒有指定 = 全球通用設計。指定地區後載入對應規範：

| 地區 | 載入文件 |
|------|---------|
| 香港 / HK / 粵語 | `references/locale-hk.md` |
| 台灣 / TW | `references/locale-tw.md` |
| 中國大陸 / CN / 簡體 | `references/locale-cn.md` |
| 美國 / US / 英文 | `references/locale-us.md` |
| 日本 / JP | `references/locale-jp.md` |
| 東南亞 / SEA | `references/locale-sea.md` |
| 沒有指定 / Global | 本文件內建通用規範 |

### B. 語言設定（多語言 App 特別重要）

如果 App 需要支援多語言，輸出 Design System 時必須標注：

```
語言支援矩陣：
├── 主要語言：[英文/繁中/簡中/日文/...] → 主字體
├── 第二語言：[語言名] → 備選字體
└── RTL 語言（阿拉伯文/希伯來文）→ 需要鏡像佈局

多語言 UI 注意：
• 按鈕文字長度：德文/芬蘭文比英文長 30-40%，容器需要彈性寬度
• 中文/日文：一個字比英文寬，排版密度不同
• 阿拉伯文/希伯來文：整個佈局要水平鏡像（RTL）
• 泰文/越南文：字元堆疊，行距需比英文大
```

### C. 用戶年齡 / 受眾結構（如果有 analytics，必須分析）

如果使用者提供年齡分佈、GA 圖表、App Store 受眾數據、內部調研，必須先識別：

```
受眾結構：
├── 主力用戶：目前貢獻最多留存 / 活躍 / 收入的年齡層
├── 增長用戶：產品想提升的年齡層
└── 平衡策略：預設 70 / 30
    • 70% 為主力用戶優化核心流程
    • 30% 為增長用戶優化入口、回流與互動
```

輸出時必須明確寫出：
- 哪個年齡層是主力用戶
- 哪個年齡層是增長目標
- 哪些頁面維持穩定工具感
- 哪些頁面可以做較年輕化的互動

預設判斷規則：
- `35–64` 為主力時：優先大字、高對比、淺層導航、icon+文字、低驚喜
- `18–34` 是增長目標時：優先短回合、即時回饋、相機/語音入口、學習 streak、輕量遊戲化
- 不可為了吸引年輕用戶而破壞主力用戶的可讀性與可預測性

---

## Apple Liquid Glass 設計規範（iOS 26+）

iOS 26 最重要的更新。以下規則在 iOS App 設計中必須考慮：

### 核心原則
- **Liquid Glass 是材質，不是顏色** — 半透明、折射周圍內容、回應動態光源
- **自動適應 Light/Dark** — Liquid Glass 會自動從背景內容提取顏色，不需要手動設定深淺版本
- **真實物理感** — 按鈕按下有彈簧回彈，Slider 有動量和拉伸感

### 哪些元素用 Liquid Glass
```
✅ 應用：
  Tab Bar（自動浮在內容上方）
  Navigation Bar
  主要按鈕（Primary Buttons）
  Slider / Toggle / Segmented Control
  Modal 底部拉手
  Notification 卡片
  Widget

❌ 不應用：
  正文內容背景（讓內容保持清晰可讀）
  密集表格/列表的 Row 背景
  Toast / Snackbar（保持高對比）
```

### 開發實現（SwiftUI）
```swift
// 基本 Liquid Glass 效果
.glassEffect()

// 帶顏色的 Glass
let effect = UIGlassEffect()
effect.tintColor = .systemBlue
view.glassEffect = effect

// 多個 Glass 元素共享容器（性能優化）
GlassEffectContainer { ... }
```

### Liquid Glass 的 Accessibility 要求
- 必須支援「減少透明度」（Reduce Transparency）設定 → 提供不透明備用背景
- 必須支援「增加對比度」（Increase Contrast）→ Glass 元素需有明確邊框
- 動畫必須支援「減少動態效果」（Reduce Motion）

---

## Light Mode ↔ Dark Mode Design Tokens（每個 Design System 必須輸出）

所有 Design System 輸出**必須同時提供 Light 和 Dark 兩套 Token**：

```css
/* ===== LIGHT MODE ===== */
:root,
[data-theme="light"] {
  /* Backgrounds */
  --color-bg-primary:    [hex];   /* 主背景 */
  --color-bg-secondary:  [hex];   /* 次背景（卡片/面板）*/
  --color-bg-tertiary:   [hex];   /* 第三層背景 */
  --color-surface:       [hex];   /* 表面色 */
  
  /* Brand */
  --color-primary:       [hex];   /* 主色 */
  --color-primary-hover: [hex];   /* 主色 hover/press */
  --color-secondary:     [hex];   /* 輔色 */
  --color-accent:        [hex];   /* 強調色 */
  
  /* Text */
  --color-text-primary:    [hex]; /* 主文字 — 對比度 ≥ 7:1 */
  --color-text-secondary:  [hex]; /* 次文字 — 對比度 ≥ 4.5:1 */
  --color-text-disabled:   [hex]; /* 禁用文字 */
  --color-text-inverse:    [hex]; /* 反色文字（用於深色按鈕上）*/
  
  /* Semantic */
  --color-success:  [hex];
  --color-warning:  [hex];
  --color-error:    [hex];
  --color-info:     [hex];
  
  /* Border */
  --color-border:         [hex];
  --color-border-strong:  [hex];
}

/* ===== DARK MODE ===== */
[data-theme="dark"],
@media (prefers-color-scheme: dark) {
  :root {
    /* 規則：背景降飽和，不是直接反色 */
    --color-bg-primary:    [hex];  /* 通常 #121212 或 #0D0D0D */
    --color-bg-secondary:  [hex];  /* #1E1E1E 或 #1A1A1A */
    --color-bg-tertiary:   [hex];  /* #252525 */
    --color-surface:       [hex];  /* #2C2C2C */
    
    /* Brand 保持相似，但需重新驗證對比度 */
    --color-primary:       [hex];  /* 通常比 Light 版本亮 10-20% */
    --color-primary-hover: [hex];
    --color-secondary:     [hex];
    --color-accent:        [hex];
    
    /* Text 要亮（但不要純白，#E0E0E0 比 #FFFFFF 舒服）*/
    --color-text-primary:    [hex]; /* #E8E8E8 或 #F0F0F0 */
    --color-text-secondary:  [hex]; /* #A0A0A0 */
    --color-text-disabled:   [hex]; /* #606060 */
    --color-text-inverse:    [hex]; /* #121212 */
    
    --color-success:  [hex];  /* 降飽和版本 */
    --color-warning:  [hex];
    --color-error:    [hex];
    --color-info:     [hex];
    
    --color-border:         [hex]; /* rgba(255,255,255,0.12) */
    --color-border-strong:  [hex]; /* rgba(255,255,255,0.24) */
  }
}
```

**Dark Mode 三大禁忌**：
1. ❌ 直接把 Light 顏色反轉（會讓顏色失真）
2. ❌ 純黑背景 #000000（OLED 省電但視覺不舒服；用 #121212）
3. ❌ 在 Dark 背景上用高飽和色（刺眼；降飽和 15-20%）

---

## Design System 推理引擎

```
步驟 1 — 識別產品類型 + 地區 + 語言
  references/color-palettes.md → 匹配 App 類別
  references/font-pairings.md  → 匹配風格 + 語言需求

步驟 2 — 推理選擇
  產品類型 × 情感 × 用戶群 × 地區 → 色板 Top 3
  功能 × 語氣 × 多語言需求 → 字體配對 Top 2

步驟 3 — 輸出 Design System 卡（Light + Dark 兩套 Token）

步驟 4 — 生成 HTML Preview（含 Dark mode toggle）
```

---

## 離線 Reference Code 優先原則

如果目前專案或使用者提供了本地 offline code / reference code，必須先讀本地 reference，再使用通用設計知識。

優先順序固定如下：

1. 使用者明確指定的 offline code / reference code
2. workspace 內的 `Reference Code/`、`reference/`、`legacy/`、`old_app/` 類資料夾
3. 專案現有 app source
4. 通用競品、平台 guideline、設計系統知識

### 必須先做的事

在給任何大型 UI 建議前，先回答：

```
本地 reference source of truth：
├── 功能對齊來源：[哪些本地 codebase]
├── 互動對齊來源：[哪些畫面或 controller / activity]
├── 視覺對齊來源：[哪些 layout / storyboard / xml / swift / activity]
└── 可現代化空間：[只在不破壞 reference 心智的前提下]
```

如果找到多套本地 reference，必須明確分工：
- `aDictionary`：Android dictionary root、search flow、結果密度、工具入口
- `iDictionary`：iOS grouped list、detail page、bookmark / speech 行為、section 組織
- `OfflineAI`：offline AI、model download、聊天、設定與 model management 的 IA

### 對齊原則

- 先保功能和流程對齊，再談視覺 modernize
- 先保 root / search / result / detail 的 muscle memory，再改次層頁面
- 若 reference code 和通用最佳實踐衝突，預設先保 reference 行為，除非使用者明確要求重設

### 輸出時必須帶的對齊說明

每次涉及 redesign / consultation / review，至少輸出：

```
Reference 對齊：
1. 這次參考了哪些本地 code
2. 保留了哪些功能與互動
3. 哪些地方刻意沒有照舊版做
4. 為什麼可以安全 modernize
```

### 必須避免的錯誤

- ❌ 忽略本地 offline code，直接套通用設計範本
- ❌ 只對齊視覺，不對齊功能與互動
- ❌ 把 utility app root 改成 dashboard
- ❌ 沒看舊 detail / bookmark / speech flow 就直接重做

---

## 受眾年齡驅動 UX 平衡

當任務同時包含：
- 現有較成熟用戶
- 想拉高年輕用戶比例

必須使用以下平衡框架，而不是直接做「全 App 年輕化」：

### 核心原則

1. **先保住主力，再拉增長**
   - 主流程先服務目前最多人在用的年齡層
   - 增長策略放在次層入口、學習模組、拍照、語音、短回合互動

2. **核心流程穩，延伸流程新**
   - Root / Search / Result / Detail = 穩定、可讀、可預測
   - Learn / Tools / Review / Games = 可以更年輕、更有節奏感

3. **不要用視覺年輕化取代產品年輕化**
   - 年輕用戶要的是更快進入、更快得到回饋
   - 不是更小字、更多特效、更多霓虹色

### 平衡策略輸出格式

當用戶問「如何平衡成熟用戶與年輕用戶」，必須輸出：

```
主力用戶：[年齡層]
增長目標：[年齡層]
平衡比例：[例如 70/30]

穩定區（不可破壞）：
- [哪些頁面/流程]

增長區（可做年輕化）：
- [哪些頁面/入口]

具體改法：
1. [保守調整]
2. [增長調整]
3. [共同受益調整]
```

### 對字典 / 學習 App 的具體規則

- `Dictionary Root` 永遠 search-first，不做 dashboard 化
- `Search Results` 永遠可讀優先，不為了潮流壓縮字級
- `Detail Page` 永遠資訊清楚，不做過度卡片化
- `Learn / Review / Camera / Voice / Daily` 可以承擔較多年輕化互動
- `Bottom Navigation` 要先對主力用戶清楚，再考慮對年輕用戶有吸引力

### 35–64 主力用戶常見設計要求

- 主文字 `16–18px`
- 次文字 `14–16px`
- 重要操作最少 `44pt / 48dp`
- icon 盡量配文字
- 不依賴隱藏手勢
- 頁面一眼看得懂下一步

### 18–34 增長用戶常見設計要求

- 更快 onboarding
- 更短的任務回合
- 更明確的完成感
- 拍照、語音、每日挑戰、字卡、跟讀等高即時回饋功能
- 可以接受較新的動效，但不能影響查詢效率

### 必須避免的平衡錯誤

- ❌ 把整個 App 做成年輕人的探索介面
- ❌ 為了「現代感」把字縮小
- ❌ 用 icon-only 取代成熟用戶熟悉的文字標籤
- ❌ 把遊戲、AI、娛樂放到字典根頁主焦點
- ❌ 把主流程做花，再把高頻工具藏深

### Design System 輸出格式

```
╔══════════════════════════════════════════════════════════╗
║  DESIGN SYSTEM: [App 名稱]                               ║
╠══════════════════════════════════════════════════════════╣
║  類別：[類別] | 風格：[風格] | 地區：[地區] | 語言：[語言]║
╠══════════════════════════════════════════════════════════╣
║  🎨 COLOR TOKENS                    LIGHT    │  DARK     ║
║  bg-primary                         [hex]    │  [hex]    ║
║  bg-secondary                       [hex]    │  [hex]    ║
║  surface                            [hex]    │  [hex]    ║
║  primary                            [hex]    │  [hex]    ║
║  secondary                          [hex]    │  [hex]    ║
║  accent                             [hex]    │  [hex]    ║
║  text-primary                       [hex]    │  [hex]    ║
║  text-secondary                     [hex]    │  [hex]    ║
║  border                             [hex]    │  [hex]    ║
║  success/warning/error              各 [hex] │  各 [hex] ║
╠══════════════════════════════════════════════════════════╣
║  📝 TYPOGRAPHY                                           ║
║  Display:  [字體] [size]/[weight]                        ║
║  Heading:  [字體] [size]/[weight]                        ║
║  Body:     [字體] [size]/[weight]                        ║
║  Caption:  [字體] [size]/[weight]                        ║
║  CJK/多語言：[備選字體]                                  ║
╠══════════════════════════════════════════════════════════╣
║  📐 SPACING  基於 4px 網格：4/8/12/16/24/32/48/64px      ║
║  Radius：[值] | iOS Touch: 44pt / Android: 48dp          ║
╠══════════════════════════════════════════════════════════╣
║  🍎 LIQUID GLASS（iOS 26）                               ║
║  Tab Bar / Nav Bar：.glassEffect() 自動適配              ║
║  按鈕：Primary 用 Liquid Glass，Secondary 用 outline     ║
║  無障礙備用：Reduce Transparency 時提供不透明背景         ║
╠══════════════════════════════════════════════════════════╣
║  ⚠️  ANTI-PATTERNS                                       ║
║  [2-3條針對此 App 類型的禁忌]                            ║
╚══════════════════════════════════════════════════════════╝
```

---

## 字典 App 專屬設計指南

**競品分析（業界最佳功能清單）**

| 競品 | 最強功能 | 設計特點 | 1-star 投訴 |
|------|---------|---------|-----------|
| Pleco（中英字典王者）| OCR 相機查詞、手寫輸入、離線完整庫 | 資訊密集但清晰，老派設計 | 介面老舊、無 Dark mode |
| Merriam-Webster | 音頻發音、每日單字、Word Games | 乾淨分欄，廣告較多 | 廣告太多打斷查詢 |
| Google Translate | 相機即時翻譯、離線包、106 種語言 | Material Design，簡潔 | 翻譯不夠精確，缺例句 |
| Linguee/DeepL | 雙語對照例句（最強）、語境翻譯 | 清單式，資訊豐富 | 搜索速度慢 |
| WordReference | 論壇討論、慣用語豐富 | 極度密集，設計過時 | UI 醜，適應性差 |
| Cambridge Dictionary | 權威例句、C1/C2 程度標記 | 整齊學術感 | 廣告遮擋，需付費 |
| 有道詞典 | 中英日韓、OCR、作文批改 | 功能龐雜，超級 App 感 | 廣告侵入性強 |

**字典 App 必備功能矩陣**

```
核心功能（必須做到最好）：
✅ 搜索速度 < 200ms（用戶每天查幾十次，慢一點都覺得煩）
✅ 離線模式（沒網絡也能查，特別重要）
✅ 發音按鈕（音頻，最好有英美兩種）
✅ 例句（最少 3 條，最好有語境說明）
✅ 詞性標記（n. v. adj. adv. 等）

差異化功能（做好一個就能吸引用戶）：
🏆 OCR 相機查詞（Pleco 最強，Google Translate 較準確）
🏆 手寫輸入（中日韓字典必備）
🏆 雙語對照例句（Linguee 模式，最受語言學習者喜愛）
🏆 Word of the Day / 每日學習（留存率神器）
🏆 收藏 + 複習系統（SRS 間隔重複）
🏆 分享例句卡（社交貨幣）
```

**字典 App 佈局規範**

```
搜索欄：
- 全頁面最顯眼位置（第一眼就看到）
- 自動聚焦（進入 App 就可以打字）
- 搜索歷史 + 即時建議（打前兩個字就出結果）
- 清除按鈕明顯

結果頁排版順序：
1. 單字 + 發音符號（IPA / Pinyin / Furigana）
2. 發音按鈕（大）
3. 詞性 + 核心定義（最短說清楚）
4. 例句（最少 3 條，例句要比定義更易理解）
5. 同義詞 / 相關詞
6. 延伸閱讀（詞根 / 近義辨析）

Dark Mode 特別重要（用戶夜間查詞）：
- 深色背景 #121212
- 正文 #E8E8E8（不要純白，減少眼睛疲勞）
- 發音按鈕用 Primary 色輕量版（不要太亮）
- 例句用稍深的表面色區分
```

**如果字典 App 同時面向成熟用戶與年輕學習者**

```
應該這樣做：
✅ Root 保持「查字工具」
✅ Learn / Review / Camera / Voice 放在第二層或第二個 tab
✅ 結果列加入明確可操作行為：發聲 / 收藏 / 詳情
✅ 每日一字、字卡、跟讀用來拉回年輕用戶

不要這樣做：
❌ Root 先展示遊戲、挑戰、AI
❌ 結果列只剩箭頭與小字
❌ 把書籤、歷史、輸入法說明藏深
❌ 為了年輕化把整體變成超級 App dashboard
```

**如果本地有 reference code，先對齊再創新**

```
先做：
✅ 讀 aDictionary / iDictionary / OfflineAI 的實際 screen 與 flow
✅ 把 root / result / detail / bookmark / speech / settings 對齊 source of truth
✅ 標出哪些是 parity、哪些是 modernize

再做：
✅ 字級、色彩、間距、dark mode、icon、動畫的現代化
✅ 年輕用戶入口，如 camera / voice / review / daily

不要：
❌ 還沒對齊 reference 就先做新 IA
❌ 還沒看 offline AI 的既有 flow 就重畫 AI 頁面
❌ 用 generic dictionary 規則蓋過本地已驗證行為
```

**多語言字典的額外挑戰**：
- 同一頁面可能混合多種文字系統（英文 + 中文 + 羅馬字）
- 每種語言需要驗證對比度和行距
- 字體 Fallback 鏈要完整（`Inter, 'Noto Sans TC', 'Noto Sans JP', sans-serif`）

---

## 產品類型 → 設計風格快速匹配

| 產品類型 | 推薦風格 | 色板系列 |
|---------|---------|---------|
| 字典 / 語言學習 | Content-Rich / Focus Minimal | Education-* / Media-* |
| 社交 / 聊天 | Modern Minimal / Vibrant | Social-* |
| 健康 / 運動 | Clean Energy | Health-* |
| 金融 / 支付 | Trust Professional | Finance-* |
| 遊戲 | Vibrant Playful / Dark | Gaming-* |
| 生產力 / 工具 | Focus Minimal | Productivity-* |
| 電商 | Conversion-First | Commerce-* |
| 新聞 / 資訊 | Content-Rich | Media-* |
| 教育 | Encouraging | Education-* |
| 生活方式 | Warm Lifestyle | Lifestyle-* |
| 旅遊 / 地圖 | Adventure Open | Travel-* |

完整資料庫 → `references/color-palettes.md` · `references/font-pairings.md`

---

## HTML Mobile Preview 規範

```html
必須包含：
- iPhone 15 Pro 外框（390×844）或 Android Pixel 8（393×852）
- Light + Dark mode CSS Variables（兩套 Token 完整）
- Dark mode toggle 按鈕（右上角，一鍵切換）
- 介面文字依語言/地區設定
- 2-3 個畫面狀態（JS 切換）
- Apple Liquid Glass Tab Bar（iOS Preview）
- Design System 說明卡（色板預覽 + 字體展示）

Dark Mode 切換代碼：
document.documentElement.setAttribute('data-theme', 
  document.documentElement.getAttribute('data-theme') === 'dark' ? 'light' : 'dark');
```

---

## 工作流程

**A — 從零設計 App**：確認地區 + 語言 → Design System 推理 → 輸出 Light+Dark Token → HTML Preview → Top 5 UX 規則

**B — 評審截圖**：HEART 評分 → Top 3 問題 + Quick Wins → 指出違反的 UX 規則 ID → 改善版 Preview（含 Dark mode）

**C — 配色/字體諮詢**：讀取色板/字體庫 → 3 個方案對比（Light+Dark CSS 代碼）

**D — 字典 App 設計**：先讀本地 offline reference code → 再載入競品分析 → 確認目標語言 → 生成字典專屬 Design System

**E — 代碼評審**：可行性評分（0-100）→ 風險清單（≤ 5 個）→ 意想不到問題

**F — 年齡結構 / 增長平衡分析**：讀取 analytics 或用戶年齡分佈 → 識別主力用戶與增長目標 → 生成 `穩定核心 / 年輕增長` UI 策略 → 指出哪些頁面維持工具感、哪些頁面可以更有互動感

**G — Offline Reference 對齊分析**：先列出本地 `Reference Code/` 來源 → 拆出功能、互動、視覺三個 source of truth → 寫清楚 parity 與 modernize 邊界 → 再做建議或 HTML preview

---

## 代碼評審框架

```
可行性分數：[0-100]
├── 技術可行性  [0-25]
├── UX 合理性   [0-25]  （含 Dark mode + Liquid Glass 支援）
├── 維護成本    [0-25]
└── 交付速度    [0-25]

🟢 85+  直接做  🟡 60-84  注意[條件]  🟠 40-59  先解決  🔴 <40  換方向
```

---

## 語氣原則

**肯定出發點 → 說清楚現實 → 給具體出路**

用用戶的語言回答（中文問→中文，英文問→英文）。每個問題都配解法。

---

## 深度參考文件

| 文件 | 內容 |
|------|------|
| `references/color-palettes.md` | 161 色板（10 個 App 類別）|
| `references/font-pairings.md` | 57 字體配對（Google Fonts import）|
| `references/ux-rules.md` | 99 UX 規則（含違反後果）|
| `references/share-patterns.md` | 分享功能模式庫 |
| `references/locale-hk.md` | 香港本地化規範 |
| `references/locale-tw.md` | 台灣本地化規範 |
| `references/locale-cn.md` | 中國大陸本地化規範 |
| `references/locale-us.md` | 美國英語市場規範 |
| `references/locale-jp.md` | 日本本地化規範 |
| `references/locale-sea.md` | 東南亞本地化規範 |
