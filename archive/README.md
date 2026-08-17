# 封存文件

> ⛔ **這裡的文件全部已封存，不得作為實作依據。**

## 這些是什麼

專案決策鏈的早期文件（第 1–8 層）。它們曾經是權威，內容也寫得詳細——但描述的是已被推翻的設計：Flutter 客戶端、2.5D Billboard、viseme 時間軸、Google Maps SDK、Geofencing API、任務失敗重試上限。

## 為什麼搬到這裡

在 2026-08-17 之前，這幾份文件住在 `citysoul-backend/docs/`，而該 repo 的 `.gitignore` 以 `docs/*` 排除整個目錄。其中兩份（`業務規則與API契約`、`acceptance_criteria`）當時仍是**現行權威**——也就是說，專案一半的業務規則只存在單一台機器上，無版本、無備份、其他協作者讀不到。

SDD v2.2 把它們的內容吸收進 §18（業務規則）與 §19（驗收標準），文件本身封存於此。先進版控、再封存，順序不可顛倒。

## 內容去哪了

| 封存文件 | 現行位置 |
|---|---|
| `city_soul_AR_業務規則與API契約.md` | SDD v2.2 **§18** |
| `city_soul_AR_acceptance_criteria.md` | SDD v2.2 **§19** |
| `city_soul_AR_frontend_interaction.md` | 距離模型與 Sense Token → SDD **§18.1／§18.2** |
| `city_soul_AR_WBS_API.md` | 模組職責 → SDD **§6** |
| `city_soul_AR_schema_api_sprint.md` | schema → `citysoul_data_schema.md`；排程 → SDD **§13** |
| `city_soul_AR_document_index.md` | 導覽對象已封存；接手者改讀 SDD **§1** |

## 已知會誤導人的地方

| 封存文件寫的 | 現行規格 |
|---|---|
| 任務失敗當天最多重試 3 次，超過鎖定 | **已移除**，憑證過期不計失敗（§18.4.2） |
| 回應含 `daily_limit_reached` 狀態 | **已移除**（§10.4） |
| 地圖用 Google Maps SDK ＋ Tile Overlay | Unity 自建 raster tile renderer（§1） |
| 距離偵測用 OS 原生 Geofencing API | 前景輪詢，間隔 10 秒（§5.5） |
| B10 產出 viseme 時間軸 | 只回音檔 URL，對嘴改客戶端即時分析（§8） |
| 前端為 Flutter | Unity 6（§4.1） |
| 本機地標辨識、照片不上傳 | 雲端 Gemini 辨識、即用即丟（見 `../city_soul_AR_landmark_recognition.md`） |
