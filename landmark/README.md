# 地標史實研究記錄

每個地標一份，內容是查資料階段的記錄，**不是正式資料**。正式資料的終點是 `brain.landmark_souls` / `brain.city_souls`。

## 檔案

| 檔案 | 地標 | 行政區 |
|---|---|---|
| `landmark_research_sop.md` | 建立流程 SOP，開新地標之前先讀 | — |
| `lead_decisions_v3.md` | Lead 決策摘要 v3（2026-08-11），含 14 項待拍板事項 | — |
| `longshan_temple.md` | 艋舺龍山寺 | 萬華 |
| `ximen_red_house.md` | 西門町紅樓 | 萬華 |
| `bopiliao_historic_block.md` | 剝皮寮歷史街區 | 萬華 |
| `xiahai_city_god_temple.md` | 霞海城隍廟 | 大同 |
| `taiwan_new_cultural_movement_memorial.md` | 臺灣新文化運動紀念館 | 大同 |
| `national_palace_museum.md` | 國立故宮博物院 | 士林 |
| `taipei_astronomical_museum.md` | 天文台 | 士林 |
| `taipei_fine_arts_museum.md` | 臺北市立美術館 | 中山 |
| `moca_taipei.md` | 台北當代藝術館 | 中山 |
| `songshan_cultural_park.md` | 松山文創園區 | 信義 |
| `rongjin_gorgeous_time.md` | 榕錦時光生活園區 | 大安 |

11 個地標，內容皆已填寫並經 Lead 覆核（各檔末尾的「下一步」checklist 可看完成狀態）。

## 三件已知的事

### 1. 中正紀念堂缺一份

`lead_decisions_v3.md` 記載「地標範本 12 / 12 全部完工」，並把中正紀念堂列為新建的兩份之一，但這裡只有 11 份。可能是合併總冊時漏掉，也可能是依 A 級決策「建議不納入首發，或不排早期上線」而刻意排除。要向撰寫者確認。

### 2. `spirit_id` 有三個版本，尚未定案

以龍山寺為例：

| 出處 | 值 |
|---|---|
| `longshan_temple.md` 本檔 | `taipei_longshan` |
| `citysoul-backend` 的 `app/db/seed.py`（已寫入資料庫） | `longshan_temple` |
| SDD §7.3.1 的 Addressables key（客戶端載 3D 模型用） | `spirit_longshan` |

第三個是**客戶端資源鍵，本來就跟後端主鍵不同**，不衝突。真正要處理的是前兩個。定案之前不要開始寫 `landmarks/*.yaml`——改資料庫主鍵比改文件貴得多。

### 3. 兩條「必須寫死進角色知識層」的事實

`lead_decisions_v3.md` 標出兩處空間關係的事實錯誤，性質跟 `taboos`（不談什麼）不同，是「談的時候不能講錯」：

- 蔣渭水沒有被關在臺灣新文化運動紀念館現存的建築裡（他 1931 年逝世，現存建築 1933 年完工）
- 榕錦時光是刑務所的**官舍**，不是牢房也不是刑場

`brain.character_personas` 目前沒有欄位放這類「事實更正」。要嘛併進 `landmark_souls.key_events` 用敘述表達，要嘛另加欄位——待決。
