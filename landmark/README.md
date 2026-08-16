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

10 個地標，內容皆已填寫並經 Lead 覆核（各檔末尾的「下一步」checklist 可看完成狀態）。

## 三件已知的事

### 1. 中正紀念堂不納入專案（2026-08-13 決定）

`lead_decisions_v3.md` 把它列為新建的兩份範本之一，並開了一項 A 級阻擋決策（角色採哪一種語域）。**該決策已作廢，不需要再拍板**，也不要為它補範本。地標總數是 11 個，不是該文件寫的 12 個。

### 2. `spirit_id` 全部定案（2026-08-13）

10 個全數拍板，各研究檔的基本資訊區已寫入。

| 地標 | `spirit_id` |
|---|---|
| 艋舺龍山寺 | `longshan_temple` |
| 西門町紅樓 | `ximen_red_house` |
| 剝皮寮歷史街區 | `bopiliao_historic_block` |
| 霞海城隍廟 | `xiahai_city_god_temple` |
| 臺灣新文化運動紀念館 | `new_cultural_movement_memorial` |
| 國立故宮博物院 | `national_palace_museum` |
| 天文台 | `taipei_astronomical_museum` |
| 臺北市立美術館 | `taipei_fine_arts_museum` |
| 台北當代藝術館 | `moca_taipei` |
| 松山文創園區 | `songshan_cultural_park` |

命名規則三條：

1. **`{專名}_{類型}`，全稱不縮寫。** `longshan_temple` 是既有值，其餘對齊它。研究檔原本的 `npm`（故宮）在任何有 JavaScript 的環境裡都是別的東西；`tp`、`tfam` 是同一類問題。
2. **不用營運品牌名，用文資登錄名。** 品牌會換約、會改名，登錄名不會。目前 10 個沒有踩到這條，但下一個地標可能會。
3. **不用角色暱稱**（SDD §7.3.1：角色名可能變更，不得進入技術命名）。

唯一的例外是 `moca_taipei`——展開的 `museum_of_contemporary_art_taipei` 幾乎沒人使用，館方對外正式英文名就是 MOCA Taipei。北美館沒有這個問題，所以用全稱。

> **客戶端的 Addressables key 是另一套識別碼**，不受影響。SDD §7.3.1 的 `spirit_longshan` 是載 3D 模型用的資源鍵，跟後端主鍵本來就不同。

### 2.1 `character_id` 不重複整個 `spirit_id`

既有值是 `longshan_watcher`，不是 `longshan_temple_watcher`。慣例是 `{地名}_{英文角色定位詞}`——地名取短的那一段，定位詞用角色在敘事裡的功能。

萬華三個地標的建議值（尚未建立）：

| 地標 | `character_id` |
|---|---|
| 艋舺龍山寺 | `longshan_watcher`（已存在） |
| 西門町紅樓 | `red_house_collector` |
| 剝皮寮歷史街區 | `bopiliao_keeper` |

### 3. 兩條「必須寫死進角色知識層」的事實

`lead_decisions_v3.md` 標出兩處空間關係的事實錯誤，性質跟 `taboos`（不談什麼）不同，是「談的時候不能講錯」：

- 蔣渭水沒有被關在臺灣新文化運動紀念館現存的建築裡（他 1931 年逝世，現存建築 1933 年完工）

`brain.character_personas` 目前沒有欄位放這類「事實更正」。要嘛併進 `landmark_souls.key_events` 用敘述表達，要嘛另加欄位——待決。
