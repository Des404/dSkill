---
name: db-design
description: >
  KMP 跨平台數據庫設計顧問，專注 SQLDelight + 離線優先架構，適用任何 Mobile App。
  觸發：數據庫設計、SQLDelight、Room、SQLite、Schema 設計、數據遷移、離線優先、
  本地緩存、數據同步、說「怎麼設計數據庫」「本地存儲」「Schema」「遷移」
  「離線」「緩存」「本地DB怎麼做」「SQLDelight 設置」「Migration 怎麼寫」。
  Do NOT use for: UI/UX design, API client code, or tasks unrelated to local database design.
---

# DB Design（SQLDelight + 離線優先架構）

## 標準指令區塊

知識實時性：必須聯網搜索確認 SQLDelight / Room 最新版本號和語法。
引用規範：所有來源用 [來源N](URL) 封裝，嚴禁裸露 URL。
誠實原則：找不到精準來源時直說，嚴禁資訊幻覺。
務實原則：只解當前真實問題，最簡方案，拒絕過度設計。
探針提問：請求模糊時，問一個最關鍵的澄清問題，確認後再執行。

---

## 使用前：了解你的數據需求

> 在設計 Schema 前，先確認：
> - **主要數據類型**：你的 App 存什麼數據？（用戶記錄、內容、設定、緩存）
> - **數據來源**：純本地 / API 緩存 / 混合？
> - **離線需求**：需要完整離線功能嗎？還是只是緩存？
> - **數據量級**：幾百條 / 幾萬條 / 幾十萬條？

---

## 使用指南

| 你想做的事 | 跳到哪裡 |
|-----------|---------|
| 選 SQLDelight 還是 Room | §1 技術選型 |
| 設計 Schema | §2 Schema 設計 |
| 寫 Migration | §3 遷移策略 |
| 離線優先架構 | §4 離線優先 |

---

## §1 技術選型

### SQLDelight vs Room

| | SQLDelight | Room |
|--|-----------|------|
| KMP 支持 | ✅ iOS + Android | ❌ Android only |
| 類型安全 | ✅ 編譯時生成 Kotlin | ✅ |
| 遷移方式 | SQL 文件（`.sqm`） | Kotlin DSL |
| 查詢語言 | 原生 SQL | 原生 SQL + 部分 Kotlin |
| **推薦** | **KMP 項目首選** | Android only 項目 |

**KMP 項目 → 用 SQLDelight**

### SQLDelight 設置

```kotlin
// build.gradle.kts (shared)
sqldelight {
    databases {
        create("[YourApp]Database") {
            packageName.set("com.yourapp.db")
            version = 1
        }
    }
}

// expect/actual Driver（跨平台核心）
// commonMain
expect class DatabaseDriverFactory { fun createDriver(): SqlDriver }

// androidMain
actual class DatabaseDriverFactory(private val context: Context) {
    actual fun createDriver(): SqlDriver =
        AndroidSqliteDriver([YourApp]Database.Schema, context, "yourapp.db")
}

// iosMain
actual class DatabaseDriverFactory {
    actual fun createDriver(): SqlDriver =
        NativeSqliteDriver([YourApp]Database.Schema, "yourapp.db")
}
```

---

## §2 Schema 設計

### 設計原則

```
1. 每個表只存一類事物（單一職責）
2. 用 TEXT 存 UUID/ID（跨平台兼容）
3. 時間戳用 INTEGER（Unix timestamp，毫秒）
4. 必要的 INDEX（搜索/過濾字段都要建）
5. 外鍵加 ON DELETE CASCADE（保持數據一致性）
```

### 通用 Schema 模板

```sql
-- 核心數據表（根據你的 App 調整）
CREATE TABLE [YourMainEntity] (
    id TEXT NOT NULL PRIMARY KEY,
    -- 你的核心字段...
    [field1] TEXT NOT NULL,
    [field2] TEXT,
    [field3] INTEGER NOT NULL DEFAULT 0,
    is_favorite INTEGER NOT NULL DEFAULT 0,  -- 收藏功能
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL
);

-- 常用 INDEX（根據查詢需求建立）
CREATE INDEX [entity]_[field]_idx ON [YourMainEntity]([field1]);

-- 用戶行為記錄表（瀏覽歷史、操作記錄）
CREATE TABLE UserAction (
    id TEXT NOT NULL PRIMARY KEY,
    entity_id TEXT NOT NULL,
    action_type TEXT NOT NULL,  -- 'view', 'favorite', 'share' 等
    performed_at INTEGER NOT NULL
);

-- 用戶設定表
CREATE TABLE UserPreference (
    key TEXT NOT NULL PRIMARY KEY,
    value TEXT NOT NULL,
    updated_at INTEGER NOT NULL
);

-- 緩存元數據表（追蹤緩存版本）
CREATE TABLE CacheMeta (
    cache_key TEXT NOT NULL PRIMARY KEY,
    version INTEGER NOT NULL DEFAULT 1,
    last_synced_at INTEGER NOT NULL,
    expires_at INTEGER  -- NULL = 永不過期
);
```

### 常用查詢模板

```sql
-- 搜索（模糊匹配）
searchItems:
SELECT * FROM [YourMainEntity]
WHERE [field1] LIKE '%' || :query || '%'
ORDER BY CASE WHEN [field1] = :query THEN 0 ELSE 1 END, [field1]
LIMIT :limit OFFSET :offset;

-- 按類別過濾
getByCategory:
SELECT * FROM [YourMainEntity]
WHERE [category_field] = :category
ORDER BY created_at DESC
LIMIT :limit;

-- 收藏列表
getFavorites:
SELECT * FROM [YourMainEntity]
WHERE is_favorite = 1
ORDER BY updated_at DESC;

-- 最近查看
getRecentActions:
SELECT e.* FROM [YourMainEntity] e
JOIN UserAction a ON e.id = a.entity_id
WHERE a.action_type = 'view'
ORDER BY a.performed_at DESC
LIMIT :limit;

-- Insert or Update
upsertItem:
INSERT OR REPLACE INTO [YourMainEntity] VALUES ?;

-- 標記收藏
toggleFavorite:
UPDATE [YourMainEntity] SET is_favorite = :isFavorite, updated_at = :now WHERE id = :id;
```

### Repository 實現

```kotlin
class [YourApp]LocalDataSource(database: [YourApp]Database) {
    private val queries = database.[yourMainEntity]Queries

    fun searchItems(query: String, limit: Long = 20): Flow<List<[YourMainEntity]>> =
        queries.searchItems(query, limit, 0)
            .asFlow()
            .mapToList(Dispatchers.Default)
            .map { rows -> rows.map { it.toDomain() } }

    suspend fun upsertItem(item: [YourMainEntity]) {
        queries.upsertItem(item.toEntity())
    }

    fun getFavorites(): Flow<List<[YourMainEntity]>> =
        queries.getFavorites()
            .asFlow()
            .mapToList(Dispatchers.Default)
            .map { rows -> rows.map { it.toDomain() } }
}
```

---

## §3 遷移策略

```sql
-- sqldelight/migrations/2.sqm（版本 1 → 2）
-- 加新字段（必須有 DEFAULT）
ALTER TABLE [YourMainEntity] ADD COLUMN [new_field] TEXT;
ALTER TABLE [YourMainEntity] ADD COLUMN [new_flag] INTEGER NOT NULL DEFAULT 0;

-- sqldelight/migrations/3.sqm（版本 2 → 3，加新表）
CREATE TABLE [NewEntity] (
    id TEXT NOT NULL PRIMARY KEY,
    [parent_id] TEXT NOT NULL,
    FOREIGN KEY ([parent_id]) REFERENCES [YourMainEntity](id) ON DELETE CASCADE
);
```

**遷移規則：**
- 每個版本一個 `.sqm` 文件，文件名是目標版本號
- 只用 DDL（`ALTER TABLE` / `CREATE TABLE` / `CREATE INDEX`）
- **不刪除舊列**（向後兼容），加新列必須有 `DEFAULT` 值
- SQLite 不支持 `DROP COLUMN`（需要的話要重建表）
- **在含有舊數據的設備上測試**，不只是新安裝

### 遷移測試

```kotlin
@Test
fun `migration 1 to 2 preserves existing data`() {
    // 1. 用版本1 schema 創建數據庫並插入數據
    val db1 = createV1Database()
    db1.insertTestData()

    // 2. 跑遷移
    val db2 = migrateToV2(db1)

    // 3. 確認數據完整
    assertEquals(expectedCount, db2.query().count())
}
```

---

## §4 離線優先架構

### 核心模式：Cache-First

```kotlin
// Repository：先讀本地，後台同步網絡
override fun getItems(query: String): Flow<Result<List<[YourMainEntity]>>> = flow {
    // 1. 立即返回本地數據（用戶不等待）
    val localData = localDataSource.searchItems(query).first()
    emit(Result.success(localData))

    // 2. 後台同步網絡（如果有網絡）
    if (networkMonitor.isConnected) {
        try {
            val remoteData = remoteDataSource.fetch(query)
            localDataSource.upsertItems(remoteData)
            // 3. 返回更新後的數據
            emit(Result.success(localDataSource.searchItems(query).first()))
        } catch (e: Exception) {
            // 網絡失敗不影響本地結果，靜默處理或發出 warning
            if (localData.isEmpty()) emit(Result.failure(e))
        }
    }
}
```

### 緩存過期策略

```kotlin
// 根據數據特性選擇策略
enum class CacheStrategy {
    NEVER_EXPIRE,           // 靜態數據（版本內不變）
    TIME_BASED,             // 有時效性的數據（TTL = X 小時）
    VERSION_BASED,          // 跟著 App 版本更新
    USER_TRIGGERED          // 用戶手動刷新
}

suspend fun shouldRefresh(cacheKey: String): Boolean {
    val meta = db.cacheMeta(cacheKey) ?: return true
    return meta.expiresAt?.let { System.currentTimeMillis() > it } ?: false
}
```

### 離線數據包（大型靜態數據集）

```kotlin
// 適合場景：App 有大量只讀數據（目錄、內容庫、參考數據）
object OfflinePackageManager {
    suspend fun downloadIfNeeded(packageId: String) {
        if (isPackageInstalled(packageId)) return

        val url = "https://cdn.yourapp.com/$packageId.db.gz"
        val progress = MutableStateFlow(0f)
        downloadWithProgress(url, packageId, progress)
    }

    // 差異更新（只下載有變化的部分）
    suspend fun checkForUpdates(packageId: String) {
        val localVersion = getLocalVersion(packageId)
        val remoteVersion = api.getLatestVersion(packageId)
        if (remoteVersion > localVersion) {
            downloadDelta(packageId, fromVersion = localVersion)
        }
    }
}
```

---

## 🔌 推薦 MCP

| 需求 | MCP |
|------|-----|
| 查 SQLDelight / Room 官方文件 | Context7 |
| 讀寫本地 Schema / migration 文件 | Filesystem MCP |
| 本地 DB 查詢測試 | SQLite MCP |

## 自查清單

- [ ] 已聯網確認 SQLDelight 版本和語法？
- [ ] 每個查詢字段都有對應 INDEX？
- [ ] Migration 在舊數據上測試過（不只是空庫）？
- [ ] 新列有 DEFAULT 值（向後兼容）？
- [ ] 離線優先：先返回本地數據，後台同步？
