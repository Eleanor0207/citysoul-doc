# 萬華主線運行架構 v1（無景觀拍攝辨識）

> **狀態：方案 B／完整工程交付草案。**本文件與景觀辨識 v1 互斥；不呼叫影像辨識、不傳送照片，其他主線規格相同。
>
> 此方案不傳送玩家照片、不呼叫地標影像辨識，亦不改動既有 `landmark-photo` API。對話與歷史文案以 [無景觀辨識完整互動腳本 v1](wanhua_district_storyline_aming_no_landmark_photo_v1.md) 為準。

## 1. 決策摘要

玩家仍必須真的到場：以既有 **Session + Encounter Token** 為 50m 到場邊界。到場後，玩家在該地標完成數個預先審核的公共導覽觀察點，做一次反思選擇，回到靈魂處交件。

```text
有效 Encounter → 接取對話 → 點按公共導覽觀察 → 反思選擇 → 交件 → 物件／Beat／下一站
```

不拍攝、不上傳影像、不做電腦視覺辨識。這降低相機權限、辨識配額、模型誤判與場域拍攝敏感度，但到場防偽強度只依 Encounter Token，不包含景觀辨識的第二道證據。

## 2. 不變的架構與產品原則

- 萬華固定由龍山寺入口開始，所有必要 Beat、任務與結局都在 `district_id=wanhua`；不強制跨區。
- 可跨日、無期限。未完成任務不扣親近度、不失去物件、不重置進度。
- Body（FastAPI／PostgreSQL）是任務、物件、Beat、選項與共鳴的唯一真相；Brain 只能在 Body commit 後產生受人格卡和 taboo 約束的補充台詞。
- Unity 只呈現導覽、對話與任務 UI，不能以本機旗標決定完成、發物或推進 Beat。
- 不保存原始 GPS、位置歷史、軌跡、影像、EXIF 或可回推的衍生資料。此方案根本不需要相機權限或影像傳輸。
- 各靈魂親近度只增加可選導覽短札與細節：龍山寺談修復與日常照看、紅樓談空間與用途、剝皮寮談居住與街區記憶；不得隱藏核心歷史、主線任務或任何區域入口。
- 資訊卡、合作優惠、商家／場館活動、分享與帳號登入都不是任務條件。

## 3. 任務狀態與互動規格

```text
LOCKED → AVAILABLE → IN_PROGRESS → READY_TO_TURN_IN → COMPLETED
```

| 狀態 | 後端條件 | Unity 行為 |
| --- | --- | --- |
| `LOCKED` | 前一 Beat／物件不足 | 僅顯示區域引導。 |
| `AVAILABLE` | 已滿足前置、尚未接取 | 靠近地標並召喚後可接取。 |
| `IN_PROGRESS` | 接取對話已確認 | 顯示 N 個核可觀察點與 0/N 進度。 |
| `READY_TO_TURN_IN` | 所有觀察點與一個反思選擇已確認 | 顯示回去對話的提示。 |
| `COMPLETED` | quest、物件、Beat 在同一交易完成 | 顯示物件、資訊卡與同區下一站理由。 |

每個觀察點必須有文字名稱、圖示、色彩以外的狀態提示與至少 44×44 pt 點按區。玩家點按「我看見了」後，Unity 以當次 Encounter Token 提交 `quest_id`、`point_id`；後端驗證 token、地標、任務前置、point 白名單和去重。App 中斷或跨日後只讀取後端進度，不做本機補寫。

現場施工、封閉或安全問題時，服務端可回傳內容審核過的示意圖模式；示意圖與現場導覽給予相同進度，並只記錄內容版本，不記錄玩家位置。

### 3.1 對話閱讀體驗（MVP 必要）

劇本以分支後匯流控制成本：選擇寫入旗標，改變後續台詞、短札與可選導覽細節，但不製造錯過結局。Unity 必須提供可調文字速度、首次點按完成逐字、第二次前進、只跳過已讀文字、選項暫停、自動播放與最近 100–200 句回顧。後端持久化 Beat／物件／選擇；客戶端只保存 UI node、已讀文字與閱讀設定，不能藉此自行完成任務。

## 4. 萬華三站任務

| 任務 | 前置 → 交件 | 完成條件 | 核可觀察點（均待實勘／審核） |
| --- | --- | --- | --- |
| `q_longshan_repair_trace` | `beat_longshan_gate` + 信件 → `beat_longshan_clue` + `item_homeward_painting` | 有效 Encounter + `ls_preserved_detail` + `ls_repair_trace` + `repair_lens` | 公共建築留存細節、可公開的修復線索。不可指向神像、供品、參拜者、儀式或個人。 |
| `q_redhouse_two_forms` | 龍山寺線索 + 回家的畫 → `beat_redhouse_clue` + 畫背殘句 | 有效 Encounter + `rh_octagon_hall` + `rh_cross_hall_connection` + `building_lens` | 八角堂外觀、與十字樓相接關係。不依賴市集、展演、店家或人潮；對話保留「紅樓 1908 年才出現」的史實界線。 |
| `q_bopiliao_street_lines` | 紅樓線索 + 兩份畫作物件 → `beat_bopiliao_reveal` + 年代簿結尾 | 有效 Encounter + `bp_doorway_line` + `bp_eave_line` + `bp_arcade_line` + `reveal_lens` | 公共可見的門洞、屋簷、騎樓線條。不可要求看住戶室內、門牌、路人、車牌、塗鴉或可變動招牌。 |

反思選項皆無正誤，僅影響台詞與個人年代簿：`repair_lens`（留存／修復／兩者）、`building_lens`（形狀／用途／再看）、`reveal_lens`（街／人／回家）。任務完成後依序給保底台詞、物件／年代簿頁、可略過歷史卡、同區下一站理由與手動方向提示。

**MVP 年代簿定案：**結局只產生一頁不超過 90 字的個人化短札，由 `story_focus`、`reveal_lens`、`ending_mark` 三次選擇的預寫片段組合；親近度只可微調用詞／排序。每日重訪任務、餘白短札與輪播不納入 MVP。

## 5. 後端資料與 API 提案

現有 API 與 schema 不可被假定已經支援 story quest observation；若採此方案，後端需依 code-first 實作並更新 OpenAPI。

```yaml
quest_definition:
  quest_id: q_longshan_repair_trace
  arc_id: wanhua_homeward_painting
  district_id: wanhua
  place_id: longshan_temple
  prerequisite: { beats: [beat_longshan_gate], inventory: [item_wanhua_letter] }
  observation_points:
    - { id: ls_preserved_detail, required: true }
    - { id: ls_repair_trace, required: true }
  reflection: { choice_key: repair_lens, required: true }
  turn_in: { dialogue_beat: beat_longshan_clue, grants: [item_homeward_painting] }
```

| 動作 | Unity 傳送 | 後端驗證／回傳 |
| --- | --- | --- |
| 讀取狀態 | arc ID | 可見 quest、已確認觀察點、下一個同區目標、內容版本。 |
| 確認觀察 | `quest_id`、`point_id`、Encounter Token | 地標、前置、白名單、去重；回傳觀察進度。 |
| 提交反思 | `quest_id`、白名單 choice、Encounter Token | 所有必要點完成；回傳 `READY_TO_TURN_IN`。 |
| 交件 | `quest_id`、對話／選項 ID、Encounter Token | 任務 ready、Beat 順序與冪等；回傳物件、Beat、預寫台詞與下一站。 |

```text
驗證 Session + Encounter 與前置
  → 去重寫 observation／choice／quest progress
  → 交件時同一交易寫 inventory + players_story_progress + player_story_choices
  → commit → 可選 Brain 台詞；失敗回退預寫台詞
```

## 6. Unity 實作範圍

| 元件 | 要做 | 不做 |
| --- | --- | --- |
| `QuestStatePresenter` | 依伺服器狀態呈現及重連刷新。 | 用 PlayerPrefs 判定進度。 |
| `ObservationOverlay` | 顯示已審核點、導覽文字、無障礙點按及完成回饋。 | 用自由相機、影像辨識或本機結果驗收。 |
| `QuestDialogueBridge` | 接取、反思、交件與可重看文字。 | 客戶端發物或推 Beat。 |
| 權限與網路 | 只請必要位置／相機以外的權限；相機不納入主線流程。 | 顯示拍照任務、上傳影像、呼叫 landmark-photo API。 |

## 7. QA 與採用門檻

- 無 Encounter Token 的觀察、反思或交件必須拒絕；重送不產生第二份物件或共鳴。
- 少任一觀察點或反思不可交件；跳過資訊卡仍可完成。
- 斷網、背景化、跨日返回後，已確認觀察點保留；未確認點可重試。
- 完成萬華不開啟其他區域的強制前置；合作頁不影響任何故事狀態。
- 開發／內測可用標示 `DRAFT`／`INTERNAL` 的內容，不將人工審核作為開發阻塞；但正式公開啟用前仍必須完成 7 個公共觀察點的安全、無障礙、歷史與場域審核，並定義施工／封閉時的替代示意圖。

## 8. 與景觀辨識方案的決策比較

| 面向 | 本方案：無辨識 | 現行方案：有辨識 |
| --- | --- | --- |
| 玩家動作 | 到場、觀察、反思 | 到場、觀察、拍公共景觀、反思 |
| 到場證據 | Encounter Token | Encounter Token + 景觀辨識嘗試 |
| 隱私／場域敏感度 | 不取用影像，最低 | 即用即丟，但仍需相機與場域構圖規則 |
| 誤判／挫折 | 無影像模型誤判 | 辨識失敗不擋主線，但玩家可能想重拍 |
| 開發與維運 | observation 狀態機與實勘 | 再加相機、multipart、配額、辨識品質與構圖維護 |
| 導覽記憶點 | 偏慢看與對話 | 多一個「我親自拍下來」的儀式感 |

建議先用本方案做龍山寺垂直切片，再以小規模實測判斷景觀拍攝是否明顯提升到場感、完成率或城市記憶；若沒有，就不必為了技術而加入辨識。
