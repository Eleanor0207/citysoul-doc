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

| 檔案 | 用途 |
|---|---|
| `wanhua_district_storyline_aming_landmark_photo_v1.md` | 萬華〈回家的畫〉**完整互動腳本**——劇情、任務、逐句對話、資料規格與治理規則。內容作者與審核的正本 |
| `wanhua_story_runtime_architecture_v1.md` | 同一條 arc 的**工程交付規格**——資料模型、API 增量、寫入順序、Unity 元件責任、驗收 |
| `story_arc_writing_sop.md` | 寫新 arc 前先讀：前置條件、beat 設計的五條硬規則、交付前檢查表 |

### 為什麼只剩一套

曾經有兩套並行：方案 A（含景觀拍攝辨識）與方案 B（無景觀辨識），各一份腳本與一份架構。

**景觀拍攝改為完全可選之後，兩者收斂了**——B 的任務判定條件與 A 逐字相同，整份差異只剩拍照那一節。A 現在等於 B 加上一個可選的景觀印記，所以 B 沒有存在的理由。

留 A，B 已刪除（內容仍在 git 歷史裡）。維護兩份四萬字的完整腳本、差別只在一個可選功能，是劇本漂移最快的路徑——而且劇本漂移比規格漂移更難發現，沒有任何測試會抓到兩份台詞不一樣。

檔名保留 `landmark_photo` 是為了不動既有引用；它現在指的是「含可選景觀印記的版本」，不是「必拍版本」。

## 目前的狀態

**尚無任何 arc 內容正式匯入或啟用。** 劇情所需資料表已由後端 migrations 0013／0015 建立，但 Beat 狀態機、故事 API、內容匯入與人工啟用流程尚未實作；三個地標的人格內容也仍須人工審核。

完整的前置清單見 `story_arc_writing_sop.md` 的前置檢查表，與腳本 §10「實作與審核清單」。
