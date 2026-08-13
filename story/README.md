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

- `story_arc_writing_sop.md` — 主線劇情撰寫 SOP。寫新 arc 之前先讀：前置條件、beat 設計的五條硬規則、交付前檢查表。
- `wanhua_district_storyline_aming_v0.2.md` — 萬華區「阿明的遺願」，目前唯一一條 arc，狀態為草案。

## 目前的狀態

**沒有任何一條 arc 進得了資料庫。** `story_beats` 等九張表在 `citysoul-backend` 都還沒建（`citysoul_data_schema.md` §8/§9 已完成設計，migration 尚未跟上），三個地標裡也只有龍山寺有靈魂記錄，而且人格卡還是未審核的草稿。

完整的前置清單見 `wanhua_district_storyline_aming_v0.2.md` §9.1。
