# 萬華區主線劇情 — 阿明的遺願

> **對應文件**：`citysoul_data_schema.md` §2／§4／§5／§7–10（`brain.districts`、`brain.story_arcs`、`brain.story_beats`、`brain.resonance_unlockables`、`public.quests`、`public.dialogue_turns`、`public.players_story_progress`、`public.player_inventory`、`public.media_assets`）、`SDD_v2.1_Unity_3D.md` §12.2（龍山寺內容治理）
> **狀態**：劇情設計草案 v0.2，原始大綱由 阿和 提供
> **地標範圍**：龍山寺、西門紅樓、剝皮寮（三者共屬 `brain.districts` 的「萬華區」）
> **v0.1 → v0.2 修訂重點**：見文末附錄 A

---

## 0. 一句話大綱

一個死於 1815 年淡水地震、生前思鄉心切的少年阿明，死後化為遊蕩的靈魂，玩家要透過三個地標靈魂的破碎記憶拼湊真相——他放不下的不是愛情，是**回家**。

**核心反轉**：玩家一路以為在找「阿明喜歡的女孩」，最後發現那個「女孩」的形象，其實是阿明對故鄉剝皮寮的眷戀所投射出來的樣子——剝皮寮靈魂記得的「畫中人」就是自己。

---

## 1. 世界觀對齊提醒〔請先讀，會影響下面兩個設計決定〕

SDD §3 Avoid 清單明寫：

> 「城市靈魂」是地標的擬人化集體意識，**不扮演任何特定真人或歷史人物**。

原始大綱有兩處字面上容易滑向「靈魂＝真人」，本版已在對應章節套用調整（劇情骨架完全不動，只調敘事角度）：

| 原始寫法                                                               | 風險                                               | 本版採用的寫法                                                                                                                                                                                                 |
| ---------------------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 「故事中提到的女性其實就是剝皮寮靈魂」                                 | 讀起來像剝皮寮靈魂＝那個歷史上真實存在過的女孩本人 | **剝皮寮靈魂是那個女孩「在阿明記憶裡的樣子」被地方集體意識吸收後的投影**——剝皮寮靈魂自己也說不清那到底是不是「她」，她只知道自己「一直是這個樣子被記得」。這樣反而更貼近「失憶老靈魂」的設定，也更悲傷 |
| 「如果玩家問靈魂怎麼知道那個是阿明，靈魂可以說是寺廟裡的神明告訴它的」 | 對應 §12.2 taboo：不代替神明發言、不做教義性陳述  | 靈魂用**自己的觀察力**回答，見 §5.1                                                                                                                                                                     |

這兩處調整不影響任何劇情節點、任務、彩蛋，純粹是「靈魂怎麼講這句話」的問題，敘事審查（Lead＋腦袋 B4/B5）過件會更順。

---

## 2. 角色深化

### 2.1 阿明（不存在於當代場景，僅透過三個靈魂的記憶被轉述）

| 項目     | 內容                                                                                                                                                                                                                                                   |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 出生     | 清代嘉慶二年（1797），剝皮寮土炭商人之子，獨生子                                                                                                                                                                                                       |
| 死亡     | 嘉慶二十年／西元 1815 年淡水地震，得年 18 歲；靈魂遊蕩至隔年（嘉慶二十一年，1816）百日後離開人間                                                                                                                                                       |
| 家庭     | 父母晚婚，家中經濟仰賴土炭燒製與販售；阿明從小分擔家業，個性懂事早熟                                                                                                                                                                                   |
| 興趣     | 用剩下的土炭在地上塗鴉，畫工不好——這個「畫得爛」的細節在剝皮寮線索裡會被靈魂調侃，形成前後呼應的溫柔笑點                                                                                                                                             |
| 情感核心 | 表面上是「思念一個女孩」，實際上是**思鄉**。女孩的形象是阿明用來安放「家的安全感」的具體符號，他自己未必分得清這兩者的差別——這也是為什麼死後的他聽不見剝皮寮靈魂呼喚：不是不認得她，是他要找的「家」已經回不去了，任何形式的呼喚都進不了他心裡 |
| 死亡瞬間 | 出門採買時遭遇倒塌重物掩埋，最後意識停留在「認不出這裡是哪裡」的破碎地景——這個細節解釋了他死後為什麼一直迷路：他連自己是在哪裡死的都不知道                                                                                                           |

**寫作提醒**：阿明沒有任何一句「台詞」直接出現在遊戲裡，他完全透過三個靈魂的轉述存在。這個限制其實是優點——玩家永遠拼不出「阿明真正說過什麼」，只能拼出「三個靈魂記得他做過什麼」，天然地強化了「記憶是不可靠的、有缺口的」這個劇情母題，也天然對齊 SDD §3「世界記憶不含任何玩家輸入、靈魂不扮演真人」的邊界。

### 2.2 龍山寺靈魂

| 人格卡欄位             | 內容建議                                                                                                     |
| ---------------------- | ------------------------------------------------------------------------------------------------------------ |
| `archetype`          | 見多識廣的旁觀者，習慣看人來人往求神拜佛，語氣帶點閱盡千帆的淡然                                             |
| `speech_style`       | 沉穩、略帶古意但不迂腐；面對阿明這條線索時會轉為少見的柔軟                                                   |
| `not_this_character` | 不是廟方人員、不是任何神祇、不是宗教解說員、**不轉述神明的話語**（呼應 §1 修正）                      |
| 對阿明的態度           | 沒有太多評判，只是「剛好記得」；把畫作轉手給紅樓靈魂這件事，帶一點「這不是我該保管的東西」的分寸感，不是冷漠 |

### 2.3 西門紅樓靈魂

| 人格卡欄位       | 內容建議                                                                                                                       |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `archetype`    | 活潑健談、對「故事」本身有收藏癖，喜歡聽八卦也喜歡講古蹟歷史                                                                   |
| `speech_style` | 輕快、帶點自嘲（「我最近太忙，把這事給忘了」）                                                                                 |
| 對阿明的態度     | 對畫作感興趣的理由不是同情阿明，是單純的「這是個好故事」——這個帶點功利的好奇心，跟龍山寺的淡然、剝皮寮的沉重形成三種語氣層次 |

### 2.4 剝皮寮靈魂

| 人格卡欄位            | 內容建議                                                                                                                                             |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `archetype`         | 年邁、記憶力衰退的女性形象，語速慢、常常說到一半自己也不確定                                                                                         |
| `speech_style`      | 破碎、跳躍、帶重複語句（模擬失憶感，例如「我好像……不對，我是不是說過了」）                                                                         |
| `contingency_notes` | 條件未滿足時的「想不起來」狀態走這裡，不另開劇情節點（理由見 §3.4 註記）                                                                            |
| 關鍵反轉時刻          | 當她把畫作、塗鴉、龍山寺回憶三者連起來時，語氣應該有一瞬間「清醒」——這是全劇情最重的一拍，建議由腦袋在 Prompt 組裝時給這個 beat 特別高的敘事優先權 |

---

## 3. 資料表對照與寫入範例

九張表各自在這個劇本裡的角色分工：

| schema | 表                         | 在這個劇本裡負責什麼                                                                    |
| ------ | -------------------------- | --------------------------------------------------------------------------------------- |
| brain  | `districts`              | 萬華區的多邊形圍欄，決定玩家何時算「進入劇本範圍」                                      |
| brain  | `story_arcs`             | 這條主線本身這一筆記錄，含信件全文                                                      |
| brain  | `story_beats`            | 七個劇情節點，`prerequisite_beat_ids` 串成因果鏈                                      |
| brain  | `resonance_unlockables`  | 跟 beat 分開的另一條解鎖線——依共鳴值高低額外開放的記憶片段，不綁事件觸發              |
| public | `quests`                 | 三個靈魂各自的拍照任務定義（`steps` JSONB）                                           |
| public | `dialogue_turns`         | 每一輪對話的日誌，`story_beat_id` 記錄「這句話發生在哪個劇情節點上」                  |
| public | `players_story_progress` | 玩家實際踩過哪些 beat 的**真相來源**（`dialogue_turns` 只是日誌，不是判斷依據） |
| public | `player_inventory`       | 信件與畫作道具，`required_item_ids` 靠它判斷玩家是否持有                              |
| public | `media_assets`           | 信件與畫作的插圖檔案路徑                                                                |

> ⚠️ **這九張表目前在 `citysoul-backend` 一張都還沒建。** `citysoul_data_schema.md` 已完成設計，但 migration 0001–0011 沒有涵蓋。本文件是「內容規格」，不是「可以直接執行的 SQL」——下方所有 SQL 是欄位對照示意，實際匯入走 §10 的 YAML 管線。

### 3.1 `brain.districts` — 萬華區圍欄

```sql
INSERT INTO brain.districts
  (district_id, city_id, name, center_lat, center_lng, radius_meters, boundary)
VALUES
  ('wanhua', 'taipei', '萬華區',
   25.0330, 121.4998, 1200,
   ST_GeomFromGeoJSON('<實際萬華區行政區界 GeoJSON>'));
```

> 🚧 **這筆是整條 arc 的阻塞項，不是待辦事項。** `boundary` 是 `check_player_in_district()` 的唯一判斷依據（`citysoul_data_schema.md` §9）。沒有真實 GeoJSON，圍欄判斷恆為 false，「阿明留下的信」永遠發不出去，**整條主線的入口不存在**。`center_lat/lng/radius_meters` 補不了這個洞，它們只供地圖 UI 畫概略圓形。
>
> 解法不難：取政府開放資料平台的臺北市行政區界 GeoJSON，不需手動描點。但要在建表之前排進去。

### 3.2 `public.media_assets` — 信件與畫作插圖

```sql
-- 信件插圖（若有）
INSERT INTO media_assets (asset_id, asset_type, owner_type, owner_id, gcs_path, cdn_url, content_type)
VALUES ('<uuid_letter>', 'story_item_illustration', 'system', 'arc_wanhua_aming', '<gcs路徑>', '<cdn url>', 'image/png');

-- 少女畫作插圖（阿明留下的那張）
INSERT INTO media_assets (asset_id, asset_type, owner_type, owner_id, gcs_path, cdn_url, content_type)
VALUES ('<uuid_painting>', 'story_item_illustration', 'system', 'item_painting_of_girl', '<gcs路徑>', '<cdn url>', 'image/png');
```

兩者都是系統素材（`owner_type='system'`），`spirit_id` 留 NULL——這兩張圖屬於整條主線，不專屬單一靈魂。

### 3.3 `brain.story_arcs` — 阿明的遺願

```sql
INSERT INTO brain.story_arcs
  (arc_id, title, district_id, intro_document_title, intro_document_content, intro_document_asset_id, summary)
VALUES (
  'arc_wanhua_aming',
  '阿明的遺願',
  'wanhua',
  '阿明留下的信',
  '<信件全文，見原始大綱>',
  '<uuid_letter>',   -- 值關聯 → media_assets.asset_id，不建 FK
  '玩家透過三個地標靈魂的記憶碎片，拼湊出一名死於淡水地震的少年阿明的真正遺願——不是尋找愛人，而是回家。'
);
```

### 3.4 `brain.story_beats` — 節點鏈

#### `character_id` 命名

`character_id` 必須對到 `brain.characters` 的實際值，不是佔位字串。目前 DB 裡**只有龍山寺一個角色存在**：

| 靈魂     | `character_id`                      | 狀態                                                                    |
| -------- | ------------------------------------- | ----------------------------------------------------------------------- |
| 龍山寺   | `longshan_watcher`                  | ✅ 已存在（`app/db/seed.py`），人格卡為 `active=false` 的未審核草稿 |
| 西門紅樓 | `ximen_red_house_collector`（建議） | ❌ 尚未建立                                                             |
| 剝皮寮   | `bopiliao_keeper`（建議）           | ❌ 尚未建立                                                             |

命名慣例沿用現有的 `{landmark_id}_{英文角色定位詞}`。定位詞用**角色在敘事裡的功能**，不用角色暱稱——SDD §7.3.1：角色名可能變更，不得進入技術命名。後兩者的最終命名待新地標的人格卡建立時一併定案。

#### 節點表

| beat_id                  | arc_id           | character_id              | sequence_order | trigger_condition                                        | narrative_directive                                                                                            | prerequisite_beat_ids                         | required_item_ids         | contingency_notes                                                   | one_time |
| ------------------------ | ---------------- | ------------------------- | -------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | --------------------------------------------- | ------------------------- | ------------------------------------------------------------------- | -------- |
| `beat_longshan_gate`   | arc_wanhua_aming | longshan_watcher          | 1              | 玩家首次與龍山寺靈魂對話                                 | 以「先幫忙向各廳打聲招呼」為由引導玩家做靈魂任務；任務完成前提及阿明一律溫和帶開                               | —                                            | —                        | 見 §5.3 龍山寺帶開語氣                                             | true     |
| `beat_longshan_clue`   | arc_wanhua_aming | longshan_watcher          | 2              | `quest_progress` 中 `q_longshan_ceremony` 狀態為完成 | 敘述兩次見到阿明的經過；語氣淡然帶憐惜；**不得**用「神明告訴我」交代資訊來源                             | `beat_longshan_gate`                        | —                        | 見 §5.1                                                            | true     |
| `beat_hongluo_gate`    | arc_wanhua_aming | ximen_red_house_collector | 1              | 玩家首次與紅樓靈魂對話                                   | 先介紹紅樓歷史，引導做靈魂任務                                                                                 | —                                            | —                        | 見 §5.2                                                            | true     |
| `beat_hongluo_clue`    | arc_wanhua_aming | ximen_red_house_collector | 2              | `q_hongluo_ceremony` 完成                              | 主動拿出畫作，坦承忙到沒空追問，交付畫作道具                                                                   | `beat_hongluo_gate`, `beat_longshan_clue` | —                        | 觸發`player_inventory` 寫入 `item_painting_of_girl`（見 §3.7） | true     |
| `beat_bopiliao_gate`   | arc_wanhua_aming | bopiliao_keeper           | 1              | 玩家首次與剝皮寮靈魂對話                                 | 反問玩家為何來此，語氣疏離帶防備                                                                               | —                                            | —                        | 條件未滿足時的「想不起來」走 §5.4                                  | true     |
| `beat_bopiliao_reveal` | arc_wanhua_aming | bopiliao_keeper           | 2              | 玩家持有`item_painting_of_girl`                        | **全劇情最重的一拍**：靈魂逐漸認出畫中人是自己，語氣由破碎轉短暫清醒；調侃畫工差；揭露死亡年齡與地震年份 | `beat_bopiliao_gate`, `beat_hongluo_clue` | `item_painting_of_girl` | 見 §7 情感節奏建議                                                 | true     |
| `beat_finale_reveal`   | arc_wanhua_aming | bopiliao_keeper           | 3              | `beat_bopiliao_reveal` 完成後的下一次對話              | 完整揭露：思鄉而非思人、剝皮寮靈魂即記憶投影、1815 地震、1816 百日離開                                         | `beat_bopiliao_reveal`                      | —                        | —                                                                  | true     |

#### 三件必須說明的事

**一、所有 `trigger_condition` 都是可判定的，不依賴語意理解。**

`trigger_condition` 的判斷邏輯在 body 狀態機（`citysoul_data_schema.md` §9），而狀態機只能查資料庫狀態——它讀不出「玩家在對話中提到了什麼」。任何需要理解玩家自然語言意圖才能判斷的條件，都不能寫進這一欄。

`beat_bopiliao_reveal` 因此**不需要**「且對話提及龍山寺回憶」這個條件：前置鏈已經遞移保證了。

```
beat_bopiliao_reveal ← beat_hongluo_clue ← beat_longshan_clue
```

玩家要拿到畫作就必須先過 `beat_hongluo_clue`，而那又必須先過 `beat_longshan_clue`。「已經聽過龍山寺線索」是既成事實，不必也不能再判一次。

**二、「想不起來」的狀態不是 beat。**

v0.1 曾有一個 `beat_bopiliao_vague`（`one_time=false`，可重複觸發）。本版移除，理由有二：

- `players_story_progress` 的主鍵是 `(player_id, beat_id)`，只能記錄一次，「重複觸發」在資料上沒有地方表達。
- 沒有任何節點依賴它。

它的本質是「前置條件還沒滿足時，剝皮寮靈魂怎麼回話」——那是 `contingency_notes`／人格語氣，不是劇情節點。移到 §5.4。移除後全部七個 beat 都是 `one_time=true`，語意乾淨。

**三、`sequence_order` 是單一角色內的排序，可以跨角色重複。**

三個 `_gate` 都是 1，這是正確的（`citysoul_data_schema.md` §8：MVP 採單一角色內線性排序）。匯入時要驗證的唯一性是 `(arc_id, character_id, sequence_order)`，不是 `(arc_id, sequence_order)`。

#### 入口道具不走 beat

「阿明留下的信」不是 beat，而是玩家進入 `district_id='wanhua'` 圍欄時由 `grant_arc_intro_document()` 直接寫入 `player_inventory`（見 §3.7）——這是 `citysoul_data_schema.md` §9 既定的機制。

跨角色依賴一律沿用 `check_beat_unlockable()`：`beat_hongluo_clue` 與 `beat_bopiliao_reveal` 都是「`prerequisite_beat_ids` 跨角色」的實例，判斷邏輯統一走 `players_story_progress`，不需要為這個劇本另寫例外。

### 3.5 `brain.resonance_unlockables` — 共鳴值解鎖的記憶片段

這條線跟 beat 鏈**分開存在**：beat 是事件驅動（做了什麼才觸發），`resonance_unlockables` 是數值驅動（共鳴值到門檻就開放），兩者互不取代。

> ⚠️ **`content` 欄位放的是敘事指令，不是成品台詞。** 這一點在 `citysoul_data_schema.md` §8 沒有寫明，容易誤解——欄位名叫 `content`，但它跟 `story_beats.narrative_directive` 是同一類東西：描述「要講什麼、用什麼語氣」，由腦袋 B11 據此生成實際字句。寫成成品台詞會讓三次解鎖聽起來像罐頭，而且繞過了語氣層與 relationship_stage 的深度限制。

```sql
INSERT INTO brain.resonance_unlockables (unlock_id, character_id, min_resonance, unlock_type, content) VALUES
('unlock_longshan_aming_memory', 'longshan_watcher', 40, 'memory_fragment',
 '在「信任」階段才願意多說的一件事：牠記得阿明離開時腳步很輕，輕到不像活人。語氣是不確定要不要說出口的猶豫。'),
('unlock_hongluo_aming_memory',  'ximen_red_house_collector',  40, 'memory_fragment',
 '在「信任」階段補的一句：牠曾經考慮過幫阿明找那個女孩，但後來覺得哪裡怪怪的，說不上來。語氣是難得的收斂。'),
('unlock_bopiliao_aming_memory', 'bopiliao_keeper', 100, 'secret_story',
 '在「摯友」階段才鬆口：牠其實一直都記得，只是不敢先說，怕說出口就要承認阿明真的不會再回來了。這是三則裡唯一觸及自我認知的一則，語氣要最安靜。');
```

> 門檻沿用固定的 10/40/100（`citysoul_data_schema.md` §8 `RESONANCE_STAGES`），不為這條主線另開特例數值。

### 3.6 `public.quests` — 三個拍照任務

`story_beats` 的 `trigger_condition` 引用了 `q_longshan_ceremony` 等 `quest_id`，這些任務必須先存在於 `quests` 表。任務內容見 §4。

```sql
INSERT INTO quests (quest_id, spirit_id, title, steps, reward_resonance)
VALUES ('q_longshan_ceremony', 'longshan_temple', '向各廳打聲招呼', '<steps JSONB，見 §4.1>', 20);
```

> `reward_resonance = 20` 沿用 AC5.1 的固定給分，不為主線任務另訂數值。

### 3.7 `public.player_inventory` — 道具追蹤

```sql
-- 進入萬華區圍欄時，由 grant_arc_intro_document() 寫入
INSERT INTO player_inventory (player_id, item_type, item_id, illustration_asset_id)
VALUES ('<player_id>', 'story_document', 'letter_aming', '<uuid_letter>');

-- beat_hongluo_clue 觸發時寫入
INSERT INTO player_inventory (player_id, item_type, item_id, source_quest_id, illustration_asset_id)
VALUES ('<player_id>', 'story_key_item', 'item_painting_of_girl', 'q_hongluo_ceremony', '<uuid_painting>');
```

`uq_inventory_player_item` 唯一約束保證兩者都只會發放一次，不需要額外去重邏輯。`beat_bopiliao_reveal` 的 `required_item_ids = ['item_painting_of_girl']` 直接查這張表即可判斷。

### 3.8 `public.dialogue_turns` — `story_beat_id` 怎麼蓋章

每一輪對話寫入時，若這輪對話**促成或屬於**某個 beat，`dialogue_turns.story_beat_id` 就填對應的 `beat_id`：

```sql
INSERT INTO dialogue_turns (player_id, spirit_id, role, content, resonance_value_at_time, story_beat_id)
VALUES ('<player_id>', 'bopiliao', 'spirit', '<揭露台詞內容>', 55, 'beat_bopiliao_reveal');
```

> `dialogue_turns` 只是日誌，**不是判斷玩家進度的依據**——「這個 beat 到底觸發了沒」永遠查 §3.9 的 `players_story_progress`，避免兩邊對不上時不知道該信哪一個。

### 3.9 `public.players_story_progress` — 玩家踩過哪些 beat（真相來源）

```sql
INSERT INTO players_story_progress (player_id, beat_id)
VALUES ('<player_id>', 'beat_bopiliao_reveal')
ON CONFLICT (player_id, beat_id) DO NOTHING;
```

`check_beat_unlockable()` 判斷 `prerequisite_beat_ids` 是否都滿足，查的就是這張表；`PRIMARY KEY (player_id, beat_id)` 天然防止同一個 beat 被重複記錄。

---

## 4. 任務細節（含 `quests.steps` 範例）

### 4.1 龍山寺：5 張特定物件照片（`q_longshan_ceremony`）

原大綱標「待定，建議實際前往決定」。在實地勘查（Phase 1，全員同行）前，先給一組**可替換的暫定清單**，方便先把任務資料結構跑起來：

| 拍攝點（暫定）             | 說明                                         |
| -------------------------- | -------------------------------------------- |
| 前殿（三川殿）屋脊剪黏     | 入口意象，最好辨認                           |
| 正殿觀音殿香爐             | 對應信件中「向觀世音菩薩祈求平安」的線索伏筆 |
| 後殿（媽祖殿或其他配祀殿） |                                              |
| 東護龍                     |                                              |
| 西護龍                     | 東西對稱，呼應彩蛋「回」字結構的一圈         |

```json
[
  {"step_id": "s1", "description": "前往前殿拍攝屋脊剪黏", "objective_type": "photo_submission"},
  {"step_id": "s2", "description": "前往正殿拍攝香爐", "objective_type": "photo_submission"},
  {"step_id": "s3", "description": "前往後殿拍攝特定物件", "objective_type": "photo_submission"},
  {"step_id": "s4", "description": "前往東護龍拍攝特定物件", "objective_type": "photo_submission"},
  {"step_id": "s5", "description": "前往西護龍拍攝特定物件", "objective_type": "photo_submission"}
]
```

> ⚠️ 五個拍攝點的路線本身就是彩蛋「繞行一圈＝陪阿明回家」的實際體感來源，建議勘查時特別留意動線是否真的能繞成一圈，這個彩蛋的成立與否取決於現場走法。

### 4.2 西門紅樓：兩個定點（`q_hongluo_ceremony`）

已在原大綱給定，不需要「待定」：

```json
[
  {"step_id": "s1", "description": "前往八角樓內部拍攝中央天花板設計", "objective_type": "photo_submission"},
  {"step_id": "s2", "description": "前往十字樓外部拍攝「河岸留言」文字（含建築外觀）", "objective_type": "photo_submission"}
]
```

### 4.3 剝皮寮：3 張塗鴉照（`q_bopiliao_ceremony`）

暫定紅磚建築區域內取 3 個不同立面的塗鴉牆，避免玩家在同一面牆左右移動兩步就拍完三張（會讓任務感覺敷衍）。建議勘查時挑選**視覺風格差異明顯**的三處（例如寫實派、文字塗鴉、抽象色塊各一），順便呼應「阿明畫工差」這個線索——玩家看過真正的塗鴉藝術後，回頭聽靈魂調侃阿明的畫，反差會更有趣。

---

## 5. 對話設計 / 防呆補完

### 5.1 龍山寺靈魂 — 資訊來源怎麼交代（對齊 §1 的內容安全調整）

> 玩家問：「你怎麼知道那是阿明？」
> 回應方向：靈魂說是**阿明自己在參拜時報上名字**，或是**聽他對著少女喃喃自語提過**——用「我剛好在旁邊，聽見了」取代「神明告訴我」，把資訊來源錨定在靈魂自身的「在場」與「觀察」，而不是假借神明權威。這樣既保留了懸疑感，也不觸碰 §12.2 的 taboo（不代替神明發言、不做教義性陳述）。

### 5.2 西門紅樓靈魂

> 玩家問：「你為什麼對這幅畫這麼有興趣？」
> 回應方向：紅樓靈魂坦率承認自己就是單純覺得「這是個好故事」，帶點自嘲——這比讓她假裝深情更符合她活潑健談的人設，也讓三個靈魂對阿明的「動機」形成清楚對比：龍山寺是淡然的旁觀、紅樓是好奇的收藏癖、剝皮寮是切身的、逐漸想起的痛。

> 玩家在任務完成前追問細節：
> 回應方向：用「等你逛完再說，不然我這故事要分兩次講很沒效率」這類帶點俏皮的方式帶開，維持語氣一致性，不要用制式的「請先完成任務」。

### 5.3 全域帶開語氣（三個靈魂共用的原則）

任務未完成前提及阿明，**不要用同一句罐頭台詞**——三個靈魂個性不同，帶開的方式也該不同：

| 靈魂     | 帶開語氣方向                                   |
| -------- | ---------------------------------------------- |
| 龍山寺   | 從容地把話題轉回眼前的事，像長輩不疾不徐地打斷 |
| 西門紅樓 | 俏皮地賣關子，強調「效率」或「順序」           |
| 剝皮寮   | 真實地表示自己想不起來、需要一點提示才有辦法想 |

這三種語氣差異寫進各自的 `character_personas.tone_override`，或 `story_beats.contingency_notes`。

### 5.4 剝皮寮靈魂 — 「想不起來」狀態（原 `beat_bopiliao_vague`）

這一段**不是劇情節點**（理由見 §3.4 說明二），而是 `beat_bopiliao_gate` 的 `contingency_notes` 內容：

> 玩家在尚未持有畫作時提及阿明，或反覆追問：
> 回應方向：表示「好像有點印象」但想不起來，語氣是真實的困惑而非敷衍推託——她不是在守密，是真的忘了。可以主動表示「你要是有什麼東西能讓我看看，說不定我想得起來」，給玩家一個模糊但成立的方向提示，不直接說出「去找畫」。

---

## 6. 內容安全對照表（§12.2 逐條檢查）

| §12.2 要求                                                           | 本劇本現況                                                                                         | 是否需調整                                                                                      |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `taboo_topics` 涵蓋教義解釋、神祇位階、靈驗與否、占卜結果、宗教比較 | 劇本提到「拜月老還願」、「向觀世音菩薩祈求平安」，皆為**敘事背景**，未涉及靈驗判定或教義裁決 | 不需調整劇本本身，但腦袋 B4/B5 的攔截邏輯要能分辨「劇情敘事引用」與「玩家真的在問運勢」兩種情境 |
| `not_this_character` 排除「廟方人員」「任何神祇」「宗教解說員」     | 原大綱「神明告訴它」已修正                                                                         | ✅ 見 §5.1                                                                                     |
| 不對信仰內容作教義性陳述或裁決                                        | 劇本無此類陳述                                                                                     | 不需調整                                                                                        |
| 「城市靈魂」不扮演真人                                                | 原大綱「剝皮寮靈魂＝女孩」已改為投影／記憶被吸收                                                   | ✅ 見 §1、§2.4                                                                                |

---

## 7. 情感節奏與共鳴設計建議

建議把 `beat_bopiliao_reveal` 設計成**共鳴值加成的敘事觸發點**（而非單純的劇情解鎖）：玩家走到這一拍時，情感重量最大，適合搭配：

- 剝皮寮靈魂語氣從破碎轉清醒的瞬間，UI 上可考慮短暫的視覺提示（不在本文件範圍，交給臉／F4 評估）
- 完成 `beat_finale_reveal` 後，三個靈魂的日常問候語（`canned_greetings`）可以各自加入一句「知情後」的變化版本，讓玩家感覺到「我知道的事，靈魂們也記得我知道了」——這是低成本但很有效的沉浸感手段，不需要額外對話生成，寫死幾條 `canned_greetings` 即可。

> `canned_greetings` 的外鍵指向 `(character_id, version)`，所以「知情後版本」的問候語會綁在當時生效的那一版人格上。人格改版時這幾條要一起帶過去，不會自動繼承。

---

## 8. 彩蛋整理

| 彩蛋               | 說明                                                               | 備註                                                                          |
| ------------------ | ------------------------------------------------------------------ | ----------------------------------------------------------------------------- |
| 龍山寺「回」字動線 | 玩家依任務繞行寺廟一圈，象徵「陪阿明回家」                         | 動線是否真能繞成一圈，待實地勘查確認（見 §4.1 提醒）                         |
| 阿明死亡年份推算   | 出生年份（嘉慶二年 / 1797）＋ 死亡歲數（18）＝ 1815 淡水地震       | 數字已核對一致，可直接使用                                                    |
| 畫工差的呼應       | 阿明用土炭塗鴉畫工不好 → 剝皮寮靈魂調侃「跟塗鴉一樣，像小孩畫的」 | 建議 §4.3 選塗鴉點時特意挑一處風格明顯較「稚拙」的牆面，讓這句調侃有視覺對照 |

---

## 9. 前置相依與待決事項

### 9.1 硬前置（沒有這些，這條 arc 跑不起來）

| # | 項目                                       | 現況                             | 卡住什麼                                                              |
| - | ------------------------------------------ | -------------------------------- | --------------------------------------------------------------------- |
| 1 | 萬華區行政區界 GeoJSON                     | ❌ 無                            | `districts.boundary` → 圍欄恆假 → 信件發不出去 → 整條 arc 無入口 |
| 2 | 西門紅樓、剝皮寮的史實研究 | ✅ 已完成並經 Lead 覆核（`landmark/ximen_red_house.md`、`landmark/bopiliao_historic_block.md`） | — |
| 3 | 兩張新人格卡草稿 + 人工審核 flip`active` | ❌ 無                            | `characters` 不存在 → `story_beats.character_id` 外鍵插不進去    |
| 4 | 兩個新`spirits` 記錄（含召喚點座標）     | ❌ 目前只有`longshan_temple`   | 玩家召喚不到紅樓／剝皮寮靈魂                                          |
| 5 | 九張表的 migration                         | ❌ 全部未建                      | 無處可寫                                                              |
| 6 | 實地勘查（三個地標的拍攝點）               | ❌ 未執行                        | §4 三組`steps` 皆為暫定                                            |

> 龍山寺自己的人格卡目前也是 `active=false` 的未審核草稿（內容為 `PENDING_NARRATIVE_REVIEW` 佔位），連垂直切片都還沒過。

### 9.2 待決事項

- [ ] `beat_bopiliao_reveal` 是否要額外設計共鳴值加成或視覺提示，待與臉／腦袋討論可行性
- [ ] `ximen_red_house_collector` / `bopiliao_keeper` 的最終 `character_id` 命名，隨兩張人格卡一併定案
- [ ] `story_beats` / `resonance_unlockables` 要不要加 `reviewed_by` 欄位——`narrative_directive` 會直接注入 prompt，內容治理風險與人格卡同級，但目前 schema 沒有任何審核欄位（見 SOP §5）

### 9.3 建表時程

**建表不等待內容。** 九張表的 migration 可以先做，空表無害，而早建早撞問題（#46 的 PostGIS 就是這樣提前撞出來的）。內容照樣等三個地標齊備再匯入。

v0.1 曾建議「等三地標齊備後再建表」，本版改掉——那會讓 schema 問題累積到內容完成後才一次爆發，屆時每個問題都是一支要改既有資料的 migration。

---

## 附錄 A：v0.1 → v0.2 修訂

| 項目                              | v0.1                                         | v0.2                                               | 理由                                                                        |
| --------------------------------- | -------------------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------------- |
| `beat_bopiliao_reveal` 觸發條件 | 「持有畫作，**且對話提及龍山寺回憶**」 | 只留「持有畫作」                                   | 後半句需要語意理解，body 狀態機判不出來；且前置鏈已遞移保證                 |
| `beat_finale_reveal` 觸發條件   | 「完成後玩家再次詢問」                       | 「完成後的下一次對話」                             | 同上，改為可判定的措辭                                                      |
| `beat_bopiliao_vague`           | 獨立 beat，`one_time=false`                | 移除，改為 §5.4 的`contingency_notes`           | `players_story_progress` 主鍵只能記一次，無法表達重複觸發；且無節點依賴它 |
| `character_id`                  | `char_longshan` 等佔位                     | `longshan_watcher`（現有實際值）+ 兩個待定建議名 | 佔位值對不上 DB                                                             |
| 表清單                            | 八張                                         | 九張（補`public.quests`）                        | §4 引用了`quests.steps` 與 `quest_id`，該表同樣未建                    |
| `resonance_unlockables.content` | 未說明是台詞或指令                           | 明確標示為敘事指令                                 | 欄位名易誤解，寫成成品台詞會繞過語氣層                                      |
| `districts.boundary`            | 「待補」                                     | 標為阻塞項                                         | 沒有它整條 arc 沒有入口，不是可以延後的待辦                                 |
| 建表時程                          | 建議等三地標齊備再建                         | 建表先行，內容後補                                 | 避免 schema 問題累積                                                        |
| 前置相依                          | 散在各處                                     | 集中為 §9.1 表格                                  | 六項硬前置目前全部未完成，需要一眼看得到                                    |
