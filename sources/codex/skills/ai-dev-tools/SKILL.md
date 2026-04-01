---
name: ai-dev-tools
description: >
  AI 功能整合、Prompt 設計與 MCP Server 構建一站式顧問。
  觸發：AI 功能、Claude API、OpenAI API、On-device ML、Core ML、TFLite、
  LLM 整合、AI 成本控制、Prompt 設計、Skill 設計、AI 輸出質量差、
  MCP、MCP server、整合外部 API 給 Claude 用、Claude 工具、
  說「加 AI 功能」「整合 Claude」「智能搜索」「AI 功能」「本地 AI」
  「怎麼讓 AI 處理 X」「幫我設計 Prompt」「AI 輸出不準確」「建 MCP」。
  Do NOT use for: non-AI feature work, pure UI design, general coding tasks.
---

# AI Dev Tools（AI功能整合 + Prompt設計 + MCP構建）

## 標準指令區塊

知識實時性：AI API 和 SDK 更新極快，必須先聯網搜索確認最新版本和用法。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用前：了解你的 AI 需求

首次使用時，先請用戶回答：

1. **你想加什麼 AI 功能？**（例：內容生成、智能搜索、圖像識別、個人化推薦）
2. **需要離線支持嗎？**（影響選擇 Cloud API vs On-device ML）
3. **用戶在哪個場景觸發 AI？**（影響延遲容忍度和成本控制策略）
4. **預計 AI 調用頻率？**（影響成本估算和緩存策略）

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 在 App 中加入 AI 功能 | §1 AI 功能整合 |
| 設計 Prompt / Skill | §2 Prompt 設計 |
| 構建 MCP Server | §3 MCP Builder |

---

## §1 AI 功能整合

### 技術選型

| 方案 | 優點 | 缺點 | 適合場景 |
|------|------|------|---------|
| **Claude API** | 質量最高、中文優秀、推理強 | 需要網絡、有費用 | 內容生成、分析、解釋 |
| **OpenAI API** | GPT-4 圖像能力強 | 成本高、延遲高 | 圖像理解、多模態 |
| **On-device ML** | 離線、快速、免費 | 能力有限 | 基礎分類、搜索排序 |
| **Core ML（iOS）** | 蘋果優化、隱私保護 | 只限 iOS | 端側推理 |
| **TFLite（Android）** | 跨平台、輕量 | 需要轉換模型 | 輕量分類任務 |

> 告訴我你的 App 需要哪種 AI 功能，我會推薦最適合的技術組合。

### Claude API 整合（KMP）

```kotlin
// commonMain — AI 服務接口（根據你的 App 功能定義方法）
interface Ai[YourFeature]Service {
    suspend fun process(input: String, context: String): Result<String>
    suspend fun generate(prompt: String): Result<List<String>>
}

// 實現
class ClaudeAiService(private val httpClient: HttpClient) : Ai[YourFeature]Service {
    private val baseUrl = "https://api.anthropic.com/v1"

    override suspend fun process(input: String, context: String): Result<String> =
        runCatching {
            val response = httpClient.post("$baseUrl/messages") {
                header("x-api-key", BuildConfig.CLAUDE_API_KEY)
                header("anthropic-version", "2023-06-01")
                contentType(ContentType.Application.Json)
                setBody(ClaudeRequest(
                    model = "claude-haiku-4-5-20251001",  // 低延遲，成本低
                    maxTokens = 300,
                    messages = listOf(Message(
                        role = "user",
                        content = buildPrompt(input, context)  // 你的 Prompt 構建邏輯
                    ))
                ))
            }
            response.body<ClaudeResponse>().content.first().text
        }

    private fun buildPrompt(input: String, context: String): String {
        // 根據你的 App 功能定製 Prompt
        return """
            上下文：$context
            輸入：$input
            [你的具體指令]
        """.trimIndent()
    }
}
```

### AI 成本控制

```kotlin
// 緩存 AI 結果（相同輸入不重複 API 調用）
class CachedAiService(
    private val service: Ai[YourFeature]Service,
    private val cache: SqlDelightCache
) : Ai[YourFeature]Service {
    override suspend fun process(input: String, context: String): Result<String> {
        val key = "${input}_${context.hashCode()}"
        cache.get(key)?.let { return Result.success(it) }

        return service.process(input, context).also { result ->
            result.onSuccess { cache.put(key, it, ttlHours = 24) }
        }
    }
}

// 成本追蹤（監控每日 API 費用）
Analytics.logEvent("ai_api_call", mapOf(
    "model" to "claude-haiku",
    "tokens_estimated" to 300,
    "feature" to "[your_feature_name]"
))
```

### On-device ML：智能排序/分類

```kotlin
// 不需要網絡的輕量 ML 邏輯（根據你的 App 實體替換）
class SmartRanker {
    fun rankResults(query: String, results: List<[YourEntity]>): List<[YourEntity]> {
        return results.sortedByDescending { item ->
            val exactMatch = if (item.title == query) 10.0 else 0.0
            val prefixMatch = if (item.title.startsWith(query)) 5.0 else 0.0
            val popularityScore = item.popularity * 0.1  // 熱門內容排前
            exactMatch + prefixMatch + popularityScore
        }
    }
}
```

---

## §2 Prompt 設計

### 設計原則

```
1. 明確目標（一個 Prompt 解決一個問題）
2. 提供上下文（讓 AI 理解場景）
3. 約束輸出（格式、長度、語言）
4. 用正向指令（「做 X」比「不要做 Y」更有效）
5. 加例子（few-shot 比 zero-shot 效果好 30-50%）
```

### Prompt 通用模板

```
# 內容生成類
---
背景：{context}
任務：根據以上背景，[你的具體任務]（用{language}，{N}字以內）。
格式：{具體輸出格式要求}
---

# 分析解釋類
---
輸入：{user_input}
請分析並解釋：[你的分析要求]
要求：
- 語言：{language}
- 長度：{N}字以內
- 格式：{具體格式}
---

# 分類/判斷類
---
輸入：{content}
請判斷：[你的判斷標準]
只返回：{選項A} 或 {選項B}（不要其他文字）
---
```

### Prompt 質量評估

```
測試一個 Prompt 是否好用：

□ 在 3 個不同輸入上測試（典型、邊界、異常）
□ 輸出格式穩定（不會有時有序號，有時沒有）
□ 長度符合預期（不會無緣無故很長或很短）
□ 沒有幻覺（輸出內容是真實的）
□ 在不同語言輸入下行為一致

評估矩陣：
準確性（輸出正確）× 穩定性（格式一致）× 效率（Token 用量少）
```

### Skill 設計原則

```
好的 Skill 特徵：
1. 單一職責（一個 Skill 解決一類問題）
2. 觸發詞精確（不會誤觸發）
3. SKILL.md ≤ 500 行（超過移到 references/）
4. 有使用指南（用戶知道怎麼用）
5. 有自查清單（質量保證）
```

---

## §3 MCP Builder

### MCP 是什麼

Model Context Protocol（MCP）是讓 Claude 調用外部工具的標準協議。通過 MCP Server，Claude 可以讀取你的文件、調用你的 API、查詢你的數據庫。

### TypeScript MCP Server 模板

```typescript
// src/index.ts — 將 [your-app] / [your_tool] 替換為實際名稱
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { CallToolRequestSchema, ListToolsRequestSchema } from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
    { name: "[your-app]-mcp", version: "1.0.0" },
    { capabilities: { tools: {} } }
);

// 定義工具
server.setRequestHandler(ListToolsRequestSchema, async () => ({
    tools: [{
        name: "[your_tool]",
        description: "[清楚描述這個工具的用途，Claude 會讀這個來決定是否調用]",
        inputSchema: {
            type: "object",
            properties: {
                query: { type: "string", description: "[輸入參數說明]" },
                filter: { type: "string", description: "[可選過濾參數]", default: "all" }
            },
            required: ["query"]
        }
    }]
}));

// 實現工具
server.setRequestHandler(CallToolRequestSchema, async (request) => {
    if (request.params.name === "[your_tool]") {
        const { query, filter = "all" } = request.params.arguments as { query: string; filter?: string };
        const results = await your[Tool]Implementation(query, filter);
        return {
            content: [{
                type: "text",
                text: JSON.stringify(results, null, 2)
            }]
        };
    }
    throw new Error(`Unknown tool: ${request.params.name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### MCP 配置（claude_desktop_config.json）

```json
{
    "mcpServers": {
        "[your-app]": {
            "command": "node",
            "args": ["/path/to/[your-app]-mcp/dist/index.js"],
            "env": {
                "DB_PATH": "/path/to/[your-app].db",
                "API_KEY": "your_api_key_here"
            }
        }
    }
}
```

### MCP 工具設計原則

```
每個工具：
□ 名稱動詞開頭（search_items, get_detail, list_favorites, create_record）
□ description 說清楚用途（Claude 會讀這個來決定是否調用）
□ inputSchema 嚴格定義（required 欄位明確）
□ 錯誤有意義（不只是 throw Error("failed")）
□ 返回結構化數據（不要返回大段文字）
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 Claude API / Anthropic SDK 文件 | Context7 |
| 讀寫本地數據庫（MCP Server 數據源） | SQLite MCP |
| 讀取 App 文件（MCP Server 文件源） | Filesystem MCP |

## 自查清單

- [ ] 已聯網確認 API 版本和 Token 限制？
- [ ] AI 調用有成本追蹤（Analytics）？
- [ ] AI 結果有緩存（相同查詢不重複調用）？
- [ ] Prompt 在 3 個測試案例上驗證過？
- [ ] MCP 工具 description 清晰（Claude 能理解用途）？
