# 主線劇情（story arc）

一次性的劇情內容：有前置因果鏈、會寫入玩家進度、跑過就不會再跑一次。

對應資料表：`brain.story_arcs`、`brain.story_beats`、`brain.resonance_unlockables`、`brain.districts`，玩家側進度記在 `public.players_story_progress`。

## 跟 `event/` 的分界

| | `story/`（這裡） | `event/` |
|---|---|---|
| 內容 | 主線劇情 | B9 當日情境素材 |
| 生命週期 | 一次性 | 每日輪替 |
| 因果關係 | 有前置鏈（`prerequisite_beat_ids`） | 無 |
| 玩家進度 | 記錄在 `players_story_progress` | 不記錄 |
| 生成路徑 | 腦袋依 `narrative_directive` 組 prompt | B9 當日情境生成 |

同一個地標兩種內容都會有，但走不同的表與不同的生成路徑，不要混在一起寫。

## 檔案

### v1 基線（下一輪討論與工程交付）

- `wanhua_district_storyline_aming_no_landmark_photo_v1.md` — 無景觀辨識的完整互動腳本。
- `wanhua_district_storyline_aming_landmark_photo_v1.md` — 景觀辨識的完整互動腳本。
- `wanhua_story_runtime_architecture_no_landmark_photo_v1.md` — 無景觀辨識運行架構。
- `wanhua_story_runtime_architecture_v1.md` — 景觀辨識運行架構。

- `story_arc_writing_sop.md` — 主線劇情撰寫 SOP。寫新 arc 之前先讀：前置條件、beat 設計的五條硬規則、交付前檢查表。
- `wanhua_district_storyline_aming_no_landmark_photo_v0.2.md` — 萬華區〈萬華年代簿・回家的畫〉無景觀辨識互動腳本，狀態為草案。
- `wanhua_district_storyline_aming_landmark_photo_v0.1.md` — 互動腳本的景觀拍攝辨識互斥增補版，狀態為方案 A／未定案。
- `wanhua_story_runtime_architecture.md` — 萬華主線的引擎、API、資料庫、玩家紀錄與三個任務的工程規格提案。
- `wanhua_story_runtime_architecture_no_landmark_photo.md` — 不使用景觀拍攝辨識的互斥替代架構提案，供產品決策比較。

## 目前的狀態

**尚無任何 arc 內容正式匯入或啟用。** 劇情所需資料表已由後端 migrations 0013／0015 建立，但 Beat 狀態機、故事 API、內容匯入與人工啟用流程尚未實作；三個地標的人格內容也仍須人工審核。

完整的前置清單見 `wanhua_district_storyline_aming_no_landmark_photo_v0.2.md` §9.1。
