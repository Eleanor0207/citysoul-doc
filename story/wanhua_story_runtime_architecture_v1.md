# 萬華主線運行架構 v1（景觀拍攝辨識）

> **狀態：方案 A／完整工程交付草案。**本文件自行包含主線、任務、景觀辨識、資料、API、Unity、隱私、MVP 與驗收規格。
>
> **權威邊界**：產品與架構決策以 `../SDD_v2.2_Unity_3D.md` 為準；現行 API 以 `../../citysoul-backend/contracts/openapi.json` 為準；資料庫實況以 `../../citysoul-backend/migrations/versions/` 為準。本文件不取代三者。

## 1. 結論

不新增遊戲引擎、不更換資料庫。萬華主線應建立在既有定案上：**Unity 6** 呈現遊戲、**FastAPI Body** 判定劇情狀態、**PostgreSQL** 保存長期真相、**Redis** 保存短暫狀態，Brain／LLM 只生成受人格卡與 Beat 約束的補充敘事。

### 1.1 區域主線與玩家體驗規格〔定案〕

- 每個行政區有一個固定的 arc 起始地標；萬華為 `longshan_temple`。**入口文件由當次行政區圍欄判定觸發**（`brain.districts.boundary` 多邊形內含），玩家不必先接近該地標。
- Arc 的所有必要 Beat、任務與結局必須在同一 `district_id` 完成。**不得**由其他行政區的到訪、任務、商家或故事作為前置條件。
- 劇情進度永久保存、可跨日恢復、沒有截止時間。Unity 只顯示「可繼續」與下一站理由；不顯示倒數或強制導航。
- 下一站方向提示為玩家手動開啟的 UI 輔助，回應只能提供同區地標；伺服器不輸出跨區必經路徑。
- 各靈魂親近度沿用每個 `spirit_id` 各自的共鳴值與既有門檻，不另建可扣分的好感系統；親近度不可作為其他區域或主線的路徑鎖。
- 親近度只額外開啟該靈魂的**可選導覽短札與細節**：龍山寺偏修復與日常照看、紅樓偏空間與用途變遷、剝皮寮偏居住與街區記憶。核心歷史卡、主線對話、任務與所有區域入口必須對所有玩家可見。

```text
Unity 6（地圖、3D 靈魂、對話、年代簿）
                │
                ▼
FastAPI Body（到場／任務／選項／Beat 的確定性判定）
       │                         │
       ▼                         ▼
PostgreSQL（長期真相）      Brain（可選敘事包裝）
       │                         │
       └──────────────┬──────────┘
                      ▼
             Unity 呈現下一個節點

Redis：session、短期對話、配額、暫時狀態；不保存劇情真相。
```

## 2. 引擎與客戶端責任

| 層 | 選型／責任 | 不負責 |
| --- | --- | --- |
| Unity 6 | 地圖、3DoF 實境疊層、3D 靈魂、對話 prefab、**年代簿 UI 面板（非獨立場景，SDD §5.2 場景清單不變）**、任務 UI、離線呈現快取、玩家手動開啟的同區方向提示 | 判定玩家是否完成 Beat、判定任務成功、決定可選選項或以本機資料開啟跨區 arc。 |
| Unity 對話資料載入器 | 顯示後端 DTO 中的 `line`／`choice`／`command`；保存當前 UI node 與已讀文字 | 自行解讀 YAML 條件或繞過後端寫入玩家進度。 |
| YAML 劇本 | 供內容撰寫、逐字審核、版本控制與匯入 | 作為客戶端的長期真相來源。 |

不引入真 AR、AR Foundation、ARCore／ARKit 或 Geospatial。劇情層沿用 Unity 的對話 UI，不因使用 YAML 或日後採用 Ink/Yarn 原型工具而改變「後端擁有跨裝置、跨日狀態」的原則。

### 2.1 對話閱讀體驗（MVP 必要）

劇本採「分支後匯流」：選擇記錄旗標並改變後續台詞、短札與可選細節，不能製造不可逆的錯過結局。Unity 對話層須提供文字速度、首次點按完成逐字顯示、第二次前進、只跳過已讀文字、選項自動暫停、最近 100–200 句回顧與可中斷的自動播放。跨日／跨裝置的真相仍是後端 Beat／選擇／物件；客戶端另保存當前 UI node、已讀文字與本機閱讀設定，重新讀取狀態後不得用它們推進任務。

## 3. 既有後端能力與缺口

| 領域 | 已有且可重用 | 劇情尚缺 |
| --- | --- | --- |
| 玩家 | `POST /api/v1/players`、匿名 player／session、帳號綁定基礎 | 劇情選擇的持久紀錄。 |
| 到場 | `/sense`、`/summon`、三種 token 邊界 | 劇情只在有效到場／任務完成後推進的編排。 |
| 對話 | `/spirits/{place_id}/dialogue`、人格卡、配額與安全路徑 | 回傳目前 story Beat 與已核可選項。 |
| 任務 | `/quests/daily`、`/quests/{id}/complete`、`quest_progress`、共鳴去重 | 多地標故事任務目錄與輪播。 |
| 劇情資料 | `brain.story_arcs`、`brain.story_beats`、`resonance_unlockables`；0013／0015 migrations 已建表 | Beat 讀寫狀態機、內容匯入器、內容審核版本與對外 API。 |
| 玩家收藏 | `player_inventory`、`media_assets`、`players_story_progress` | 玩家對主線選項、個人化年代簿頁的耐久資料。 |

`players_story_progress` 是「Beat 是否完成」的唯一真相來源；`dialogue_turns.story_beat_id` 僅是日誌，不能反向推進故事。

## 4. 主線資料模型

### 4.1 可直接使用的表

| 資料 | 表 | 萬華主線用途 |
| --- | --- | --- |
| Arc 與 Beat | `brain.story_arcs`、`brain.story_beats` | 〈回家的畫〉、前置鏈、角色、敘事指令。 |
| 共鳴解鎖 | `brain.resonance_unlockables` | 與主線平行的角色關係／解鎖內容，不替代 Beat。 |
| 主線任務 | `public.quests`、`quest_progress` | 三地標觀察任務與完成狀態。 |
| 劇情物件 | `player_inventory`、`media_assets` | 留給街的信、回家的畫、年代簿插圖。 |
| 劇情進度 | `players_story_progress` | 每位玩家踩過的 Beat。 |
| 對話與記憶 | `dialogue_turns`、玩家私有摘要 | 回顧與角色連續性；不得混入世界記憶。 |

### 4.2 必要新增：玩家敘事選擇

現有進度表不能保存 `story_focus`、`reveal_lens`、`ending_mark`。建議新增最小表：

```text
players_story_choices          # 命名對齊既有 players_story_progress
  player_id        UUID FK players
  arc_id           TEXT
  choice_key       TEXT       # story_focus / reveal_lens / ending_mark
  choice_value     TEXT
  selected_at      TIMESTAMPTZ
  PRIMARY KEY (player_id, arc_id, choice_key)
```

它只記敘事選擇；不存座標、影像、裝置硬體資訊或由選項推論出的敏感身分。

個人化年代簿由這些選擇、已完成 Beat 與每個靈魂既有共鳴值組合產生；它只能改變文案、收藏排序與可見回顧，不能作為其他 arc 的啟動或阻擋條件。

**MVP 輸出定案：**完成主線時由 `story_focus`、`reveal_lens`、`ending_mark` 三次選擇，組合一頁不超過 90 字的預寫個人化年代簿短札。MVP 不實作每日重訪任務、餘白輪播、其 API 或排程；既有共鳴只可微調該頁用詞／排序。

### 4.3 必要新增：內容版本與人工啟用

人格卡有版本、人工審核與 `active` 流程；`story_beats.narrative_directive` 也會直接影響玩家內容，應有同級治理。正式方案可採「story content version」父物件，包含：

- `content_version`
- `review_status`、`reviewed_by`、`reviewed_at`
- `active`
- 劇本 YAML 的來源版本／雜湊

在此方案落地前，故事資料只能走「Lead 人工覆核後才匯入」的流程，且不得自行宣告 active。

## 5. API 最小增量（提案）

既有 API 不重做。劇情新增**五支**：三支 arc 層級的讀寫邊界，加上任務執行期的兩支寫入（§9.4 的動作表用到它們）。

| Endpoint | 行為 | 回應重點 |
| --- | --- | --- |
| `GET /api/v1/story-arcs/{arc_id}/state` | 取得玩家可見劇情狀態，不產生副作用 | current beat、道具、可選選項、同區下一目標、年代簿摘要與每靈魂親近度。 |
| `POST /api/v1/story-arcs/{arc_id}/choices` | 提交一個已開放選項；後端驗證目前 Beat 與選項合法性，再持久化 | 已接受的選擇、後續 line／command、更新後狀態。 |
| `GET /api/v1/story-arcs/{arc_id}/chronicle` | 取得已解鎖信件、畫作、資訊卡、餘白短札 | 僅回該玩家已解鎖內容。 |
| `POST /api/v1/quests/{quest_id}/observations` | 確認一個白名單觀察點；驗證 token、地標、前置與去重 | 已完成點位、是否可進入反思。 |
| `POST /api/v1/quests/{quest_id}/turn-in` | 交件；同一交易寫 inventory＋beat＋choice | 物件、Beat、保底台詞、同區下一站。 |

> 既有的 `POST /quests/{quest_id}/complete` 是每日任務的完成端點，語意與 story 交件不同（後者要在同一交易寫物件與 Beat）。是否併用同一支由實作時決定，但**不可讓每日任務誤觸 story 的發物路徑**。

### 5.1 寫入順序

```text
Unity 提交 choice_id／完成 story quest
  → FastAPI 驗證 session／encounter token 與目前狀態
  → transaction 內寫 players_story_progress、inventory、players_story_choices
  → 以唯一約束／ON CONFLICT 保證重送不重複發放
  → commit 成功
  → （可選）Brain 以 persona + taboo + current beat 產生包裝文字
  → 回傳 Unity
```

任務與 Beat 完成永遠由確定性規則判定；LLM 不能決定「玩家是否完成劇情」。任何 Brain 失敗都不能回滾已完成的 Body 寫入，必須回退人工預寫台詞。

### 5.2 區界與方向提示的後端規則

1. `arc.district_id` 與入口地標的 `district_id` 必須相同；內容匯入器拒絕不一致的資料。
2. Arc 入口只能由 `arc.district_id` 圍欄的**首次成功內含判定**建立；判定在伺服器端（不下發多邊形給客戶端），原始座標算完即丟，只持久化「已發放入口文件」的結果。重複觸發由 `player_inventory` 的唯一索引擋下。

   **前置狀態**：`data/districts/wanhua.geojson`、PostGIS、`is_point_in_polygon()`、`load_districts.py` 皆已具備；待做的是 `check_player_in_district()` 與對外端點。

   **耗電**：判定掛在 SDD §5.5 既有的前景輪詢上（近距 5 秒／遠距 15 秒），是同一個 GPS fix 上的另一個判定，不是新的定位來源。

   ⚠️ `ST_Contains` 對剛好落在區界線上的點回傳 `False`（OGC 定義）。需要含邊界時換 `ST_Covers`，不是加 buffer。
3. `/state` 的 `next_target` 僅可回傳與 arc 同 `district_id` 的地標，並含可選 `direction_hint`；沒有任何「跨區下一站」欄位。
4. 跨日恢復只讀長期 Beat／道具／選擇狀態；不得要求重新走已完成路段或重新發放物件。

## 6. 玩家紀錄與隱私

### 應保存

- 隨機 install/device identifier 對應的匿名 `player_id`，以及帳號綁定資訊。
- 已完成 Beat、主線選擇、持有劇情物件、任務完成與共鳴流水帳。
- 玩家私有的對話紀錄與摘要，供該玩家自己的角色連續性使用。

### 不得保存

- 原始 GPS、位置歷史、移動軌跡或可回推軌跡的衍生欄位。
- 地標辨識後的影像、EXIF、影像位元組或推播中的影像資訊。
- 世界共享的玩家發言、玩家選擇或由選項推論出的敏感人格標籤。

### 6.1 未來合作推廣的隔離規則

觀光推廣、場館或在地店家優惠屬未來的**可選合作模組**，不屬故事／親近度／任務狀態機。若實作：

- 僅在玩家主動開啟合作頁後顯示；不注入人格 prompt、角色台詞或歷史資訊卡。
- 以一次性兌換碼讓合作單位驗證資格；合作單位不可查詢 `player_id`、故事選項、親近度、對話或位置資料。
- 兌換紀錄只保存防重複所需的最小資料，且不作為任何故事或跨區路線前置。

`dialogue_turns` 不加座標；玩家是否到場只由當次驗證使用，完成後不把座標寫入劇情或對話資料。

## 7. 實作順序

1. 完成龍山寺垂直切片：玩家、感應、召喚、對話、任務、共鳴。
2. 建立可審核的劇本 YAML 匯入流程與故事內容人工啟用門檻；MVP 開發／內測以 `DRAFT`／`INTERNAL` 隔離內容進行，不將人工審核當作開發阻塞，但公開啟用前的審核門檻不變。
3. 新增 `players_story_choices`、同區 arc 入口／方向驗證、故事 state／choice／chronicle API，並更新 OpenAPI 合約與 Unity DTO。
4. 實作 `story_beats` 狀態機與三地標 story 任務目錄。
5. 接上〈回家的畫〉、年代簿與萬華餘白每日輪播。

## 8. 驗收重點

- 同一選項重送不會產生第二筆選擇、第二份道具或第二次共鳴。
- 玩家不能從 Unity 偽造未開放的 choice／Beat。
- 萬華入口只在**萬華區圍欄首次判定成立**後啟動；其他行政區的進入事件不得發放萬華文件或推進萬華 Beat。玩家先到紅樓或剝皮寮時照樣拿得到信（同一個區），但主線標記只亮龍山寺。
- `next_target` 永遠屬於當前 arc 的 `district_id`；跨區資料即使存在，也不能進入必要路徑。
- 中斷後重啟，玩家從後端保存的狀態恢復，而非依本機猜測。
- 選擇與年代簿內容只屬該玩家，且不含定位歷史。
- B2、B9、B11、任務包裝與 B14 都注入人格卡及 taboo。
- 新 API 以 FastAPI code-first 產生並更新 `contracts/openapi.json`；破壞性變更受 CI 合約閘門阻擋。

## 9. 萬華主線任務設計〔提案／未匯入〕

本節是〈回家的畫〉景觀辨識方案的唯一工程任務規格；對話與歷史文案以 [景觀拍攝辨識完整互動腳本 v1](wanhua_district_storyline_aming_landmark_photo_v1.md) 為準。所有地標點位、照片構圖與歷史資訊仍待實勘及人工內容審核，不能直接啟用。

### 9.1 核心循環與硬規則

```text
有效到場 → 對話接取 → 公共觀察／景觀拍攝 → 反思選擇 → 回原靈魂交件 → 物件、Beat、下一站
LOCKED → AVAILABLE → IN_PROGRESS → READY_TO_TURN_IN → COMPLETED
```

- 三個必要任務都在 `district_id=wanhua`；可跨日、無期限，已確認進度不重置、不扣分。
- Unity 只呈現和提交事件；到場、觀察、反思、交件、共鳴、物件與 Beat 全由後端確定性判定。重送必須冪等。
- 資訊卡與重拍建議可略過，不能作為前置。不得要求宗教判讀、拍攝神像／參拜者／路人／住戶／店家、辨識人臉或追隨即時活動。
- 每站有一張核可的**公共景觀照片，全程可選**：玩家可完全不拍就交件。若要拍，只可由 App 內相機取得，使用 Encounter Token 呼叫既有 `POST /api/v1/quests/{quest_id}/landmark-photo`。影像只在記憶體辨識後立即釋放，不保存照片、EXIF、GPS、雜湊或訓練資料。
- **拍照不是完成條件**：不拍、送不出去（無訊號／逾時／429）、或 `landmark_recognized=false`，三者都可正常交件，差別只在有沒有景觀印記。硬性要求上傳會讓收訊變成主線 gate，違反 SDD §18.7.3。Encounter Token 才是到場安全邊界；後端不能由 JPEG 證明是否來自相簿，故相簿禁用是 Unity 流程要求而非伺服器保證。

### 9.2 共通互動、恢復與 UI

1. 召喚後完成接取對話，讀取該任務的核可觀察點與照片構圖設定。
2. 觀察點以文字、圖示與不小於 44×44 pt 的點按區呈現；點按確認交由後端驗證當次 token、地標、任務前置與去重。
3. 所有觀察點與一個白名單反思選項完成後，狀態即為 `READY_TO_TURN_IN`；再次對話才原子地交件。**景觀拍攝不參與此判定。**
4. 背景化、斷網或離場保留伺服器已確認進度；未確認事件提示可於另一天重試。施工或安全問題時只能使用內容審核過的替代示意圖，結果等同現場模式。
5. 每次交件順序固定為：已審核保底台詞 → 物件／年代簿頁 → 可略過的 60–90 字資訊卡 → 同區下一站理由與手動方向提示。

### 9.3 三站任務目錄

| 任務／前置／交件 | 玩家目標與後端完成條件 | 核可觀察與照片（全數待實勘） |
| --- | --- | --- |
| `q_longshan_repair_trace`；前置 `beat_longshan_gate` + `item_wanhua_letter`；交件寫 `beat_longshan_clue`、授予 `item_homeward_painting` | 看見留存與修復。有效到場 + `ls_preserved_detail` + `ls_repair_trace` + `repair_lens`。`ls_public_roofline_photo` 為可選印記，不參與判定。下一站：紅樓。 | `ls_preserved_detail`：公共建築細節；`ls_repair_trace`：可公開的修復線索；`ls_public_roofline_photo`：只取寺廟外觀／屋頂輪廓，禁止神像、供品、香客、儀式、人臉。 |
| `q_redhouse_two_forms`；前置 `beat_longshan_clue` + `item_homeward_painting`；交件寫 `beat_redhouse_clue`、授予 `item_painting_reverse_fragment` | 看見八角堂與十字樓如何相接。有效到場 + `rh_octagon_hall` + `rh_cross_hall_connection` + `building_lens`。`rh_public_facade_photo` 為可選印記。下一站：剝皮寮。紅樓對話必須明說其 1908 年才出現，非阿明時代見證者。 | `rh_octagon_hall`：八角堂外觀；`rh_cross_hall_connection`：兩棟量體相接；`rh_public_facade_photo`：八角堂公共外觀，不以市集、展演、店家、招牌或人潮辨識。 |
| `q_bopiliao_street_lines`；前置 `beat_redhouse_clue` + 兩份畫作物件；交件寫 `beat_bopiliao_reveal`，開啟年代簿結尾 | 比對門洞、屋簷、騎樓，理解畫中的「她」也可能是一條街。有效到場 + `bp_doorway_line` + `bp_eave_line` + `bp_arcade_line` + `reveal_lens`。`bp_street_facade_photo` 為可選印記。下一步：年代簿，不是地理目標。 | `bp_doorway_line`、`bp_eave_line`、`bp_arcade_line`：公共街屋線條；`bp_street_facade_photo`：公共立面，禁止住戶門牌、室內、路人、車牌、塗鴉與可變動招牌。 |

反思選項均無正誤：`repair_lens`（留存／修復／兩者）、`building_lens`（形狀／用途／再看）、`reveal_lens`（街／人／回家）。它們只能改變後續台詞與個人年代簿，不改變任務門檻、物件或路線。

### 9.4 資料、API 與交易規格

以下為實作提案，不能假定既有 `/quests`、schema 或 OpenAPI 已支援；完成後必須 code-first 更新 `contracts/openapi.json`。

```yaml
quest_definition:
  quest_id: q_longshan_repair_trace
  arc_id: wanhua_homeward_painting
  district_id: wanhua
  place_id: longshan_temple
  prerequisite: { beats: [beat_longshan_gate], inventory: [item_wanhua_letter] }
  observation_points: [ls_preserved_detail, ls_repair_trace]
  photo_evidence: { point_id: ls_public_roofline_photo, optional: true, required_upload: false, recognition_required: false }
  reflection: { choice_key: repair_lens, required: true }
  turn_in: { dialogue_beat: beat_longshan_clue, grants: [item_homeward_painting] }
```

| 動作 | Unity 傳送 | 後端驗證／回傳 |
| --- | --- | --- |
| 讀取狀態 | arc ID | 回傳狀態、已確認點、同區下一站與內容版本。 |
| 確認觀察 | `quest_id`、`point_id`、當次 token | 驗證 token 範圍、地標、前置、白名單與去重；回傳已完成點與是否可反思。 |
| 景觀拍攝 | App 相機 JPEG、Encounter Token | 沿用 landmark-photo 的配額與即用即丟；回傳 `landmark_recognized`，不回傳 URL、雜湊、尺寸或座標。 |
| 反思／交件 | `choice_key/value` 或交件對話 ID、當次 token | 白名單、全部條件、Beat 順序與冪等；交件交易回傳物件、Beat、保底台詞與下一站。 |

```text
驗證 Session + Encounter 與前置
  → 去重寫入 observation／photo-attempt／choice／quest progress
  → 達交件門檻時同一交易寫 inventory + players_story_progress + players_story_choices
  → commit → 可選 Brain 補充台詞；失敗回退已審核預寫台詞
```

Body 寫入成功後 Brain 不得回滾；跨日只讀持久化 Beat、物件、點位與選擇。任何主線狀態、日誌或推播不得含原始座標、影像、EXIF 或完整對話。

### 9.5 Unity 元件責任

| 元件 | 必須做 | 不得做 |
| --- | --- | --- |
| `QuestStatePresenter` | 以伺服器狀態顯示任務，重連後刷新。 | 用本機旗標決定完成。 |
| `ObservationOverlay` | 顯示已審核導覽點與完成回饋。 | 用本機／客戶端影像辨識作驗收。 |
| `LandmarkPhotoCapture` | App 內相機、避開私人資訊提醒、Encounter Token 上傳與結果提示；上傳失敗時靜默降級，不阻擋交件。 | 相簿入口、長期快取影像、上傳 EXIF，自行判定完成，或把上傳失敗呈現為錯誤。 |
| `QuestDialogueBridge` | 接取、反思、交件及可重看文字。 | 在客戶端發物、推 Beat 或判斷選項正確。 |
| `QuestLogBadge` | HUD 任務 icon 的未讀標記；佇列中有任何未讀即顯示驚嘆號，全部讀過才消失。 | 把 icon 狀態綁在單一任務上（第二個任務進來就要重寫）。 |
| `StoryLetterView` | 全螢幕信件／畫作／信紙手寫問句；**開啟即清除該項目的未讀標記**（不是選完才清）；選完後另外兩句淡出但保留。 | 用對話氣泡呈現（此節點無靈魂在場）、選完後移除未選項、自動全螢幕彈出。 |

**序章的呈現規格見腳本 §2.3。** 三個要點在此重述，因為它們會直接寫進元件行為：

1. **發放與閱讀分開。** 圍欄觸發時機不受控（捷運上、西門町、騎車等紅燈），因此發放只加驚嘆號，全螢幕由玩家主動點開。龍山寺主線標記在**發放當下**就亮，不等玩家讀信。
2. **未讀標記在「開啟」時清除，不是「完成選擇」時。** 玩家可能點開看一眼就被打斷；再回來時不該又是未讀。此時項目狀態為「進行中」而非完成。
3. **`story_focus` 是可重讀狀態，不是消耗掉的選擇。** 年代簿重讀時三句都在，選過的有標記——所以 `type: choice` 在此處是**非互斥呈現**，客戶端不得在選後移除未選項。

### 9.6 QA 驗收與實作前門檻

- 首次進入萬華只發一次信；在區界附近來回移動、或每次開 App 重新判定，都不得重複發放（唯一索引驗證）。其他行政區不得發放。中斷後重啟保留已確認的點位與物件。
- 無有效 Encounter Token 的觀察／拍攝拒絕且不寫入；重送交件不重複發物或加共鳴。
- 少任何觀察或反思都不可交件；跳過資訊卡不影響完成。
- **完全不拍照可交件**；拍了送不出去（斷網／逾時／429）可交件；`landmark_recognized=false` 可交件。三者只差有無景觀印記，且都不得顯示為錯誤。相簿在 Unity 無入口，但不宣稱後端可辨識 JPEG 來源。
- 完成剝皮寮只開啟年代簿，絕不要求其他行政區；合作頁不影響任務、共鳴、對話、物件或結局。
- 啟用前必須完成：7 個點按觀察點與 3 個拍攝點的安全／無障礙／公共視角實勘、場域與歷史審核、替代示意圖及停用流程；再決定 story quest 對既有 `public.quests`／`quest_progress` 與 story 表的正式 schema 和 migration。
