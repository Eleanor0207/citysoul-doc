# 城市靈魂(City Souls / Project Echo)資料庫與後端邏輯技術文件

> 本文件為純後端技術參考,涵蓋 PostgreSQL 完整 schema、Pydantic 介面定義、與核心組裝邏輯。
> 產品設定、敘事文案、美術資產等非資料庫內容,請參照《專案重點整理》《開發紀錄》《WBS 與 API 切分》與整合文件。
> 資料庫分兩個 schema:預設 `public`(body 模組)與 `brain`(brain 模組)。

## 讀這份文件之前必須知道的四條硬限制

這四條不是設計偏好,是已經寫進 CONTEXT.md 與驗收標準的邊界。新增欄位或新增表時**先對照這裡**:

1. **匿名優先**。玩家不登入就能玩,`players.device_id` 是身分,登入資訊全部可為 NULL。
2. **不存移動軌跡**。任何帶 `player_id` + 時間 + 座標的持久資料列都是軌跡,不論欄位叫什麼。原始 GPS 只在驗證期間使用,用完即丟。
3. **身體與腦袋不建跨 schema 外鍵**(WBS-API 決策4)。`public` 與 `brain` 之間只用 `player_id` / `spirit_id` / `character_id` 的**值**做邏輯關聯,不寫 `REFERENCES`。本文件中所有跨 schema 的關聯都標記為「值關聯」。
4. **人格卡只能人工審核通過**。`brain.character_personas.active` 沒有任何程式碼路徑可以自動 flip。

---

## 目錄

1. [Body 模組 — 玩家與帳號](#1-body-模組--玩家與帳號)
2. [Body 模組 — 靈魂實體與共鳴](#2-body-模組--靈魂實體與共鳴)
3. [Body 模組 — 任務系統](#3-body-模組--任務系統)
4. [Body 模組 — 媒體與內容審核](#4-body-模組--媒體與內容審核)
5. [Body 模組 — 對話紀錄與主線進度](#5-body-模組--對話紀錄與主線進度)
6. [Body 模組 — 配額(Usage Tier)](#6-body-模組--配額usage-tier)
7. [Brain 模組 — 靈魂三層資料](#7-brain-模組--靈魂三層資料)
8. [Brain 模組 — 共鳴解鎖與主線節點](#8-brain-模組--共鳴解鎖與主線節點)
9. [Brain 模組 — 跨角色主線案件與地理圍欄](#9-brain-模組--跨角色主線案件與地理圍欄)
10. [Brain 模組 — 記憶系統](#10-brain-模組--記憶系統)
11. [Pydantic 介面與 Context 組裝邏輯](#11-pydantic-介面與-context-組裝邏輯)
12. [核心後端邏輯函式](#12-核心後端邏輯函式)
13. [尚未定案 / 延伸事項](#13-尚未定案--延伸事項)

---

## 1. Body 模組 — 玩家與帳號

```sql
CREATE TABLE players (
    player_id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id            TEXT NOT NULL UNIQUE,    -- 匿名身分,一定有值
    display_name          TEXT,                    -- 以下四欄僅「已綁定帳號」的玩家有值
    auth_provider          TEXT,                   -- google/apple/email
    auth_provider_id        TEXT,
    avatar_url              TEXT,
    usage_tier_id            VARCHAR(32) NOT NULL REFERENCES usage_tiers(tier_id),
    created_at                TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_active_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
    total_summons               INT NOT NULL DEFAULT 0,
    notification_opt_in          BOOLEAN NOT NULL DEFAULT true,
    -- 防止「有 provider 沒有 id」的半綁定狀態
    CONSTRAINT ck_players_auth_pair
        CHECK ((auth_provider IS NULL) = (auth_provider_id IS NULL))
);

-- 只約束已綁定的玩家。不能寫成表上的 UNIQUE(auth_provider, auth_provider_id):
-- 匿名玩家全是 (NULL, NULL),而 Postgres 的 UNIQUE 不擋重複 NULL,
-- 那樣寫不會報錯,但它也就完全沒有在保護任何東西。
CREATE UNIQUE INDEX uq_players_auth ON players(auth_provider, auth_provider_id)
    WHERE auth_provider IS NOT NULL;

CREATE TABLE player_settings (
    player_id           UUID PRIMARY KEY REFERENCES players(player_id),
    language              TEXT NOT NULL DEFAULT 'zh-TW',
    sound_enabled          BOOLEAN NOT NULL DEFAULT true,
    haptics_enabled         BOOLEAN NOT NULL DEFAULT true
);
```

**匿名 → 綁定的升級路徑**:綁定是對既有列做 `UPDATE`,`player_id` 不變,共鳴值、收藏、任務進度全部原地保留。不是建立新玩家再搬資料。

**`usage_tier_id` 是 NOT NULL,所以 `usage_tiers` 必須先有資料 `players` 才插得進去。** 這是架構相依而非 seed data——空資料庫沒有 `closed_beta` 那筆的話連匿名玩家都建不出來。因此 `closed_beta` tier 與其 limits 寫在 **Alembic migration 裡**,不放在 seed script(見第6節)。

**備註**:`player_settings` 建議在玩家註冊流程中同步建立一筆預設值,避免查詢時需額外判斷是否存在。

---

## 2. Body 模組 — 靈魂實體與共鳴

```sql
CREATE TABLE spirits (
    spirit_id             TEXT PRIMARY KEY,        -- 例如 'longshan_temple'
    character_id           TEXT,                   -- 值關聯 → brain.character_personas,不建 FK
    landmark_id             TEXT,                  -- 值關聯 → brain.landmark_souls,不建 FK
    display_name             TEXT NOT NULL,
    thumbnail_url             TEXT,
    addressable_key            TEXT,               -- Unity Addressables 角色資產 key
    latitude                   NUMERIC(9,6) NOT NULL,
    longitude                   NUMERIC(9,6) NOT NULL,
    sense_radius_meters          INT NOT NULL DEFAULT 150,
    summon_radius_meters          INT NOT NULL DEFAULT 50,
    is_active                      BOOLEAN NOT NULL DEFAULT true,
    CONSTRAINT uq_spirits_character_id UNIQUE (character_id)  -- 1對1:一個地標靈魂僅一種人格
);

CREATE TABLE resonance (
    player_id             UUID REFERENCES players(player_id),
    spirit_id               TEXT REFERENCES spirits(spirit_id),
    resonance_value          INT NOT NULL DEFAULT 0,
    first_summoned_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_interaction_at        TIMESTAMPTZ,
    PRIMARY KEY (player_id, spirit_id)
);

-- 共鳴值流水帳。這張表是防止重複入帳的唯一依靠。
CREATE TABLE resonance_events (
    resonance_event_id     UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id               UUID NOT NULL REFERENCES players(player_id),
    spirit_id                TEXT NOT NULL REFERENCES spirits(spirit_id),
    source_type               TEXT NOT NULL,   -- 'encounter_collection' / 'quest'
    source_id                  TEXT NOT NULL,
    amount                      INT NOT NULL,   -- encounter_collection=10;quest=20
    awarded_at                   TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_resonance_events_source UNIQUE (player_id, source_type, source_id)
);

CREATE TABLE encounter_collections (
    collection_id        UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id             UUID NOT NULL REFERENCES players(player_id),
    place_id               TEXT NOT NULL REFERENCES spirits(spirit_id),
    recognized_label        TEXT,          -- 本機辨識結果,不含原始照片
    collected_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
    CONSTRAINT uq_encounter_collections UNIQUE (player_id, place_id)
);
```

**設計原則**:

- **兩個半徑是兩段不同的體驗,不是同一個值的寬鬆版本**(SDD §5.5、AC2、AC11)。`sense_radius_meters`(150m)進入感應範圍,地標淡淡發光、可以隔空聊天;`summon_radius_meters`(50m)才算在場成立,可以召喚與挑戰任務。
- `spirits.character_id` 有 UNIQUE 約束,強制 1 對 1。未來若要支援人格變體,需移除此約束並改用中介表,不影響既有資料。
- `relationship_stage`(疏離/熟悉/信任/摯友)**不存欄位**,由 `resonance_value` 運行時計算(見第11節)。同理 `resonance` 表也**沒有** `stage` 欄位——一個永遠不被信任的快取欄位,存在的唯一效果是讓下一個人誤用它。
- **座標用 `NUMERIC(9,6)` 而不是浮點數**。6 位小數約 11 公分,足夠;而且距離判斷是遊戲規則的一部分,不應該有浮點誤差。

### 為什麼需要 `resonance_events`(以及 `encounter_collections` 的 UNIQUE)

`resonance` 只記「現在幾分」,`resonance_events` 記「每一分是從哪來的」。它存在的理由不是稽核,是**正確性**:

AC5.2 要求「該地標收藏只會加這一次」。直覺寫法是先查再寫:

```python
if not already_awarded(player, source):   # 查
    resonance.resonance_value += 10       # 寫
```

玩家連點兩下或 client 自動重試,兩個請求同時進來:A 查到沒入過帳,B 也查到沒入過帳(A 還沒寫完),然後兩邊各加 10。中間沒有任何一步做錯。**「先查再寫」這個結構本身擋不住它**,兩步靠得再近,中間永遠有一條縫。

正確的做法是把順序倒過來,讓資料庫判定:

```python
db.add(ResonanceEvent(player_id, spirit_id, "encounter_collection", place_id, 10))
try:
    db.flush()                      # UNIQUE 約束在這裡裁決
except IntegrityError:
    return already_awarded()        # 撞到 = 別人已經入過帳
resonance.resonance_value += 10
```

**先寫,撞到約束才知道自己是重複的。** UNIQUE 約束是原子的,兩個並行請求裡必然只有一個成功。

> ⚠️ 這個 UNIQUE **不含 `spirit_id`**:是 `(player_id, source_type, source_id)` 三欄。同一個 `source_id` 對同一個玩家全域只能入帳一次,跨靈魂也不行。因此 `source_id` 必須自帶地標資訊(`longshan_temple`、`q_longshan_lantern`)才不會互撞。

附帶效果:`resonance.resonance_value` 因此變成可重算的快取(`SUM(amount)`),給分規則改了或算錯了都能修。

---

## 3. Body 模組 — 任務系統

```sql
CREATE TABLE quests (
    quest_id        TEXT PRIMARY KEY,
    spirit_id         TEXT REFERENCES spirits(spirit_id),
    title             TEXT NOT NULL,
    quest_type         TEXT NOT NULL,      -- 'daily' / 'story' / 'resonance_gated'
    min_resonance      INT NOT NULL DEFAULT 0,
    story_beat_id       TEXT,              -- 值關聯 → brain.story_beats,不建 FK。僅 story 類型有值
    steps              JSONB NOT NULL DEFAULT '[]',
    -- steps 範例:
    -- [
    --   {"step_id": "s1", "description": "靠近地標50公尺內", "objective_type": "geo_proximity"},
    --   {"step_id": "s2", "description": "拍攝一張照片", "objective_type": "photo_submission"},
    --   {"step_id": "s3", "description": "與靈魂對話三次", "objective_type": "dialogue_count", "target_value": 3}
    -- ]
    reward_type        TEXT,               -- 'resonance' / 'item'(MVP階段單一獎勵類型,不支援複合)
    reward_value        JSONB,
    -- resonance 類型: {"resonance_gain": 20}   ← 數值由 AC5.1 定,不可自行調整
    -- item 類型:      {"item_type": "badge", "item_id": "badge_longshan_first_visit"}
    is_active          BOOLEAN NOT NULL DEFAULT true
);

CREATE TABLE quest_progress (
    progress_id          BIGSERIAL PRIMARY KEY,
    player_id              UUID NOT NULL REFERENCES players(player_id),
    quest_id                TEXT NOT NULL REFERENCES quests(quest_id),
    issued_date              DATE,                          -- daily 才有值;story/resonance_gated 為 NULL
    status                  TEXT NOT NULL DEFAULT 'in_progress',  -- in_progress/completed/failed/expired
    current_step_index         INT NOT NULL DEFAULT 0,
    step_states               JSONB NOT NULL DEFAULT '{}',
    -- ⚠️ step_states 只記「通過了」這個事實,**不得寫入任何座標**。
    --    正確: {"s1": {"completed_at": "2026-08-07T10:12:00+08:00"}}
    --    禁止: {"s1": {"completed_at": "...", "location": [25.037, 121.499]}}
    --    這是持久資料列,帶 player_id 與時間;塞進座標就是移動軌跡,只是叫別的名字。
    attempts_today             INT NOT NULL DEFAULT 0,
    attempts_date               DATE NOT NULL,               -- Asia/Taipei 的「哪一天」
    current_token_issued_at      TIMESTAMPTZ,
    started_at                    TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at                   TIMESTAMPTZ,
    expires_at                      TIMESTAMPTZ,
    last_updated_at                  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 一次性任務(story/resonance_gated):同玩家同任務只能有一筆
CREATE UNIQUE INDEX uq_quest_progress_onetime
    ON quest_progress(player_id, quest_id) WHERE issued_date IS NULL;

-- 每日任務:同玩家同任務同一天只能有一筆,不同天可重複發放新紀錄
CREATE UNIQUE INDEX uq_quest_progress_daily
    ON quest_progress(player_id, quest_id, issued_date) WHERE issued_date IS NOT NULL;

CREATE TABLE quest_narrative_cache (
    quest_id        TEXT REFERENCES quests(quest_id),
    player_id        UUID REFERENCES players(player_id),
    generated_text     TEXT NOT NULL,
    generated_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (quest_id, player_id)
);

CREATE TABLE player_inventory (
    inventory_id      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id           UUID REFERENCES players(player_id),
    item_type            TEXT NOT NULL,   -- 'badge' / 'collectible' / 'story_memento' / 'story_key_item' / 'story_document'
    item_id               TEXT NOT NULL,   -- 對應靜態物品定義,MVP階段寫死在 code
    source_quest_id       TEXT REFERENCES quests(quest_id),
    illustration_asset_id   UUID REFERENCES media_assets(asset_id),  -- NULL 表示無圖(如 badge)
    acquired_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_inventory_player ON player_inventory(player_id);

-- 防止同一道具重複發放(例如主線劇情道具、區域進場信件)
CREATE UNIQUE INDEX uq_inventory_player_item ON player_inventory(player_id, item_id);
```

**`progress_id` 用 BIGSERIAL 而不是 `(player_id, quest_id)` 複合主鍵**:daily 任務每天重新發放一筆,複合主鍵表達不了「同一玩家同一任務的多筆紀錄」。`quest_photo_submissions` 也需要一個單一欄位可以指。唯一性改由上面兩個 partial index 保證。

**`issued_date` 與 `attempts_date` 是兩件不同的事,兩個都要**:

| 欄位 | 回答的問題 |
|---|---|
| `issued_date` | 這筆 daily 任務是哪一天發的(決定唯一性與過期) |
| `attempts_today` / `attempts_date` | 這個玩家今天試了幾次(決定 `daily_limit_reached`) |

`daily_limit_reached` **不是資料庫狀態**,是查詢當下才算得出來的結果——它跟今天是哪一天有關,存進 `status` 欄位隔天就是錯的。`status` 只有 `in_progress` / `completed` / `failed` / `expired`。

**每日任務發放策略**:採**惰性發放(lazy issue)**,不由 Cloud Scheduler 預先幫全部玩家寫入。玩家當天首次觸發 daily quest 查詢時,即時檢查並建立當日紀錄(見第12節 `get_or_issue_daily_quest`)。

**每日重置機制**:固定 **Asia/Taipei 午夜**(SDD 決策8),不是 UTC。這是一款台北在地遊戲,玩家對「今天」的認知就是台北的今天;UTC 午夜等於台北早上八點,玩家吃早餐時任務突然刷新。Cloud Scheduler 僅負責將前一天未完成的 daily quest 標記為 `expired`,不負責發放新任務:

```sql
UPDATE quest_progress
SET status = 'expired'
WHERE issued_date < (now() AT TIME ZONE 'Asia/Taipei')::date
  AND status = 'in_progress';
```

> 此 SQL 天生不會影響 story 類型任務,因為 `issued_date IS NULL` 時比較結果為 NULL(PostgreSQL NULL 比較行為),故無需額外加 `WHERE quest_type = 'daily'`。

**Daily resonance 任務與主線任務互不限制**:兩者分屬不同 `quest_type`,完成時分別只觸碰 `resonance` 表或 `players_story_progress` 表,路徑完全獨立(見第12節 `complete_quest`)。

---

## 4. Body 模組 — 媒體與內容審核

```sql
CREATE TABLE media_assets (
    asset_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_type       TEXT NOT NULL,   -- 'spirit_avatar' / 'landmark_photo' / 'quest_illustration'
                                       -- / 'player_summon_photo' / 'quest_verification_photo'
                                       -- / 'story_item_illustration'(主線道具插圖,見第9節)
    owner_type       TEXT NOT NULL,   -- 'system' / 'player'
    owner_id         TEXT,            -- player_id 或 spirit_id,依 owner_type 而定
    spirit_id         TEXT REFERENCES spirits(spirit_id),  -- 系統素材為 NULL
    gcs_path          TEXT NOT NULL,
    cdn_url           TEXT NOT NULL,
    content_type       TEXT NOT NULL,
    width             INT,
    height             INT,
    uploaded_at         TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE quest_photo_submissions (
    submission_id       UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    progress_id           BIGINT REFERENCES quest_progress(progress_id),
    step_id                 TEXT,  -- 對應 quests.steps 中的 step_id,標示驗證哪一步
    asset_id               UUID REFERENCES media_assets(asset_id),
    verification_status      TEXT NOT NULL DEFAULT 'pending',   -- pending / approved / rejected
    verification_method      TEXT,                              -- 'vision_api_auto' / 'manual'
    verified_at              TIMESTAMPTZ,
    submitted_at              TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE content_moderation_flags (
    flag_id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    asset_id             UUID REFERENCES media_assets(asset_id),
    flag_reason           TEXT,             -- 'auto_detected' / 'user_reported'
    status               TEXT NOT NULL DEFAULT 'pending',   -- pending/reviewed/removed
    flagged_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
    reviewed_at             TIMESTAMPTZ
);

CREATE INDEX idx_moderation_status ON content_moderation_flags(status) WHERE status = 'pending';
```

> ⚠️ `media_assets` **不存 EXIF**。手機照片的 EXIF 內含拍攝地點座標與時間,原樣保存等同建立位置紀錄。上傳處理流程必須在寫入 Cloud Storage 之前剝除 EXIF。

**照片用途區分**:
- **收藏照片**(`player_summon_photo`):玩家自願拍攝,無需審核流程,僅走內容審核掃描
- **任務驗證照片**(`quest_verification_photo`):需經 `quest_photo_submissions` 管理審核狀態

**審核觸發時機**:採**上傳當下自動掃描**(呼叫 Vertex AI Vision Safe Search),而非事後排程或僅依賴使用者檢舉。僅 `player_summon_photo` 與 `quest_verification_photo` 需掃描,系統素材(`landmark_photo` 等)不需要。

**審核與任務驗證的順序關係**:`content_moderation_flags` 檢查必須排在 `quest_photo_submissions` 驗證邏輯之前——若照片有未處理的 `pending` 審核標記,任務驗證流程需暫緩,不進行辨識判定(見第12節)。

---

## 5. Body 模組 — 對話紀錄與主線進度

```sql
CREATE TABLE dialogue_turns (
    turn_id                 BIGSERIAL PRIMARY KEY,
    player_id                UUID REFERENCES players(player_id),
    spirit_id                  TEXT REFERENCES spirits(spirit_id),
    role                      TEXT NOT NULL,        -- 'player' / 'spirit'
    content                   TEXT NOT NULL,
    resonance_value_at_time     INT,
    story_beat_id              TEXT,                 -- 值關聯 → brain.story_beats,不建 FK
    created_at                 TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_dialogue_player_spirit_time ON dialogue_turns(player_id, spirit_id, created_at);

CREATE TABLE players_story_progress (
    player_id           UUID REFERENCES players(player_id),
    beat_id                TEXT,                     -- 值關聯 → brain.story_beats,不建 FK
    triggered_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (player_id, beat_id)
);
```

**保留策略**:`dialogue_turns` 目前為永久保留、不設計封存/清理機制(MVP 階段決定)。日後資料量成長至百萬筆規模時,需另行評估分區或封存策略,目前索引足以應付 MVP 到中期規模查詢。

> 但「永久保留」不等於「無法刪除」。玩家刪除帳號時必須能連帶清掉 `dialogue_turns` 與 `brain.memory_embeddings`,這條刪除路徑目前**尚未實作**,列在第13節。

**用途**:此表除了作為對話日誌,亦是 `brain.memory_embeddings` 排程萃取記憶的資料來源(見第10節)。

**`dialogue_turns` 不存座標**。哪一次對話發生在哪裡,由 `spirit_id` 表達(靈魂本身就是地點),不需要也不允許另存玩家當下座標。

---

## 6. Body 模組 — 配額(Usage Tier)

對應 AC8 與《業務規則與API契約》§9.5。

```sql
-- 分級定義(例如:封測期、正式公開期,未來也可以是免費/付費)
CREATE TABLE usage_tiers (
    tier_id      VARCHAR(32) PRIMARY KEY,   -- 'closed_beta', 'public_default', ...
    display_name VARCHAR(64) NOT NULL,
    is_default   BOOLEAN NOT NULL DEFAULT FALSE,
    created_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 「只能有一筆 is_default = TRUE」。BOOLEAN DEFAULT FALSE 擋不住兩筆都是 TRUE,
-- 這個 partial unique index 才是真的在保證那句話。
CREATE UNIQUE INDEX uq_usage_tiers_default ON usage_tiers(is_default) WHERE is_default;

-- 每個分級底下,各種資源類型的配額
CREATE TABLE usage_tier_limits (
    tier_id       VARCHAR(32) NOT NULL REFERENCES usage_tiers(tier_id),
    resource_type VARCHAR(32) NOT NULL,
    limit_value   INTEGER NOT NULL,
    PRIMARY KEY (tier_id, resource_type)
);
```

**關鍵決定:用「一個 tier 對多筆 limit」的正規化設計,不是每個資源類型開一個欄位。** 欄位式設計以後每加一種新資源就要 `ALTER TABLE`;正規化之後,加新資源類型只是**多寫一筆資料**,不用動 schema、不用重新部署。`landmark_recognition_daily` 就是這個設計成立的實例——《地標辨識》文件加這個配額時,一行 `INSERT` 解決。

### `closed_beta` 初始值

| resource_type | 意義 | 計算週期 | 初始值 |
|---|---|---|---|
| `dialogue_calls_daily` | 每日 `/dialogue` 呼叫次數 | 每日(Asia/Taipei 午夜) | 50 |
| `daily_tokens` | 每日 Gemini token 用量(input+output) | 每日 | 150,000 |
| `prompt_max_chars` | 單次 `user_input` 字數上限 | 每次請求(非累計) | 500 |
| `api_rate_per_minute` | 每分鐘 API 呼叫次數 | 每分鐘滑動窗口 | 6 |
| `landmark_recognition_daily` | 每日地標辨識呼叫次數 | 每日 | 10 |

這組數字**只是起點**,封測有實測數據後直接改 `usage_tier_limits` 的資料即可,不用重新部署。

**`closed_beta` 這筆 tier 與上表的 limits 寫在 Alembic migration 裡,不放在 seed script。** 因為 `players.usage_tier_id` 是 NOT NULL,沒有它的資料庫連匿名玩家都建不出來——這是 schema 能不能運作的前提,不是範例資料。

### 執行層:計數器在 Redis,不落地 Postgres

**所以沒有 `quota_usage` 表。**

```python
def check_and_consume_quota(player_id: str, resource_type: str, amount: int = 1) -> None:
    """
    在 B4 安全檢查之前呼叫(dialogue API 的第一道關卡)。
    超過配額時 raise QuotaExceededError,由 API 層轉成 429。

    Redis key:quota:{player_id}:{resource_type}:{YYYY-MM-DD, Asia/Taipei}
    每日型用 INCRBY + TTL 到當天 Asia/Taipei 午夜;
    api_rate_per_minute 用滑動窗口,key 加分鐘級時間戳、TTL=60 秒。
    """
```

- limit 數值本身變動頻率低,應用啟動時載入記憶體快取即可,不需要即時通知機制。
- `prompt_max_chars` 是唯一不用 Redis 計數器的一種——它是單次請求的靜態檢查(`len(user_input) <= limit`)。**且此檢查必須排在扣配額之前**(AC8.3:超長輸入回 422 且不消耗配額)。
- AC8.2:150m 隔空聊天與 50m 召喚後對話**共用同一個 Redis key**,不是兩組計數器。

### 已知取捨:Redis 掉了會 fail-open

計數器只在 Redis,Redis 重啟或被 evict 時當日用量歸零,配額等於暫時消失。**封測期接受這個風險**,因為玩家數只有幾十人、單日最壞損失有限。

> ⚠️ 但接受的前提是 **GCP 專案級預算警示已經設定**。`daily_tokens` 是成本控制,而 SDD 把「AI 對話成本」列為 🔴 高風險;預算警示是比 Redis 更可靠、且免費的最後一道防線。玩家規模擴大後應改為 `daily_tokens` 額外落地 Postgres(見第13節)。

### 已知取捨:`daily_tokens` 必然事後記帳

其他配額都能事前檢查,但 token 用量要等 Gemini 回來才知道。因此:

- **`daily_tokens` 的判定是「上一次呼叫結束後已經超過,這一次就擋」**,不是「這一次會不會超過」。玩家一定能超額一點點,最多超出一次呼叫的量。這是結構限制,不是 bug——寫測試時不要去驗一個做不到的行為。
- 前置條件:`gemini.py` 目前只回傳字串,**尚未把 `usage_metadata` 的 token 數回傳出來**。要加一個回傳欄位(與現有的 `last_truncated` 同一個位置)才能記帳。

### 429 回應

回應內容要說明**哪一種配額用完**與**下次重置時間**。前端會用這個資訊做符合人設的提示(「今天已經聊得很盡興了,明天再回來找我吧」),而不是把英文錯誤訊息丟給玩家——這句 fallback 文案由腦袋模組準備。

---

## 7. Brain 模組 — 靈魂三層資料

> 本章取代舊的 `brain.persona_cards`。三層拆分是新設計,但 `persona_cards` 的**版本管理與人工審核**機制原樣保留在 `character_personas` 上——那不是舊設計的包袱,是人格內容的安全機制。

```sql
-- City 層:全城共用的基調,不需 RAG,直接整段注入 prompt
CREATE TABLE brain.city_souls (
    city_id           TEXT PRIMARY KEY,
    name              TEXT NOT NULL,
    macro_history_summary TEXT NOT NULL,
    core_tone_descriptors TEXT[],
    shared_values     TEXT[],
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Landmark 層:掛在 city 底下,史實資料
CREATE TABLE brain.landmark_souls (
    landmark_id       TEXT PRIMARY KEY,
    city_id           TEXT REFERENCES brain.city_souls(city_id),
    district_id       TEXT REFERENCES brain.districts(district_id),  -- 可為NULL,見第9節
    name              TEXT NOT NULL,
    founding_facts    JSONB NOT NULL,   -- [{year, event, detail}, ...]
    key_events        JSONB,
    cultural_significance TEXT,
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- 角色身分層:穩定的識別,其他表引用這裡
CREATE TABLE brain.characters (
    character_id      TEXT PRIMARY KEY,
    landmark_id       TEXT NOT NULL REFERENCES brain.landmark_souls(landmark_id)
);

-- Character 人格層:有版本、需人工審核。實際玩家互動的人格,優先於 city/landmark 基調
CREATE TABLE brain.character_personas (
    character_id      TEXT NOT NULL REFERENCES brain.characters(character_id),
    version           INT NOT NULL,
    archetype         TEXT NOT NULL,
    speech_style      TEXT NOT NULL,
    personality_traits TEXT[],
    values            TEXT[],
    taboos            TEXT[],
    tone_override     TEXT,             -- 允許明確聲明「不遵循 city 基調」的原因
    reviewed_by       TEXT NOT NULL,
    reviewed_at       TIMESTAMPTZ NOT NULL,
    active            BOOLEAN NOT NULL DEFAULT false,
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (character_id, version)
);

-- 一個角色同時只能有一個生效版本
CREATE UNIQUE INDEX uq_character_personas_active
    ON brain.character_personas(character_id) WHERE active;
```

**為什麼多一張 `brain.characters`**:人格有版本,主鍵是 `(character_id, version)`。`spirits`、`resonance_unlockables`、`story_beats` 要指的是「這個角色」而不是「這個角色的第 3 版」,需要一個穩定的單欄位主鍵可以引用。沒有這張表的話,那些外鍵無法成立。

**設計原則**:

- 史實層(facts)決定角色「知道什麼」,人格層(persona)決定角色「怎麼說話」。兩者刻意分離,避免生成時互相干擾。
- **Character 層人格永遠優先於 City/Landmark 基調**——city/landmark 只提供背景音,不強制角色語氣。
- 三層皆為**靜態設定資料**,資料量有界,不採用向量檢索(RAG),直接整段注入 prompt 更穩定、更省成本。
- **人格是 append 新版本,不是就地覆寫。** 改人格 = `INSERT` 一筆新 `version`,舊版留著;審核通過才把 `active` 換過去。`updated_at` 式的就地覆寫沒有歷史、沒有審核人記錄,出事時無法回溯是誰在什麼時候改的。

### 🔒 `active` 只能人工 flip

**程式碼裡沒有任何路徑會自動把 `active` 設為 true。** 這是 CONTEXT.md 對「人格卡」的定義邊界,不是技術限制。新版本一律以 `active = false` 寫入,由人工審核流程另行 flip。

### 🔒 `taboos` 是下限,不是建議

宗教場域的禁忌(龍山寺目前四條:不代替神明給予指示或應許、不預測個人吉凶姻緣財運、不比較宗教或信仰優劣、不給具體醫療法律投資建議)是**安全底線**。敘事審查可以往上加,**不可移除**。新版本人格若少了既有的 taboo 條目,審核流程應視為不通過。

---

## 8. Brain 模組 — 共鳴解鎖與主線節點

```sql
CREATE TABLE brain.resonance_unlockables (
    unlock_id         TEXT PRIMARY KEY,
    character_id      TEXT REFERENCES brain.characters(character_id),
    min_resonance     INT NOT NULL,        -- 對照固定門檻
    unlock_type       TEXT NOT NULL,       -- 'memory_fragment' / 'secret_story'
    content            TEXT NOT NULL,
    UNIQUE(character_id, min_resonance, unlock_type)
);

CREATE TABLE brain.story_beats (
    beat_id            TEXT PRIMARY KEY,
    character_id       TEXT REFERENCES brain.characters(character_id),
    sequence_order      INT NOT NULL,          -- MVP採線性排序,不用前置節點鏈
    trigger_condition   TEXT NOT NULL,          -- 描述性文字,實際判斷邏輯在 body 狀態機
    narrative_directive TEXT NOT NULL,
    one_time            BOOLEAN NOT NULL DEFAULT true,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

**固定 Resonance 門檻**(不存 DB,寫死於 code,全角色共用)。數值由 AC5.3 定義:

```python
# 門檻 10 / 40 / 100 出自 AC5.3,不可自行調整。
# 階段名稱與語氣指令對照見第11節 TONE_INSTRUCTIONS。
RESONANCE_STAGES = [
    (0,   "疏離"),
    (10,  "熟悉"),
    (40,  "信任"),
    (100, "摯友"),
]
```

給分規則(AC5.1 / AC5.2):任務完成 `+20`,地標相遇收藏 `+10`(每個地標一次)。

**主線劇情設計原則**:
- 主線進度為**事件驅動**,非數值驅動,完全獨立於 resonance 機制(不做雙重門檻)
- 主線觸發時**覆蓋** daily_event 的語氣層,但**保留** relationship_stage 的深度限制(即使觸發主線,玩家仍看不到超出目前信任階段的深層內容)
- `trigger_condition` 的實際判斷邏輯在 body 狀態機,brain 只負責「被觸發後該怎麼講」
- MVP 階段以 `sequence_order` 做**單一角色內**的線性排序;**跨角色的分支/依賴**已於第9節以 `prerequisite_beat_ids` 實作,兩者並存不衝突

---

## 9. Brain 模組 — 跨角色主線案件與地理圍欄

> 本章為「萬華區劇本」設計討論後新增,支援多地標共用一條主線、跨角色線索依賴、以及區域級地理觸發。

```sql
CREATE EXTENSION IF NOT EXISTS postgis;

CREATE TABLE brain.districts (
    district_id     TEXT PRIMARY KEY,
    city_id           TEXT REFERENCES brain.city_souls(city_id),
    name              TEXT NOT NULL,
    center_lat         NUMERIC(9,6),              -- 地圖UI顯示概略中心點,非判斷依據
    center_lng         NUMERIC(9,6),
    radius_meters      INT,                       -- 地圖UI顯示概略半徑,非判斷依據
    boundary          GEOGRAPHY(POLYGON, 4326)    -- 實際多邊形邊界,精確地理圍欄判斷依據
);

CREATE INDEX idx_districts_boundary ON brain.districts USING GIST(boundary);

CREATE TABLE brain.story_arcs (
    arc_id                    TEXT PRIMARY KEY,
    title                       TEXT NOT NULL,
    district_id                  TEXT REFERENCES brain.districts(district_id),
    intro_document_title           TEXT,       -- 例如「阿明留下的信」
    intro_document_content          TEXT,       -- 信件全文
    intro_document_asset_id          UUID,      -- 值關聯 → public.media_assets,不建 FK
    summary                       TEXT
);

ALTER TABLE brain.story_beats
    ADD COLUMN arc_id                TEXT REFERENCES brain.story_arcs(arc_id),
    ADD COLUMN prerequisite_beat_ids   TEXT[],   -- 可跨角色引用其他 beat_id
    ADD COLUMN required_item_ids       TEXT[],   -- 需玩家 player_inventory 持有的 item_id
    ADD COLUMN contingency_notes       TEXT;      -- 特定情境防呆話術
```

**設計原則**:

- **districts 是敘事分組標籤,不是遊戲地理主結構**:`city → landmark` 垂直結構不變,`districts` 是額外掛在 landmark 上的分組。非所有地標都需要屬於某個 district。
- **多邊形邊界為準,概略圓形僅供 UI 顯示**:`center_lat/lng/radius_meters` 保留給地圖畫概略範圍參考,**實際判斷一律以 `boundary` 多邊形為準**,用 PostGIS `ST_Contains`,避免圓形範圍造成誤觸發或漏觸發。多邊形資料建議取政府開放資料平台的行政區界 GeoJSON 轉換匯入,不需手動描點。
- **跨角色依賴一致處理**:`prerequisite_beat_ids` 不限制只能引用同一 `character_id` 底下的 beat。判斷邏輯統一為「陣列內所有 beat_id 是否都已存在於該玩家的 `players_story_progress`」,同角色與跨角色走同一套規則,不需要為劇本寫例外邏輯。
- **道具作為解鎖條件**:`required_item_ids` 檢查玩家 `player_inventory` 是否持有指定 `item_id`。
- **防呆話術獨立欄位**:`contingency_notes` 承載特定情境的預備回應,不用整段塞進 `narrative_directive` 混淆主敘事重點。
- **信件/道具插圖走既有輕量路徑**:`story_arcs.intro_document_asset_id` 與 `player_inventory.illustration_asset_id` 皆指向 `media_assets`——圖檔存 Cloud Storage、CDN 給 URL、`media_assets` 只存路徑與 metadata。前端(Unity S8 背包 UI)僅需標準圖片下載,不涉及 F1–F8 的 3D 角色 Addressables 資產管線。

> ⚠️ 地理圍欄判斷**不留下任何紀錄**。`check_player_in_district()` 收到座標、算完、回傳布林值,座標即丟。「玩家曾在某時刻進入某區域」這件事只以 `player_inventory` 裡多了一件 arc 信件的形式存在,那是遊戲進度,不是位置紀錄。

```python
def check_player_in_district(player_lat: float, player_lng: float, district_id: str) -> bool:
    """
    SELECT ST_Contains(boundary, ST_SetSRID(ST_MakePoint(lng, lat), 4326))
    FROM brain.districts WHERE district_id = %s
    座標不寫入任何資料表。
    """

def grant_arc_intro_document(player_id: str, arc_id: str):
    """
    玩家進入 district 範圍且尚未持有該 arc 的 intro 道具時呼叫,
    寫入 player_inventory(item_type='story_document'),
    依賴 uq_inventory_player_item 唯一約束避免重複發放
    """

def check_beat_unlockable(player_id: str, beat_id: str) -> bool:
    """
    prerequisite_beat_ids 皆存在於 players_story_progress,
    且 required_item_ids 皆存在於 player_inventory,兩者皆滿足(或皆為空)才 True
    """
```

---

## 10. Brain 模組 — 記憶系統

```sql
CREATE TABLE brain.memory_embeddings (
    memory_id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id             UUID NOT NULL,          -- 值關聯 → public.players,不建 FK
    spirit_id               TEXT NOT NULL,        -- 值關聯 → public.spirits,不建 FK
    source_turn_id         BIGINT,                -- 值關聯 → public.dialogue_turns,不建 FK
    summary_text          TEXT NOT NULL,
    embedding             VECTOR(768),
    source                 TEXT NOT NULL,         -- 'dialogue_summary' / 'nightly_batch'
    importance_score        FLOAT NOT NULL DEFAULT 0.5,
    created_at             TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX ix_memory_embeddings_embedding_cosine
    ON brain.memory_embeddings USING ivfflat (embedding vector_cosine_ops);

CREATE INDEX ix_memory_embeddings_player_spirit
    ON brain.memory_embeddings(player_id, spirit_id);
```

**維度是 768,不是 1536。** 768 是 Vertex AI `text-embedding` 系列的輸出維度;1536 是 OpenAI 的。LLM 已確定走 Vertex(#8),embedding 沒有理由跨到另一家——那會多一組長期憑證(違反 ADR-0003)、多一個計費來源。換模型導致維度改變是**破壞性 migration**,不是改個常數。

**ivfflat 是近似最近鄰索引**:它把向量分群,查詢時只掃其中幾群,所以可能漏掉真正的前 K 名。這對記憶檢索是可接受的取捨,但檢索函式的正確性測試不能依賴它。

**設計原則**:採「可檢索記憶」而非「全量注入 context」,控制 LLM token 成本。Redis 處理短期 session 記憶(無需 DB 表),`memory_embeddings` 處理長期可檢索記憶,兩者互補。

---

## 11. Pydantic 介面與 Context 組裝邏輯

```python
from pydantic import BaseModel
from typing import Optional

class SoulContext(BaseModel):
    """靜態身份層,由 assemble_soul_context() 產出,快取於 Redis(TTL 24hr, key=character_id:version)"""
    city_summary: str
    landmark_facts: str
    character_archetype: str
    character_speech_style: str
    character_traits: list[str]
    character_values: list[str]
    character_taboos: list[str]
    tone_directive: str
    assembled_prompt_block: str

class DynamicContext(BaseModel):
    """動態層,即時組裝或短TTL快取(key需納入 resonance_value + event_date)"""
    relationship_stage: str
    unlocked_fragments: list[str]
    tone_instruction: str
    today_event_summary: Optional[str] = None
    event_mood_modifier: Optional[str] = None

class SessionContext(BaseModel):
    """Redis 短期 session 記憶"""
    recent_turns: list[dict]
    player_mood_signal: Optional[str] = None

class StoryBeatContext(BaseModel):
    beat_id: str
    narrative_directive: str
```

> `SoulContext` 的 Redis key **要含 `version`**。人格是版本化的,審核 flip 到新版本之後,舊的快取必須自然失效——否則會出現「已經換人格了但玩家還在跟舊版說話」,而且沒有任何錯誤訊息。

**TONE_INSTRUCTIONS 對照表**(固定門檻對應語氣指令,寫死於 code):

```python
TONE_INSTRUCTIONS = {
    "疏離": "你對這位訪客還不熟悉,保持禮貌但有距離感,不主動分享私人記憶。",
    "熟悉": "你開始對這位訪客有好感,語氣放鬆一些,願意多聊幾句城市的日常故事。",
    "信任": "你已經信任這位訪客,語氣親近,願意分享一些平常不會提起的往事。",
    "摯友": "你把這位訪客當作摯友,語氣真誠溫暖,願意坦露最深層的記憶與情感。",
}
```

---

## 12. 核心後端邏輯函式

```python
def get_relationship_stage(resonance_value: int) -> str:
    """門檻 10/40/100(AC5.3)。永遠從 value 重算,不存欄位。"""
    stage = "疏離"
    for threshold, name in RESONANCE_STAGES:
        if resonance_value >= threshold:
            stage = name
    return stage

def award_resonance(player_id, spirit_id, source_type: str, source_id: str, amount: int):
    """
    先寫 resonance_events、撞到 UNIQUE 就當作重複,再加值。
    不是「先查帳本、沒有才寫」——那個順序擋不住並行請求(見第2節)。
    """

def assemble_soul_context(character_id: str) -> SoulContext:
    """join city_souls / landmark_souls / characters / character_personas(active 版本)"""

def assemble_dynamic_context(
    character_id: str,
    resonance_value: int,
    daily_event: Optional[dict] = None,
) -> DynamicContext:
    """每次對話呼叫,依 resonance 門檻篩選已解鎖內容,daily_event 由 body 傳入"""

def compose_full_context(
    soul_context: SoulContext,
    relationship_stage: str,
    daily_event: Optional[dict],
    story_beat: Optional[StoryBeatContext] = None,
) -> str:
    """
    組裝規則:
    - 史實層(city+landmark)不決定語氣,只提供背景知識
    - character 人格層優先於 city/landmark 基調
    - 主線觸發時,忽略 daily_event 語氣層,但保留 relationship_stage 深度限制
    - 無主線時,daily_event 疊加在 relationship_stage 語氣之上,不覆蓋深度軸
    """

def check_and_consume_quota(player_id: str, resource_type: str, amount: int = 1) -> None:
    """dialogue API 第一道關卡,在 B4 安全檢查之前。超過則 429(見第6節)"""

def get_or_issue_daily_quest(player_id: str, quest_id: str) -> dict:
    """惰性發放:玩家當天首次觸發時才建立 daily quest_progress 紀錄(Asia/Taipei 的今天)"""

def complete_quest(progress_id: int):
    """
    reward_type == 'resonance' -> award_resonance(..., amount=20)
    reward_type == 'item'      -> create_inventory_entry()
    quest_type == 'story' and story_beat_id 存在 -> mark_story_beat_triggered()
    兩條路徑(daily resonance / story)完全獨立,互不限制
    """

def upload_media_asset(file_data: bytes, asset_type: str, owner_id: str) -> dict:
    """
    先剝除 EXIF(內含拍攝座標),再寫入 Cloud Storage。
    上傳當下呼叫 Vertex AI Vision Safe Search 掃描
    (僅 player_summon_photo / quest_verification_photo 需掃描)
    有疑慮則寫入 content_moderation_flags,status='pending'
    """

def verify_quest_photo_submission(submission_id):
    """先檢查 asset 是否有 pending 的 content_moderation_flags,有則暫緩驗證"""

def check_player_in_district(player_lat: float, player_lng: float, district_id: str) -> bool:
    """PostGIS ST_Contains,座標不落地,詳見第9節"""

def grant_arc_intro_document(player_id: str, arc_id: str):
    """玩家進入 district 且未持有該 arc intro 道具時,寫入 player_inventory,詳見第9節"""

def check_beat_unlockable(player_id: str, beat_id: str) -> bool:
    """檢查 prerequisite_beat_ids 與 required_item_ids 是否皆滿足,詳見第9節"""
```

---

## 13. 尚未定案 / 延伸事項

以下項目已於討論中識別,但**刻意延後**,以控制 MVP 開發複雜度。日後擴充時多為**新增欄位/新增表**,不需重構既有結構:

| 項目 | 現況 | 之後如何擴充 |
|---|---|---|
| **玩家刪除帳號的資料清除路徑** | 尚未實作。`dialogue_turns` / `memory_embeddings` 沒有刪除入口 | 需一支跨 schema 的刪除流程;因為沒有跨 schema FK,不會自動 cascade,必須顯式處理 |
| **`daily_tokens` fail-open** | 計數器只在 Redis,Redis 掉了配額暫時消失。封測期接受,倚賴 GCP 專案級預算警示 | 玩家規模擴大後改為額外落地 `daily_token_usage(player_id, usage_date, tokens)` |
| **Gemini token 用量回傳** | `gemini.py` 只回字串,沒有回傳 `usage_metadata` | 加回傳欄位(與 `last_truncated` 同位置),`daily_tokens` 記帳才成立 |
| Spirit 人格變體(1對多) | `spirits.character_id` 為 1對1約束 | 移除 UNIQUE 約束,改中介表 |
| 主線覆蓋語氣的可配置性 | 寫死「主線永遠覆蓋 daily_event」 | 新增 `overrides_tone` 欄位(story_beats) |
| 任務複合獎勵 | `reward_type` 僅支援單一獎勵類型 | 待確認是否需要,需改為陣列/巢狀結構 |
| Daily event 依語氣強度分階段收斂 | daily_event 統一疊加,不分 relationship_stage 強度 | 於 `compose_full_context` 加入依 stage 調整強度的邏輯 |
| 成就/里程碑系統 | 未設計 | 新增 `achievements` / `player_achievements` 表 |
| 推播歷史紀錄 | 僅有 `push_subscriptions`(訂閱狀態),無發送歷史 | 新增 `notification_log` 表 |
| `daily_event_cache` / `push_subscriptions` 完整欄位 | 沿用既有定義,本輪未重新設計 | 詳見原始 WBS-API 文件 |
| 萬華區多邊形邊界資料來源 | 結構已定,尚無實際 GeoJSON 匯入 | 需向政府開放資料平台取得台北市行政區界資料並轉換匯入 |
| 一個 district 是否可對應多條並行主線 | 假設 1對多,單一劇本情境下實為1對1 | 若未來同區域並行多條案件,需確認 UI 呈現與觸發優先順序 |
| Unity 前端道具/背包 UI 顯示規格 | 走 `media_assets` URL 標準載入路徑 | 細節顯示規格留給 S8 UI 工作包設計 |

---

## 附錄:本版與 code 現況的差異(遷移清單)

`citysoul-backend` 現有 code 與本文件的落差,#31 Alembic 第一版 migration 要處理的:

| 項目 | code 現況 | 本文件 | 動作 |
|---|---|---|---|
| `spirits` 主鍵 | `place_id` | `spirit_id` | rename(API 路徑 `{placeId}` 不變) |
| 半徑欄位 | `summon_radius_m` / `sense_radius_m` | `..._meters` | rename |
| 座標型別 | `Float` | `NUMERIC(9,6)` | alter type |
| `players` | `device_id` + `account_id` | 見第1節 | 加欄位、加 CHECK、加 partial index |
| `resonance.stage` | 有欄位 | 不存 | drop column |
| `quest_progress` 主鍵 | `(player_id, quest_id)` | `progress_id BIGSERIAL` | 改主鍵、加 `issued_date` 與兩個 partial index |
| `brain.persona_cards` | 存在,B3 loader 在吃 | 由三層表取代 | 建三層表、重寫 loader 與 seed、drop 舊表 |
| embedding 維度 | 768 | 768 | 不變 ✓ |
| 配額表 | 不存在 | 第6節 | 新建,`closed_beta` 資料寫在 migration 裡 |
| PostGIS | 未啟用 | 需要 | `CREATE EXTENSION postgis`,Cloud SQL 需確認可用 |
