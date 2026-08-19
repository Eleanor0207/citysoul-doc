# 角色設定與人格卡內容

人格卡的正式內容（core_personality／speaking_style／factual_boundary／
taboo_topics／canned_greetings）放這裡。

⚠️ 龍山寺是宗教場所，內容受 SDD §3 的 Avoid 條目約束：不對特定宗教信仰、
神祇、儀式作出教義性陳述或裁決。治理規範見 citysoul-backend#41。

## 這個資料夾是副本，不是正本

底下每一份 `*.md` 都由 citysoul-backend 的 `scripts/export_personas_md.py`
從 `content/personas/*.yaml` 產生。**改 markdown 沒有任何效果**，匯入器不讀它。
要改人格：改 YAML、重跑匯出、重跑 `scripts.import_personas`，再由人 flip
`brain.character_personas.active`。

方向跟 `landmark/` 相反——那邊是 markdown 為正本，YAML 由腳本產生。因為史實是人
寫的研究，人格是給模型吃的結構。

## 欄位對應

文件裡用的是資料庫欄位名，跟本頁開頭的舊稱呼對照如下：

| 舊稱呼 | 實際欄位 |
| --- | --- |
| `core_personality` | `archetype` + `personality_traits` + `values` |
| `speaking_style` | `speech_style` |
| `factual_boundary` | `imagination_license`（另一半在 B5 規則與 `brain.landmark_souls`，不在人格卡） |
| `taboo_topics` | `taboos` + `not_this_character` |
| `canned_greetings` | 同名 |

## 十份人格卡

| 地標 | `character_id` | 版本 |
| --- | --- | --- |
| [艋舺龍山寺](longshan_temple_v3.md) | `longshan_watcher` | v3 |
| [國立故宮博物院](national_palace_museum_v3.md) | `palace_custodian` | v3 |
| [剝皮寮歷史街區](bopiliao_historic_block_v1.md) | `bopiliao_keeper` | v1 |
| [西門紅樓](ximen_red_house_v1.md) | `red_house_collector` | v1 |
| [霞海城隍廟](xiahai_city_god_temple_v1.md) | `xiahai_neighbour` | v1 |
| [臺灣新文化運動紀念館](taiwan_new_cultural_movement_memorial_v1.md) | `new_culture_witness` | v1 |
| [臺北市立美術館](taipei_fine_arts_museum_v1.md) | `fine_arts_curator` | v1 |
| [臺北當代藝術館](moca_taipei_v1.md) | `moca_experimenter` | v1 |
| [臺北市立天文科學教育館](taipei_astronomical_museum_v1.md) | `astronomy_observer` | v1 |
| [松山文創園區](songshan_cultural_park_v1.md) | `songshan_worker` | v1 |

龍山寺與故宮是 v3：v2 到 v3 只改稱謂（「牠」改「祂」，見
`story/story_arc_writing_sop.md` §5.2），故宮另外把自我介紹的「外雙溪」改成
「臺北市郊」。

## 檔頭的 `reviewed_by` 不是上線狀態

那是草稿檔上的簽名。實際生效與否只看資料庫的 `active`，而 2026-08-14 的簽核是
直接對資料庫下 UPDATE 的，沒有回寫 YAML——所以十份裡有九份的檔頭仍寫著
`PENDING_HUMAN_REVIEW`，但十份在 Cloud SQL 上都是 `active = true`，審核者
`Jessie_Lee`。要看真實狀態請查資料庫，或跑 `scripts.review_personas`。
