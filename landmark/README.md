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

### 1. 中正紀念堂不納入專案（2026-08-13 決定）

`lead_decisions_v3.md` 把它列為新建的兩份範本之一，並開了一項 A 級阻擋決策（角色採哪一種語域）。**該決策已作廢，不需要再拍板**，也不要為它補範本。地標總數是 11 個，不是該文件寫的 12 個。

### 2. `spirit_id`：龍山寺已定案，其餘 10 個待定

龍山寺定為 **`longshan_temple`**（2026-08-13），與 `citysoul-backend` 的 `app/db/seed.py` 已寫入資料庫的值一致。研究檔原本寫的 `taipei_longshan` 已改掉。

> SDD §7.3.1 的 Addressables key `spirit_longshan` 是**客戶端載 3D 模型用的資源鍵**，本來就跟後端主鍵是兩個不同的識別碼，不算衝突。

其餘 10 個地標的研究檔都寫「spirit_id 命名待定」，只有榕錦時光給了建議值 `tpprison_quarters`（取文資登錄名而非營運品牌名，理由見該檔 Step 1）。**開始寫 `landmarks/*.yaml` 之前要全部定案**——改資料庫主鍵比改文件貴得多。

### 3. 兩條「必須寫死進角色知識層」的事實

`lead_decisions_v3.md` 標出兩處空間關係的事實錯誤，性質跟 `taboos`（不談什麼）不同，是「談的時候不能講錯」：

- 蔣渭水沒有被關在臺灣新文化運動紀念館現存的建築裡（他 1931 年逝世，現存建築 1933 年完工）
- 榕錦時光是刑務所的**官舍**，不是牢房也不是刑場

`brain.character_personas` 目前沒有欄位放這類「事實更正」。要嘛併進 `landmark_souls.key_events` 用敘述表達，要嘛另加欄位——待決。
