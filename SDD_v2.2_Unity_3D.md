# 城市靈魂 — 系統設計文件（SDD v2.2）

> **版本**：v2.2 ／ 2026-08-17
> **取代對象**：`SDD_v2.1_Unity_3D.md`、`SDD.md`（v1，Flutter + 2.5D Billboard 版本）
> **本次變更性質**：**吸收 v1 業務規則與驗收標準，成為真正的單一權威**；修正三項行為缺陷；補齊內容生產管線的規格缺口。客戶端技術棧與視覺呈現層維持 v2.1 定案。
>
> 本文件為單一權威來源。凡與任何其他文件衝突者，一律以本文件為準。
> 標示【待確認】者代表仍需負責人拍板，不是本文件擅自決定。

### 📐 編號原則（v2.2 新增，請務必遵守）

**§0–§17 的章節編號與 v2.1 完全一致，不重排。** 後端有 12 個原始碼檔案、兩個 repo 有數十張 issue 直接引用「SDD v2.1 §6.4」「§7.3.1」「§12.2」這類編號；重排會讓那些引用全部靜默失效，而失效的引用不會有任何錯誤訊息提醒。

因此 v2.2 的做法是：**修正就地改寫，新內容附加為 §18–§20**。既有引用只需把「v2.1」讀成「v2.2」，章節位置不變。

---

## 0. 本次架構變更摘要

### 0.1 v2.2（本次）

一句話：**業務規則不再散落在本機文件，全部收進 SDD；同時修掉三個會影響玩家的行為缺陷，補上內容管線的入口。**

| # | 決策 | 狀態 |
|---|---|---|
| 1 | **吸收 v1 業務規則與驗收標準**（新增 §18、§19）。`city_soul_AR_業務規則與API契約.md` 與 `city_soul_AR_acceptance_criteria.md` 自本版起**封存**，不再是權威 | ✅ 定案 |
| 2 | **任務失敗語意修正**：憑證過期**不再**計為失敗嘗試，移除「當天最多 3 次」上限與 `daily_limit_reached` 狀態（§18.4） | ✅ 定案，**破壞性契約變更** |
| 3 | **人格卡注入成為所有生成路徑的必要輸入**：B9／B11／任務包裝台詞比照 B2，一律注入人格卡與 taboo（§6.4、§18.7） | ✅ 定案，**安全修正** |
| 4 | **新增 B14 引導提問生成**：補 v2.1 §5.2 只提元件、未定義內容來源的規格缺口（§18.8、§20.2） | ✅ 定案 |
| 5 | **新增 §20 內容生產管線**：當日情境的合格輸入由誰生產、任務目錄、故事線推進，三者從「表建好了但沒有入口」變成有規格 | ✅ 定案 |
| 6 | **新增 §5.8 效能與熱治理**：`targetFrameRate`、熱降級階梯、相機串流暫停；基準機型定義三檔 | ✅ 定案 |
| 7 | AC16.2 由「**平均**幀率 ≥ 30fps」改為「**全程** ≥ 30fps」，與 client#4 一致 | ✅ 定案 |

> **v2.2 不改變的**：客戶端技術棧、3DoF 實境疊層方案、資產產線與交付規格、對嘴方案、美術方向、Repo 結構、七階段排程骨架。這些在 v2.1 定案，本次未複審。

### 0.2 v2.1（沿用）

一句話：**客戶端從 Flutter + 2.5D 平面立牌，改為 Unity + 3D 角色 + 3DoF 感測器定向的實境疊層。後端完全不動。**

| # | 決策 | 狀態 |
|---|---|---|
| 1 | 客戶端引擎改用 **Unity 6**，Flutter 全面退場，不採用 Godot | ✅ 定案 |
| 2 | **不做真 AR**（不裝 AR Foundation / ARCore / ARKit / Geospatial），改用 **3DoF 感測器定向實境疊層** | ✅ 定案 |
| 3 | 角色改為 **3D 模型**，美術方向為 **半寫實 × 混合靈魂感**（主體實體感 ＋ 輪廓發光 ＋ 腳部淡出） | ✅ 定案 |
| 4 | 對嘴改為 **uLipSync 客戶端即時音訊分析**，推翻 B10 viseme 時間軸設計 | ✅ 定案 |
| 5 | 開發節奏改為 **3 天一個 Sprint**，排程重構為 **7 階段垂直切片** | ✅ 定案 |
| 6 | 垂直切片首個地標改為 **龍山寺** | ⚠️ 定案但有風險，見 §12 |
| 7 | 資產產線**暫定 ComfyUI / Hunyuan3D**，CC4 為備援 | ⚠️ 暫定，Phase 0 後複審 |

> ⚠️ **「AR」這個詞在本專案有精確含意**：目前採用的是**實境疊層**（相機背景 ＋ 3DoF 方位 ＋ 3D 角色），**不是** ARCore／ARKit 的 SLAM 空間錨定。§12 的耗電評估、§4.1 的無認證裝置白名單、AC17.3 的權限清單全部建立在這個前提上。若日後改採真 AR，§15.1 的三個前置條件必須先做完，且上述三項紅利同時作廢。

---

## 1. 決策推翻對照表（最重要的一節）

接手開發的人請先讀這張表。專案歷史上已換過三次技術棧（Unity → Flutter 2.5D → Unity 3D），舊文件仍在專案內，容易誤讀。

| 項目 | 舊版 | v2.1 修正後 | 理由 |
|---|---|---|---|
| 客戶端引擎 | Flutter | **Unity 6** | 3D 角色＋動畫狀態機＋相機疊層，且為團隊現有能力 |
| 曾評估的替代 | Godot | **不採用** | Android 手機 AR 僅有研究等級社群外掛（`GodotVR/godot_arcore`：0 release、11 open issue），且缺乏跨平台 AR 抽象層 |
| 視覺呈現 | 2.5D Billboard（Rive/Lottie） | **3D mesh ＋ skeletal animation** | 目標體驗需要體積感與真透視 |
| 空間定位 | 無（純相機疊圖） | **3DoF 感測器定向**（陀螺儀＋電子羅盤） | 角色留在固定方位，玩家需轉動手機尋找 |
| 真 AR / 空間錨定 | 明確排除 | **仍明確排除**，移入 Roadmap | 見 §3 |
| 對嘴方案 | B10 產 viseme 時間軸 → F5 播放 | **uLipSync 客戶端即時分析** | Google Cloud TTS **不提供** viseme／phoneme 時間軸，原設計不可實作 |
| B10 職責 | TTS ＋ viseme 時間軸 | **只做 TTS，回傳音檔 URL** | 同上 |
| F5 職責 | 播放 viseme 時間軸，不分析音訊 | **即時 MFCC 分析驅動 shape key** | 同上 |
| 素材交付 | Cloud Storage ＋ CDN（格式未定） | **Unity Addressables remote bundle** | Unity runtime 無法載入 FBX |
| 地圖底圖 | Google Maps SDK ＋ Tile Overlay | **Unity 自建 raster tile renderer** ＋ 同一套風格化圖磚 | Unity 無 Google Maps SDK 綁定；圖磚方案不變 |
| 距離偵測 | `geolocator` 前景輪詢 | **Unity `Input.location` 前景輪詢** | 純平台替換，門檻邏輯不變 |
| 狀態管理／路由／網路層 | Riverpod / GoRouter / dio | **Unity 場景管理 ＋ UnityWebRequest ＋ 自建 TokenManager** | Flutter 專屬方案全數作廢 |
| 安全儲存 | `flutter_secure_storage` | **Android Keystore（Unity Plugin 封裝）** | |
| Sprint 長度 | 兩週（隱含） | **3 天** | 本次定案 |
| 排程結構 | 7 Sprint 橫向堆疊 | **7 階段垂直切片，共約 27 個 3 天 Sprint** | 見 §13 |
| 垂直切片地標 | 天文館 | **龍山寺** | 本次定案，風險見 §12 |
| 人員代號 | A / B / C / D | **模組角色名（Lead／身體／腦袋／臉）** | 避免與工作包代號 B/S/F 衝突 |
| CONTEXT.md「完整 3D 角色」列為 Avoid | Avoid | **解除** | 本次改版核心 |
| CONTEXT.md「真 AR 角色」列為 Avoid | Avoid | **維持 Avoid** | 決策 2 |
| 裝置支援 | 全 Android/iOS | **維持全 Android/iOS**（不用 ARCore，無認證白名單） | 決策 2 的紅利 |
| 封測平台優先序 | 未定 | **Android 優先** | 定案 |

### 1.1 v2.2 修正對照表

| 項目 | v2.1（含沿用 v1） | v2.2 修正後 | 理由 |
|---|---|---|---|
| 業務規則的存放位置 | 「完全沿用 v1」，實際住在 `citysoul-backend/docs/`（**該目錄被 `.gitignore` 排除**） | **收進本文件 §18** | 權威規則只存在單一台機器上、無版本、無備份，不是權威而是風險 |
| 驗收標準的存放位置 | 指向 `city_soul_AR_acceptance_criteria.md`（同上，本機專用） | **收進本文件 §19** | 同上 |
| 憑證過期的語意 | 相遇憑證過期 = 一次**任務嘗試失敗**，`attempts_today += 1` | **不計失敗**，只清除當次嘗試，重新召喚即可再挑戰 | 手機的中斷（來電、日曬、被搭話）是常態，不是玩家做錯事 |
| 每日失敗上限 | 當天最多 3 次，超過鎖定至隔天 | **移除**。`attempts_today` 降為純觀測欄位，不參與判定 | 防刷已由 `resonance_events` 的 UNIQUE 約束達成；此上限防的是不存在的問題 |
| `quest.status` 值域 | 含 `daily_limit_reached` | **移除該值** | 上限機制取消後它沒有語意 |
| B9／B11／任務包裝的 prompt 輸入 | 只有 spirit_id ＋ 該路徑自身的參數 ＋ B5 史實邊界 | **一律注入人格卡**（含 `taboos`、`not_this_character`） | §12.2 的宗教安全下限只掛在 B2 對話路徑，其餘三條路徑繞過它 |
| 引導提問的內容來源 | 未定義（v2.1 §5.2 只寫「有這個元件」） | **B14 生成，每日隨當日情境變動**（§18.8、§20.2） | 它已被定位為主體驗；靜態清單會讓靈魂顯得像罐頭 |
| 當日情境的合格輸入 | 定義了三類白名單，未定義**誰生產** | **§20.1 定義生產者**：節慶日曆表 ＋ 預寫輪播池 ＋ 排程 | 白名單恆為空 = 每天都走 fallback = 核心迴圈的「隔天有變化」不成立 |
| 任務目錄 | 未定義（實作為每靈魂固定一個任務） | **§20.3 定義輪播規則** | 每靈魂單一任務時，共鳴值 5 次即封頂 |
| 幀率上限 | 未設定（Unity 預設 -1） | **`Application.targetFrameRate = 30`**（§5.8） | 預設值會在高階機上白燒一倍 GPU |
| 熱降級 | 無 | **三階降級階梯**（§5.8） | 相機＋3D＋即時音訊分析疊加是最高耗電情境 |
| AC16.2 幀率門檻 | 平均 ≥ 30fps | **全程 ≥ 30fps**，附最低值與掉幀分佈 | 平均會掩蓋週期性卡頓，而卡頓才是玩家感覺得到的 |

### 1.2 完全沿用、不要重寫的部分

- 全部資料庫 schema（見 `citysoul_data_schema.md`；除 §10.2 新增兩個欄位）
- 三種 Token 的設計與效期（§18.2）
- 150m 感應 / 50m 召喚雙層距離模型（§18.1）
- 共鳴值規則、配額機制、防作弊觀察期（§18.5、§18.6、§18.3）
- 腦袋模組 B1–B13 的職責定義
- 隱私原則與 Schema 硬約束（不得新增 `location_history` / `movement_trace` 類型的表）
- 時區 Asia/Taipei（UTC+8）

> ⚠️ 上列規則的**正文自 v2.2 起在 §18**，不再在其他文件。v1 的兩份文件已封存（§17.2）。

---

## 2. 產品概述

**核心迴圈**：玩家在指定地標完成「到場、召喚、當日情境短對話、可驗證微任務」的一次完整體驗，並有理由在隔天回來查看變化。

**首批玩家（TA）**：住在台北、約 18–35 歲，喜歡城市探索、角色互動與拍照分享的繁中使用者。MVP 語言統一為台灣繁體中文。

**封閉測試垂直切片**：以**單一龍山寺靈魂**驗證核心迴圈、內容生產、成本與隱私邊界，通過驗證後才擴展到首發十個城市靈魂。

> ⚠️ 此處與 CONTEXT.md 原定的「天文館靈魂」不同，屬本次改版的變更。內容治理風險見 §12.2。

---

## 3. 產品邊界（Avoid 清單，v2.1 更新版）

- ~~不做完整 3D 角色~~ → **改為採用 3D 角色**〔本次解除〕
- **不做真 AR**：不做 SLAM 空間錨定、平面偵測、遮擋判斷、深度估算〔維持〕
  - 技術含意：不引入 AR Foundation、ARCore、ARKit、Geospatial API
- **不做背景定位追蹤**，僅在玩家主動開啟召喚/相機/感應模式時前景即時運算〔維持〕
- 原始 GPS 只在驗證期間使用後即丟棄，不形成軌跡〔維持〕
- 世界記憶不含任何玩家輸入；玩家記憶僅屬單一玩家〔維持〕
- 可驗證微任務由後端確定性規則判定，不由 LLM 判定〔維持〕
- 「城市靈魂」是地標的擬人化集體意識，不扮演任何特定真人或歷史人物〔維持〕
- 玩家在 MVP 以文字輸入，不使用語音辨識〔維持〕
- 不做「經過咖啡店主動推薦」類商家內容〔維持〕
- **不對特定宗教信仰、神祇、儀式作出教義性陳述或裁決**〔本次新增，因應龍山寺切片〕

---

## 4. 技術棧

### 4.1 客戶端

| 項目 | 選型 | 備註 |
|---|---|---|
| 引擎 | **Unity 6**（URP） | 行動裝置最佳化 |
| 建置目標 | **Android（ARM64 only）** 優先 | 不出 ARMv7 |
| 3D 資產載入 | **Addressables**（remote bundle） | §7 |
| 對嘴 | **uLipSync** | §8 |
| 定位 | Unity `Input.location` 前景輪詢 | 門檻沿用 v1 |
| 方位 | `Input.gyro` ＋ `Input.compass` | 3DoF 核心 |
| 相機背景 | `WebCamTexture` → 全螢幕 RawImage | |
| 網路 | `UnityWebRequest` ＋ 自建攔截層 | |
| 安全儲存 | Android Keystore（Unity Plugin） | 不得用 `PlayerPrefs` |
| 建置 | **GitHub Actions ＋ GameCI** | §11.4 |

### 4.2 後端（不變）

Python / FastAPI ｜ GCP Cloud Run ｜ Vertex AI 單一快速模型 ｜ Cloud SQL for PostgreSQL ＋ pgvector ｜ Redis ｜ Google Cloud TTS ｜ Cloud Storage ＋ CDN。

開發期用本地 docker-compose（Postgres + Redis）。模型服務失敗時一律回退人工預寫台詞。

---

## 5. 客戶端架構（Unity）

### 5.1 3DoF 實境疊層方案〔本次新增〕

| 層級 | 手機轉動時 | 採用 |
|---|---|---|
| 0DoF 純疊層 | 角色永遠在螢幕正中 | ❌ |
| **3DoF 感測器定向** | 角色固定在某**方位角**，需轉動手機才看得到 | ✅ |
| 6DoF SLAM | 角色釘在世界座標，可繞行 | ❌ Roadmap |

**實作原則**：

1. 每個靈魂在 `spirits` 表設定方位角（§10.2）
2. 讀取 `Input.compass.trueHeading` 取得手機朝向
3. 角色螢幕水平位置 = f(靈魂方位角 − 手機朝向)；超出視野不繪製
4. 螢幕邊緣顯示方向指示器引導轉向
5. `Input.gyro.attitude` 補償俯仰/翻滾，維持角色水平站立
6. 角色縮放依 GPS 距離插值（沿用 v1 漸顯機制），非相機實算透視

**已知取捨**：無遮擋、無真實景深、羅盤受都市磁干擾、玩家平移時無視差。

**必做緩解手段**（非加分項）：接地 blob shadow（不透明度約 0.3）、相機畫面平均色溫匹配 ambient light、邊緣柔化、靈魂感 shader（§9）。

### 5.2 場景清單

| 場景 | WBS | 說明 |
|---|---|---|
| Bootstrap | S6 | 背景呼叫 `POST /players` 或帶出既有 session，無登入 UI |
| MapScene | S7 | 風格化圖磚地圖、三段式標記、150m 內嵌入對話、當日情境卡片 |
| EncounterScene | F4/F5/S9/S14 | 50m 召喚成立後 **additive load**；相機背景＋3D 角色＋可收合對話窗 |
| QuestScene | S8 | 任務狀態、嘗試次數 |
| ProfileScene | S8 | 共鳴值進度、相遇收藏、記憶摘要 |

對話 UI 做成共用 prefab，同時被 MapScene（sense_token 模式）與 EncounterScene（encounter_token 模式）使用。對話延續靠後端 Redis key（`player_id`+`spirit_id`）天然成立。

### 5.3 專案結構

```
Assets/_Project/
  Core/            # 常數、錯誤處理、事件匯流排
  Services/
    TokenManager.cs        # 三種 token 生命週期
    LocationService.cs     # 前景輪詢
    OrientationService.cs  # 陀螺儀＋羅盤（3DoF）      ★新增
    ApiClient.cs           # UnityWebRequest ＋ 攔截層
    AssetService.cs        # Addressables 載入與快取   ★新增
  Features/
    Map/  Encounter/  Quest/  Profile/  Dialogue/
  Characters/      # 🎭 臉的工作區（prefab / Animator / shader / 材質）
```

> **資料夾即分工邊界**：`Characters/` 由臉持有，`Features/` 與 `Services/` 由身體持有。兩人共用同一個 Unity 專案但不互相改對方資料夾，降低合併衝突。

### 5.4 TokenManager（邏輯沿用 v1）

| Token | 效期 | 換發時機 |
|---|---|---|
| Session | 90 天 | App 啟動時換新，玩家無感 |
| Sense | 30 分鐘 | 進入 150m 核發；剩 1–2 分鐘且仍在範圍內時**靜默**重呼 `/sense` |
| Encounter | 15 分鐘 | 50m 召喚成立核發；過期即任務嘗試失敗，不續期 |

儲存於 Android Keystore。三種 token 在後端須用不同中介層驗證，不可共用邏輯。

### 5.5 LocationService

- MapScene 前景：每 10 秒查一次位置，更新 150m / 50m 門檻狀態
- EncounterScene：連續讀取，供漸顯插值
- 切背景或離開 MapScene：停止輪詢，**不申請背景定位權限**

不使用原生 Geofencing API（v1 已論證）。v1 的「雙 Geofence 穩定性 POC」**取消**。

### 5.6 OrientationService〔新增〕

提供 `CurrentHeading`（真北方位角）與 `DeviceAttitude`；偵測 `headingAccuracy` 劣化並觸發 8 字校正提示；僅在 EncounterScene 啟用。

### 5.7 地圖

- 底圖：**Unity 自建 raster tile renderer**（quad 網格貼圖磚）
- 圖磚來源沿用 v1：OSM（ODbL）→ PyraCanny 線稿萃取 → Fooocus 風格化生成
- 三段式標記（不變）：>150m 預設樣式 ／ ≤150m 淡淡發光可聊天 ／ ≤50m 完全點亮可召喚
- 50m 觸發：玩家**主動點擊**召喚按鈕才呼叫 `/summon`，**不自動跳轉**
- 「你還不夠靠近」為模糊提示，不顯示精確距離數字

> ⚠️ Google Maps SDK 依賴已移除，手勢/縮放/定位點需自行實作。此為本次新增工作量，已提前至 Phase 2。

### 5.8 效能與熱治理〔v2.2 新增〕

EncounterScene 同時運行：相機串流、3D 角色渲染、uLipSync 每幀 MFCC 分析、網路請求、羅盤與陀螺儀。這是本專案最高耗電的情境，而玩家遭遇它的地點是**台北戶外、日照下、手持**。

#### 5.8.1 基準機型定義（補 §16 的 ★ 缺口）

| 檔 | 定義 | 用途 |
|---|---|---|
| **地板機** | 2022 年中階 Android（Snapdragon 6 系或 Dimensity 700/900 級，8 GB RAM） | **AC16.2 的驗收機**。過不了就降畫質，不是換機器 |
| 基準機 | 2023–24 中階（Snapdragon 7 系） | 日常開發回歸測試 |
| 天花板 | 團隊持有的最高階機 | 僅確認上限畫質有意義 |

地板機**必須是實體機**，模擬器測不出發熱。選型前先查台灣 Android 裝置分佈，不憑印象。

#### 5.8.2 幀率與畫質基線

- `Application.targetFrameRate = 30`、`QualitySettings.vSyncCount = 0`，於 Bootstrap 設定一次
- 相機預設 1280×720@30（`CameraFeed`）
- 進入 EncounterScene 時 `Screen.sleepTimeout = NeverSleep`；**離開時必須還原為 `SystemSetting`**

#### 5.8.3 熱降級階梯（ThermalGovernor）

單一元件訂閱系統熱狀態（Android `PowerManager.getCurrentThermalStatus()`，API 29+；iOS `ProcessInfo.thermalState`；兩者皆需 native 橋，Unity 無跨平台 API）：

| 熱狀態 | 動作 |
|---|---|
| NONE / LIGHT | 目標 30fps；相機 720p@30 |
| MODERATE | 相機降 480p@24；uLipSync 分析頻率減半；關閉靈魂感 shader 的呼吸與 dissolve 層 |
| SEVERE 以上 | **相機停止串流，凍結最後一幀作為靜態背景**；目標 24fps |

#### 5.8.4 相機串流暫停〔省電主要手段〕

角色為 3DoF 定向、**不做空間錨定**（§3），因此對話進行中相機畫面不需要逐幀更新：

- 召喚**進場動畫**期間使用即時串流——那是「靈魂自眼前街景浮現」的時刻
- 進入對話後 `WebCamTexture.Pause()`，凍結一幀作為背景
- 偵測到手機水平旋轉（`OrientationService` 已有訊號）時恢復串流

此項省下相機硬體、每幀紋理上傳與色彩轉換，成本僅一組 Pause/Play 開關。

#### 5.8.5 量測範圍

P0-D（client#4）的 5 分鐘測試驗證的是**元件**不會爆。另需一次**完整迴圈**量測（到場→召喚→當日情境→數輪對話→現場任務），記錄：

- 起訖電量％與經過分鐘 → 換算 **%/分鐘**
- 機身溫度（`adb shell dumpsys thermalservice`）
- fps 曲線，標出**開始降頻的時間點**
- 分段歸因：只開相機／只跑 3D／只跑對嘴／全開，四條曲線

**門檻**：地板機一次完整迴圈（目標 12 分鐘）耗電 ≤ 8%，全程未進入 SEVERE。8% 的理由是玩家一趟出門會玩 2–3 個靈魂，並且還要用導航與相機。

> 戶外可讀性需要**高螢幕亮度**，這是耗電項而非省電項，須計入預算。OLED 深色省電的一般建議在本專案不適用。

---

## 6. 模組邊界與工作包

三模組定義不變，唯一跨模組關聯欄位仍是 `player_id`。

### 6.1 🧠 腦袋 Brain（全部在後端）

| 代號 | 工作包 | v2.1 異動 |
|---|---|---|
| B1 | Vertex AI 串接＋失敗回退 | — |
| B2 | Prompt 組裝引擎 | — |
| B3 | 人格卡載入與版本管理 | — |
| B4 | 角色安全邊界檢查層 | ★**難度上升**（宗教場所，見 §12.2） |
| B5 | 史實邊界規則注入 | ★**難度上升**（同上） |
| B6 | 長期記憶（pgvector） | — |
| B7 | 短期記憶（Redis） | — |
| B8 | 記憶摘要 nightly batch | — |
| B9 | 當日情境內容生成 | — |
| B10 | TTS 串接 | ★**縮減**：移除 viseme 時間軸產出 |
| B11 | 共鳴值解鎖敘事生成 | — |
| B12 | 快速問候比對層 | — |
| B13 | 地標視覺辨識（雲端 Gemini，即用即丟） | — |
| **B14** | **引導提問生成** ★v2.2 新增 | 補 §5.2 規格缺口，見 §18.8、§20.2 |

> **v2.2 對 B9／B11 與任務包裝台詞的共同要求**：三者的 prompt 一律注入人格卡（見 §6.4）。這不是新工作包，是既有工作包的輸入修正。

### 6.2 🦾 身體 Body

| 代號 | 工作包 | 環境 |
|---|---|---|
| S1 | GPS 定位模組 | 客戶端 |
| S2 | 在場驗證與召喚半徑判定 | 後端 |
| S3 | 防作弊（mock location、速度合理性） | 後端 |
| S4 | 任務進度表＋狀態機 | 後端 |
| S5 | 共鳴值表＋判定邏輯 | 後端 |
| S6 | 匿名玩家身分／帳號升級綁定 | 後端＋客戶端 |
| S7 | UI：地圖／召喚點（★含自建 tile renderer） | 客戶端 |
| S8 | UI：任務列表／Profile | 客戶端 |
| S9 | 引導提問元件 | 客戶端 |
| S10 | 當日情境排程＋快取＋對外 API | 後端 |
| S11 | 訊號推播 | 後端＋客戶端 |
| S12 | 紀念照片：本機保存/分享＋呼叫 B13 | 後端＋客戶端 |
| S13 | 行銷／分享功能 | 客戶端 |
| **S14** | **3DoF 方位定向服務** ★新增 | 客戶端 |

### 6.3 🎭 臉 Face

| 代號 | v1 定義 | v2.1 定義 | 需 Unity |
|---|---|---|---|
| F1 | 角色美術造型設計（CC 產線） | ★概念設計 ＋ 生成規範制定 | ❌ |
| F2 | 精靈圖／序列幀烘焙管線 | ★Blender 清理／減面／UV／貼圖烘焙 | ❌ |
| F3 | Rive/Lottie 動畫實作 | ★Mixamo 綁骨 ＋ 動作庫 ＋ Blender shape key | ❌ |
| F4 | 相機疊圖渲染 | ★角色 prefab ＋ 靈魂感 shader ＋ 材質設定 | ✅ |
| F5 | 口型同步播放器（吃 viseme） | ★uLipSync 整合 | ✅ |
| F6 | 素材版本管理 | ★Addressables 打包 ＋ remote catalog | ✅ |
| F7 | App 端素材下載與快取 | ★下載進度／快取／版本比對 | ✅ |
| **F8** | — | ★Animator 狀態機（idle ↔ talking ↔ 揮手 ↔ 轉身） | ✅ |

> ⚠️ **人力缺口【待確認】**：F4–F8 需 Unity 能力（Animator、Shader Graph、Prefab、Addressables），屬**技術美術（Technical Artist）**範疇。
> 若臉不具備此能力，F4/F5/F7/F8 需由身體代持，**但身體已是關鍵路徑瓶頸**。
> 建議處理：① 臉補足 Unity 技能（Phase 0–1 期間）；② 或依「來不及會加人力」原則，補一名技術美術。
> 本文件排程**假設臉持有全部 F 模組**；若改由身體代持，Phase 2–5 需重新估時。

### 6.4 模組邊界設計原則

| 決策 | 內容 |
|---|---|
| 當日情境 | 內容生成（腦袋）與排程/快取/serve（身體）拆分 |
| ~~viseme 時間軸~~ | ★**廢除**：改為臉自行分析音訊 |
| 共鳴值解鎖 | 身體判定達標 → 呼叫腦袋生成敘事（順序不可顛倒） |
| 記憶 schema owner | 結構化欄位屬身體；語意化欄位屬腦袋 pgvector；僅靠 `player_id` 關聯 |
| **資產交付邊界** | ★**新增**：見 §7.3 |
| **Unity 資料夾邊界** | ★**新增**：臉持有 `Characters/`，身體持有 `Features/` `Services/` |
| **人格卡注入** | ★**v2.2 新增**：見下方硬規則 |

**Schema 硬約束（不變）**：不得新增任何 `location_history` / `movement_trace` 類型的表。

### 6.5 人格卡注入硬規則〔v2.2 新增〕

**凡是產出「城市靈魂說出口的文字」的生成路徑，一律注入該靈魂目前生效的人格卡，含 `taboos` 與 `not_this_character`。**

| 路徑 | v2.1 實際輸入 | v2.2 要求 |
|---|---|---|
| B2 對話 | 人格卡 ＋ 記憶 ＋ 當日情境 ＋ B5 | 不變（本來就對） |
| B9 當日情境 | place_id ＋ 三類合格輸入 ＋ B5 | **＋ 人格卡** |
| B11 解鎖敘事 | spirit_id ＋ 階段簡述 ＋ B5 | **＋ 人格卡** |
| 任務完成包裝台詞 | spirit_id ＋ quest_id ＋ B5 | **＋ 人格卡** |
| B14 引導提問 | — | **＋ 人格卡**（新路徑，一開始就要有） |

**為什麼這是安全規則而非品質規則**：§12.2 為龍山寺定義的宗教安全下限（不代神明應許、不預測吉凶、不比較信仰、不解籤）住在人格卡的 `taboos`。B5 史實邊界管的是史實／傳說／想像的分界，**不涵蓋宗教邊界**。因此未注入人格卡的路徑等於沒有那四條下限——而共鳴值第三階段的解鎖故事正是全遊戲語氣最深、最容易滑出邊界的一段。

**實作要求**：四條路徑共用同一個人格卡區段組裝函式，不各自複製一份。複製的版本會各自漂移，而漂移的那一份不會有人發現。

---

## 7. 資產產線與交付規格

### 7.1 產線（暫定路線）

```
① ComfyUI / Hunyuan3D    A-Pose 正面全身圖 → .glb 草模
② Blender 清理           分離四肢/頭髮/衣物、減面、UV、貼圖烘焙
③ Mixamo 綁骨            下載 Idle / Talking / Wave / Turn
                         ⚠️ Mixamo 會清除 blend shape，shape key 必須排在此步驟之後
④ Blender Shape Keys     JawOpen、Blink_L、Blink_R → 匯出 .fbx
⑤ Unity                  Animator(F8) → uLipSync(F5) → 靈魂感 shader(F4) → Addressables(F6)
```

### 7.2 ⚠️ 產線最大風險與備援

**風險**：AI 生成 mesh 常見「頭髮與肩膀熔接」「裙襬將雙腿黏連」「四肢邊界不清」，導致 **Mixamo 自動綁骨失敗**，且清理工時不可預測。

**備援（Character Creator 4）**：出廠即含骨架、臉部 rig 與標準 viseme，10 隻風格一致性易控管；為付費軟體、風格自由度較低。

**複審時點**：Phase 0 的 POC-資產 結束時，通過門檻見 §13.2。

### 7.3 資產交付規格書

臉交付給身體的每一隻角色必須符合以下規格，**交件前自檢**：

| 項目 | 規格 |
|---|---|
| 格式 | `.fbx`（含骨架、動畫、shape key） |
| 面數 | **15,000 – 20,000 tris** |
| 貼圖 | Albedo / Normal / ORM 各一，**2048×2048**，ASTC 壓縮 |
| 材質數 | ≤ 3（身體、頭髮、眼睛） |
| 骨架 | Mixamo 標準命名，Unity Humanoid rig 相容 |
| shape key 命名 | `JawOpen`、`Blink_L`、`Blink_R`（**大小寫敏感，不得增減或改名**） |
| 眼睛 | 獨立材質（供貼圖切換眨眼備援） |
| 動作 | `Idle`、`Talking`、`Wave`、`Turn` 四組完整 loop |
| 座標原點 | 腳底中心置於 (0,0,0)，面向 +Z |
| **資產包上限** | **< 10 MB**（Addressables 打包後） |

### 7.3.1 灰模交付規格（Phase 1）〔本次新增〕

**目的**：讓身體與臉在 Phase 1 即可開發 F4/F5/F8，不必等待正式角色。正式角色進場時**直接替換，不改動任何程式碼**。

**核心原則**：程式碼會接觸的一切必須與正式資產完全一致；純視覺部分不論。

**必須一致（硬規格）**：

| 項目 | 規格 |
|---|---|
| 骨架 | Mixamo 標準命名，Unity **Humanoid** rig |
| shape key | `JawOpen`、`Blink_L`、`Blink_R`（大小寫敏感） |
| 動畫 clip 名稱 | `Idle`、`Talking`、`Wave`、`Turn`；**Talking 長度 ≥ 10 秒** |
| 材質 slot | 3 個：`Body`、`Hair`、`Eyes` |
| 座標原點 | 腳底中心 (0,0,0)，面向 **+Z** |
| **身高** | **1.60 公尺**（Unity 1 unit = 1 m）——距離縮放插值基準，正式角色須一致 |
| **面數** | **15,000–20,000 tris**（不得偷懶做低面數，否則 P0-D／AC16.2 效能驗證失真） |
| Addressables key | `spirit_longshan`（與正式資產同一 key，僅換 bundle 內容） |

**Prefab 節點結構（必須定死）**：

```
spirit_root                 (Animator + uLipSync)
├── Armature/               (骨架)
├── Body    (SkinnedMeshRenderer, 含 shape key)
├── Hair    (SkinnedMeshRenderer)
└── Eyes    (SkinnedMeshRenderer)
```

> ⚠️ uLipSync 與 Animator 綁定的是 `SkinnedMeshRenderer` 的**路徑字串**。節點結構一旦變更，所有綁定失效需重接。此結構於灰模階段定案後，正式角色一律沿用。

**不論項目**：造型、比例、美感、貼圖（建議用棋盤格，可順便驗 UV）、頭髮與服裝細節。

**命名規則**：一律使用 spirit id，不使用角色名（角色名可能變更，不得進入技術命名）。
`spirit_longshan_v0`（灰模）／ `spirit_longshan_v1`（正式）。

**交付檢查表**：

```
□ Unity 匯入後 rig type 為 Humanoid，無警告
□ Blend Shapes 面板可見 JawOpen / Blink_L / Blink_R
□ JawOpen 拉至 1.0 時嘴部明顯開合且不破面
□ 四組動畫 clip 皆可 loop，Talking ≥ 10 秒
□ 面數 15,000–20,000 tris
□ 身高 1.60m，腳底貼齊 y=0，面向 +Z
□ 材質 slot 為 Body / Hair / Eyes
□ prefab 節點路徑符合上方結構
□ 已打包為 Addressables，key = spirit_longshan
```

> **附帶價值**：Mixamo 免費角色不含任何 blend shape，因此製作灰模必須實際走一次「Blender 加 shape key → 分離眼睛材質 → 匯出 .fbx」，即正式產線第 ④ 步的預演。灰模若製作不順，正式角色只會更困難。

> ⚠️ **授權**：Mixamo 角色僅供內部開發測試，**不得出現在封測或上線版本**。Phase 6 前須確認灰模已自 build 移除。

### 7.4 Addressables 交付鏈

打包 remote bundle → 上傳 Cloud Storage → 沿用既有 `GET /api/v1/assets/{avatarId}` 回傳 catalog／bundle URL 與 version。

**下載策略**：不預載全部角色；玩家進入某地標 150m 感應範圍時才觸發該角色下載。Profile 頁提供「清除快取」入口。

### 7.5 APK 體積預算

| 項目 | 目標 |
|---|---|
| APK 下載大小（不含遠端角色資產） | **< 100 MB** |
| 單一角色資產包 | < 10 MB |
| 繁中字型（子集化） | < 3 MB |
| 建置架構 | **ARM64 only** |

---

## 8. 對嘴方案（uLipSync）

### 8.1 推翻理由

v1 設計「B10 呼叫 TTS 時一併取得 viseme 時間軸 → F5 播放」。**此設計不可實作**：Google Cloud TTS 不提供 viseme 或 phoneme 時間軸輸出。此為架構缺陷，本次一併修正。

### 8.2 新設計

| 模組 | v1 | v2.1 |
|---|---|---|
| B10 | TTS ＋ viseme 時間軸 | **只做 TTS，回傳音檔 URL** |
| F5 | 播放 viseme 時間軸 | **uLipSync 即時 MFCC 分析驅動 shape key** |
| 跨模組介面 | 音檔 ＋ viseme JSON | **只剩音檔 URL** |

### 8.3 MVP 精細度：下巴開合模式

考量 AI 生成 mesh 的臉部拓樸限制，MVP **不做完整母音 viseme**：

- uLipSync 以**音量包絡**驅動單一 `JawOpen` shape key（0–1）
- 眨眼透過 `Blink_L/R`，或眼睛材質切換（備援）
- 在實境疊層觀看距離下，與完整 viseme 的體感差異有限

**完整 viseme（A/I/U/E/O/N）** 列為 Roadmap。**N（閉嘴型）為必要項**——中文雙唇音（ㄅㄆㄇ）需嘴唇閉合，未來升級不得省略。

### 8.4 效能

uLipSync 的每幀 MFCC 運算與 3D 渲染、相機串流、GPS 輪詢疊加。**中階 Android 連續 5 分鐘的幀率與發熱須於 Phase 0 實測**，優先度高於 APK 體積。

> **v2.2**：基準機型定義、幀率基線、熱降級階梯與完整迴圈量測規格，見 **§5.8**。本節僅保留「這是最高耗電情境」這個判斷。

---

## 9. 美術方向與 shader 規格

### 9.1 定案方向：半寫實 × 混合靈魂感

| 部位 | 處理 |
|---|---|
| 身體與臉 | Alpha **0.9–0.95**，保留半寫實 PBR 質感 |
| 輪廓 | 微弱 Fresnel rim light，僅邊緣一圈，**不整體發光** |
| 小腿以下 | 垂直漸層 dissolve，淡出成光點 |
| 配色 | 每地標一主題色，僅出現於 rim 與 dissolve |
| 呼吸感 | rim 強度以 sin 波起伏（週期 3–4 秒） |
| 接地 | 橢圓漸層 blob shadow，不透明度約 0.3 |

### 9.2 Unity URP Shader Graph 規格

```
基底：Lit → Surface Type = Transparent，Alpha 0.9–0.95（ScriptableObject 可調）
邊緣：Fresnel Effect → Power 2–4 → × 主題色 → Emission
腳部：Object Space Position.y → Smoothstep → × Alpha
呼吸：Time → Sine → Remap(0.8, 1.0) → × Fresnel 強度
```

**設計優勢**：靈魂感完全在 shader 層，**與資產解耦**。日後導入真 AR 空間錨定時，只需 Alpha 調至 1.0、關閉 rim 與 dissolve 即得完整實體角色，**不需重做資產**。

### 9.3 【待補】

10 個地標主題色定義 ｜ rim 強度與 dissolve 曲線具體參數 ｜ 150m 標記發光美術參數 ｜ 角色概念設計稿與風格一致性規範

---

## 10. API 契約異動

**原則：全部沿用 v1，僅以下兩處異動。**

### 10.1 `POST /api/v1/spirits/{placeId}/dialogue`

Response 200 的 `tts` 物件**移除** `viseme_timeline`：

```jsonc
// v1
{ "reply_text": "...", "tts": { "audio_url": "...", "viseme_timeline": [...] } }
// v2.1
{ "reply_text": "...", "tts": { "audio_url": "..." } }
```

同步影響內部介面 `TTSResult` pydantic model 與 `UnlockStory.tts`。

### 10.2 `GET /api/v1/spirits/{placeId}`

新增靈魂方位設定，供 3DoF 定向使用：

```jsonc
{
  // ...既有欄位
  "orientation": {
    "bearing_deg": 135.0,      // 相對召喚點的方位角（真北 0°，順時針）
    "height_offset_m": 0.0
  }
}
```

```sql
ALTER TABLE spirits ADD COLUMN bearing_deg     DOUBLE PRECISION NOT NULL DEFAULT 0;
ALTER TABLE spirits ADD COLUMN height_offset_m DOUBLE PRECISION NOT NULL DEFAULT 0;
```

### 10.3 v2.1 未異動

`POST /players`、`POST /sense`、`POST /summon`、`GET /daily-event`、`GET /quests/daily`、
`POST /quests/{questId}/complete`、`GET /resonance/{spiritId}`、`GET /players/me/memory-summary`、
`GET /assets/{avatarId}`、`POST /quests/{questId}/landmark-photo`。

Token 設計、配額規則、錯誤碼、fallback 策略全數不變。

**完整契約語意見 §18.9。** 契約的機器可讀真相是 `citysoul-backend/contracts/openapi.json`（由程式碼自動產生，見 §11.2.1）；本文件定義的是行為與語意，兩者不得分歧時以本文件為準、並修正程式碼使其一致。

### 10.4 v2.2 異動一：任務狀態值移除〔⚠️ 破壞性〕

依 §18.4，`quest.status` 的值域移除 `daily_limit_reached`：

```jsonc
// v2.1
"status": "in_progress | completed | daily_limit_reached | already_completed_today"
// v2.2
"status": "in_progress | completed | already_completed_today"
```

影響 `POST /summon` 與 `GET /quests/daily` 兩支回應。

> ⚠️ 這是 §11.2.1「只加不減」原則下的**破壞性變更**，必須在契約凍結（Phase 1）之前落地。落地順序不可顛倒：
> **本文件 → 後端實作 → 重新產生 `contracts/openapi.json` → 客戶端同步並移除該分支處理**。
>
> 客戶端依 §11.2.1 硬規則 5「Enum 採寬鬆解析，未知值 fallback 而非拋錯」，即使順序略有落差也不會崩潰——但仍應照順序做。

### 10.5 v2.2 異動二：引導提問

`POST /api/v1/spirits/{placeId}/dialogue` 的 Response 200 **新增** `suggested_questions`（僅新增欄位，非破壞性）：

```jsonc
{
  "reply_text": "...",
  "tts": { "audio_url": "..." },
  "suggested_questions": ["...", "...", "..."]   // 2–3 則；無可用來源時為 []
}
```

另新增一支端點，供尚未開始對話時取得初始提問：

| Method | Path | 驗證 | 說明 |
|---|---|---|---|
| GET | `/api/v1/spirits/{placeId}/suggested-questions` | Session Token ＋（Sense 或 Encounter Token） | 取得該靈魂今日的引導提問，見 §18.8 |

**Response 200:**

```jsonc
{ "questions": ["...", "...", "..."], "is_fallback": false }
```

**錯誤情境**：`404` 靈魂不存在或 `is_active = false`；其餘一律 `200`，無可用來源時 `questions: []`（元件依 client#17 的 AC 優雅隱藏，不顯示空殼）。

---

## 11. 團隊、專案結構與硬體

### 11.1 分工（改用模組角色名，避免與工作包代號衝突）

| 角色 | 工作包 | 主要環境 |
|---|---|---|
| 🧭 **Lead**（你） | SDD、API 契約、整合、驗收測試、地標勘查主辦 | Unity（唯讀/審查）＋ Docker |
| 🦾 **身體 Body** | **S1–S14** | Unity（`Features/` `Services/`）＋ 後端 S 部分 |
| 🧠 **腦袋 Brain** | **B1–B13** | 純後端，不碰 Unity |
| 🎭 **臉 Face** | **F1–F8** | Blender / Mixamo ＋ Unity（`Characters/`） |

> ⚠️ **負載提醒**：腦袋承擔 13 個工作包為單人最重；身體橫跨客戶端與後端且為關鍵路徑。
> Lead 應預留下海支援餘裕，優先支援腦袋的後端 S 工作包。
> 依「來不及會加人力」原則，優先補位順序建議：① 技術美術（分擔 F4–F8）② 後端（分擔 S2–S5）。

### 11.2 Repo 結構〔本次新增〕

底層已全面更換，**建議新開專案**，不沿用任何 Flutter 時期 repo。

| Repo | 內容 | 關鍵設定 |
|---|---|---|
| `citysoul-backend` | FastAPI、schema、alembic、docker-compose | 一般 Python `.gitignore` |
| `citysoul-client` | Unity 專案 | **Git LFS**（`*.fbx` `*.png` `*.wav` `*.bundle`）＋ Unity `.gitignore` ＋ **Force Text** serialization ＋ **Visible Meta Files** |
| （資產原始檔） | `.blend`、ComfyUI 輸出、4K 貼圖來源 | **不進 Git**，走雲端硬碟；僅交付用 `.fbx` 進 client repo |

Unity 專案設定務必在第一天就開好（Force Text / Visible Meta Files），事後改會產生大量無意義 diff。

#### 11.2.1 API 契約單一真相來源〔本次新增〕

拆兩個 repo 的代價是「後端改了契約、客戶端沒跟上」，以 3 天 Sprint 的節奏這會頻繁發生。處理方式為 **code-first ＋ CI 契約閘門**，不採用純 schema-first（FastAPI 不擅長從 OpenAPI 反向生成 server code，摩擦成本過高）。

**流程**：

```
① 後端     FastAPI 自動產生 /openapi.json
② 後端 CI  與已 commit 的 contracts/openapi.json 比對（oasdiff）
             破壞性變更（刪欄位／改型別／改必填）→ ❌ build 失敗
             僅新增欄位 → ✅ 通過並更新快照
③ 後端     merge 至 main 後，CI 將 openapi.json 推為 Release artifact（帶版本 tag）
④ 客戶端   script 拉取最新 openapi.json
             以 NSwag / openapi-generator 生成 C# DTO
             → Assets/_Project/Services/Generated/  ★不得手改
```

**Unity 端硬規則**：

| # | 規則 | 理由 |
|---|---|---|
| 1 | **只生成 DTO model，不生成 HTTP client** | generator 預設產出的 `HttpClient` 版本不適用 Unity；`ApiClient.cs` 維持手寫（`UnityWebRequest`） |
| 2 | **使用 Newtonsoft.Json**（`com.unity.nuget.newtonsoft-json`），**不得使用 `JsonUtility`** | `JsonUtility` 不支援 Dictionary 與 nullable，無法正確處理 `unlock_story: {...} \| null` 等結構 |
| 3 | `POST /quests/{questId}/landmark-photo` **手寫** | `multipart/form-data`，codegen 產物不可直接使用 |
| 4 | `Generated/` 檔頭標註 `DO NOT EDIT`，CI 檢查是否與最新 schema 一致 | 防止手改後被下次生成覆蓋 |
| 5 | Enum 採寬鬆解析，未知值 fallback 而非拋錯 | 後端新增 enum 值對舊版 App 屬破壞性變更 |

**版本相容原則**：

- **只加不減**：後端可新增欄位，**不得刪除或改名**既有欄位；客戶端忽略未讀欄位
- 破壞性變更必須開 `/api/v2`，舊版保留一段過渡期
- CI 閘門可由特定 PR label（`breaking-change`）跳過，但**必須同時提交 v2 路由**
- 契約凍結時點：Phase 1（見 §13.3）

### 11.3 硬體需求

| | Lead | 身體 | 腦袋 | 臉 |
|---|---|---|---|---|
| Unity ＋ Android Build Support | ✅ | ✅ | ❌ | ✅ |
| Unity Library 快取 | ✅ | ✅ | ❌ | ✅ |
| Android SDK / NDK | CI 代勞 | ✅ | ❌ | CI 代勞 |
| Docker（Postgres/Redis） | ✅ | ✅ | ✅ | ❌ |
| Blender | ❌ | ❌ | ❌ | ✅ |
| ComfyUI ＋ 3D 權重 | ❌ | ❌ | ❌ | ⚠️ 雲端 GPU |
| **建議可用空間** | **60–70 GB** | **80–100 GB** | **30–40 GB** | **60–80 GB** |
| 獨立顯卡 | ❌ | ❌ | ❌ | ❌（雲端 GPU） |

**實機**：至少 2 台 Android，**不同 SoC 廠牌**（如一台 Snapdragon、一台 Dimensity）。Lead 與身體各持一台。

### 11.4 硬體風險緩解措施〔必做，Phase 0 前完成〕

專案曾因「Unity as Library 開發時硬碟空間不足」而推翻 Unity 方案。重新導入前必須落實：

| # | 措施 | 效益 |
|---|---|---|
| ① | **Android build 交給 GitHub Actions ＋ GameCI** | Lead 不需維護完整 Unity＋NDK 環境，省 40–60 GB |
| ② | **ComfyUI 走雲端 GPU（RunPod / Vast.ai 按小時計費）** | 臉省 40–80 GB 與一張顯卡；10 隻為一次性生產 |
| ③ | **不採用 Unity as a Library**，全 Unity 單一 App | 避免重蹈 v1 失敗路徑 |

**前置條件**：上述未落實前，不啟動 Phase 1。

---

## 12. 風險排序（v2.1）

| 等級 | 風險 | 變化 | 緩解 |
|---|---|---|---|
| 🔴 | **AI 對話成本與延遲** | 不變 | 配額分級、B12 固定招呼層、單一快速模型 |
| 🔴 | **宗教場所內容治理**（龍山寺） | ★**新增** | 見 §12.2 |
| 🔴 | **AI 生成 mesh 可用性**（綁骨失敗、清理工時不可預測） | ★新增 | Phase 0 POC 早期驗證；CC4 備援 |
| 🟠 | **臉的 Unity 能力缺口**（F4–F8） | ★新增 | §6.3【待確認】；必要時補技術美術 |
| 🟠 | **身體為關鍵路徑單點** | ★新增 | Lead 預留支援；Phase 0 POC 避免並行壓在身體 |
| 🟠 | **中階 Android 效能與發熱** | ★新增 | Phase 0 實測；面數與貼圖預算控管 |
| 🟠 | **羅盤在都市磁干擾下的方位穩定性**（萬華街廓密集，較天文館嚴峻） | ★新增 | 校正 UI；方位容忍設計 |
| 🟠 | **GPS 都市峽谷效應**（萬華） | ★升級 | 實地量測後調整 `summon_radius_m` |
| 🟠 | 10 隻角色風格一致性 | ★升級（3D 後更難） | 固定 prompt/LoRA/參考構圖；或改 CC4 |
| 🟡 | ~~AR / Unity 整合~~ | ★降級（不用 AR SDK） | — |
| 🟡 | ~~Geospatial API 成本~~ | ★移除 | — |
| 🟡 | ~~ARCore 認證裝置覆蓋率~~ | ★移除 | — |
| 🟡 | Unity 繁中 IME 輸入 | ★新增 | Phase 0 驗證 |
| 🟡 | 當日事件、任務邏輯、GPS spoofing | 不變 | 沿用 v1 |
| 🟢 | 隱私 | 略升（照片觸及後端） | 即用即丟約束 |

**耗電與發熱**：因不採用真 AR，較 v1 的 AR 方案大幅改善，v1 的 AC11 省電設計得以保留。

### 12.2 ⚠️ 龍山寺切片的內容治理專節〔本次新增〕

CONTEXT.md 原選定天文館作為垂直切片，理由是中性、科普、無信仰爭議。改為龍山寺後，以下項目**必須在 Phase 3（B4/B5 安全邊界）之前處理**：

| 項目 | 說明 | 負責 |
|---|---|---|
| **寺方接洽** | 事前溝通遊戲性質與現場行為規範，取得態度確認 | Lead |
| **人格卡禁忌主題擴充** | `taboo_topics` 需明確涵蓋：教義解釋、神祇位階、靈驗與否、占卜結果、宗教比較 | Lead ＋ 敘事 |
| **`not_this_character` 明確化** | 「不是廟方人員、不是任何神祇、不是宗教解說員」 | Lead ＋ 敘事 |
| **B4 輸入端過濾強化** | 玩家詢問求籤、運勢、宗教爭議時的攔截與轉向策略 | 腦袋 |
| **B5 邊界規則注入** | 明確禁止對信仰內容作教義性陳述或裁決（對應 §3 新增 Avoid 條目） | 腦袋 |
| **現場行為指引** | App 內提示玩家在廟埕的適當舉止 | 身體 |
| **人工預寫 fallback 文案** | 觸及禁忌時的角色口吻轉向台詞 | Lead ＋ 敘事 |

> **建議【待確認】**：若上述內容工作在 Phase 3 前無法完成，建議垂直切片先回到天文館，龍山寺留在首發十個地標內、於封測後上線。
> 技術驗證（GPS、羅盤、實地勘查）仍可先在龍山寺進行，不受影響。

---

## 13. 排程（7 階段垂直切片，3 天／Sprint）

### 13.1 排程原則

**改為垂直切片**：v1 的橫向堆疊要到後期才有端到端可玩版本，代表要盲飛多個階段才知道核心迴圈成不成立。改為 Phase 2 就打通最細的一條端到端（Walking Skeleton），讓延遲、效能、現場體感三大風險提早曝光。

**代價**：Phase 1–2 會有部分臨時程式碼（假資料、寫死地標、灰模角色）之後要重寫。此代價已接受。

**三條軸線**：腦袋（後端）與臉（資產）可完全平行；**身體是唯一匯流點且為單點**，排程圍繞「不讓身體空等」設計。

**節奏**：1 Sprint = 3 個工作天。7 個 Phase，共約 **27 個 Sprint ≈ 81 個工作天 ≈ 16 週**。

### 13.2 Phase 0：技術驗證（2 Sprint／6 天）

| POC | 負責 | 通過門檻 |
|---|---|---|
| **P0-A 資產可行性** | 臉 | 見下方檢查表 |
| **P0-B 3DoF 方位穩定性** | 身體 | **在龍山寺現場**原地旋轉一圈，角色方位偏移 < 15°；含校正流程 |
| **P0-C Unity 繁中 IME** | 身體 | Android 實機中文輸入法不跳版、不吃字、不遮擋輸入框 |
| **P0-D 對嘴＋效能** | 身體 ＋ 臉 | 播 TTS 音檔 → 身體動作 ＋ 下巴開合；中階實機連續 5 分鐘 ≥ 30fps |
| **P0-E 後端環境** | 腦袋 | docker-compose 起 Postgres＋Redis，`/players` `/sense` `/summon` 通 |

**P0-A 通過門檻檢查表**（決定 ComfyUI 是否續用）：

```
□ Mixamo 自動綁骨成功，四肢權重無明顯崩壞
□ 手肘、膝蓋彎曲時無麻花狀擠壓
□ 眼睛可分離為獨立材質
□ JawOpen shape key 可製作且不破面
□ 減面至 20k tris 後外觀無明顯劣化
□ 單隻清理工時 < 4 小時（可預測性驗證）
```

**未通過 → 切換 CC4 備援路線**，並回頭修訂 §7 與 F1/F2/F3 定義。

> ⚠️ P0-B/C/D 皆由身體負責，為 Phase 0 關鍵路徑。建議順序：**B → C → D**，避免並行。
> ⚠️ **Phase 0 結束後，Phase 1–7 需複審**——P0-A 結果會改變整條資產產線定義。

### 13.3 Phase 1：骨架（上）（3 Sprint／9 天）

| 角色 | 內容 |
|---|---|
| 🧠 腦袋 | B3 人格卡載入、B7 短期記憶、S6 身分後端、S2 在場驗證 |
| 🦾 身體 | Bootstrap 場景、TokenManager、ApiClient、S1 GPS、EncounterScene 雛形 |
| 🎭 臉 | **灰模交付**（Mixamo 免費角色 ＋ JawOpen，符合 §7.3 規格但美術不論）、F1 概念設計啟動 |
| 🧭 Lead | Repo 建置、CI 建置、API 契約凍結、**龍山寺實地勘查（全員同行）** |

> **灰模是關鍵**：讓身體與臉能在 Phase 1 就開始開發 F4/F5/F8，不必等正式角色。正式角色進來時直接替換。

### 13.4 Phase 2：骨架（下）★端到端打通（4 Sprint／12 天）

**目標**：玩家走到龍山寺 → 看到標記 → 點召喚 → 進疊層畫面 → 對一句話 → 聽到語音 → 角色嘴巴動。功能全部陽春版（無任務、無共鳴值、無長期記憶）。

| 角色 | 內容 |
|---|---|
| 🧠 腦袋 | B1 Gemini 串接、B10 TTS、dialogue API 可用版 |
| 🦾 身體 | **S7 tile renderer（提前）**、地圖標記、對話 UI prefab、**S14 3DoF 定向** |
| 🎭 臉 | F5 uLipSync、F8 Animator、F2 清理管線（正式角色 #1 起手） |
| 🧭 Lead | 端到端整合測試、**首次現場實測**、**建立 §11.2.1 API 契約閘門與 codegen 流程** |

### 13.5 Phase 3：判定邏輯與安全層（4 Sprint／12 天）

| 角色 | 內容 |
|---|---|
| 🧠 腦袋 | B4 安全邊界、B2 Prompt 組裝、B12 快速問候、S3 防作弊、S4 任務表、S5 共鳴值表、**B14 引導提問生成 ★v2.2 提前** |
| 🦾 身體 | 召喚流程完整化、任務 UI 雛形、S12 拍照、**S9 引導提問元件 ★v2.2 自 Phase 5 提前** |
| 🎭 臉 | F3 Mixamo 綁骨 ＋ shape key（正式角色 #1） |
| 🧭 Lead | **§12.2 龍山寺內容治理項目必須在此階段完成**；**§6.5 人格卡注入硬規則驗收** |

> **v2.2 為何把 S9／B14 從 Phase 5 提前到 Phase 3**：引導提問已定位為**主體驗**（自由文字輸入是進階功能）。主體驗留到 Phase 5 才做，等於 Phase 3–4 的所有對話測試都在測一條不是主要路徑的介面，測到的體感不是玩家會遇到的體感。

### 13.6 Phase 4：對話品質與內容（4 Sprint／12 天）

| 角色 | 內容 |
|---|---|
| 🧠 腦袋 | B5 史實邊界、B6 長期記憶、B9 當日情境、B11 解鎖敘事、B13 地標辨識 |
| 🦾 身體 | 感應/召喚雙模式對話串接、配額與 fallback UI |
| 🎭 臉 | **F4 靈魂感 shader**、疊層視覺完成度、正式角色 #1 完成 |
| 🧭 Lead | 人格卡內容驗收、fallback 文案 |

### 13.7 Phase 5：UI 完整化（4 Sprint／12 天）

| 角色 | 內容 |
|---|---|
| 🧠 腦袋 | S10 當日情境排程、S11 訊號推播 |
| 🦾 身體 | S8 任務/Profile 場景、S13 分享、**ThermalGovernor（§5.8.3）** |
| 🎭 臉 | F6 Addressables 打包、F7 下載/快取 |
| 🧭 Lead | 對外 API 全串通驗收、**§20.1 節慶日曆與輪播池內容備齊** |

> S9 引導提問元件已於 Phase 3 完成，不在本階段。

### 13.8 Phase 6：封測準備（3 Sprint／9 天）

| 角色 | 內容 |
|---|---|
| 全員 | 端到端 QA、效能調校、實機多機種測試、**龍山寺現場多次實測** |
| 🧠 腦袋 | B8 記憶摘要 batch |
| 🧭 Lead | 封測招募、現場行為指引、事故應變流程 |

### 13.9 Phase 7：封測執行與收尾（3 Sprint／9 天）

封測執行、數據回收、`usage_tiers` 調校、Prompt 參數依實測調整。

> **角色 #2–#10 不在此排程內。** 依 CONTEXT.md 垂直切片原則，**通過封測驗證後才擴展**到首發十個地標。此為獨立的內容擴充階段，不佔用上述 27 個 Sprint。

---

## 14. 驗收標準異動

> **v2.2**：完整清單**已收進本文件 §19**，不再指向 `city_soul_AR_acceptance_criteria.md`（該文件封存，見 §17.2）。本節保留 v2.1 的異動說明作為歷史脈絡；逐條驗收請用 §19。

### 14.1 沿用不變

AC1（在場驗證與召喚）、AC2（感應驗證）、AC3（mock location 觀察期）、~~AC4（微任務生命週期）~~〔**v2.2 改寫，見 §19.4**〕、AC5（共鳴值）、AC6（Token 邊界）、AC7（混合式聊天）、AC8（配額）、AC9（Fallback）、AC10（地圖標記狀態機）、AC13（開發環境）、AC14（地標視覺辨識）

### 14.2 需改寫

| AC | 異動 |
|---|---|
| **AC11** 距離偵測省電方案 | 平台改為 Unity `Input.location`；輪詢間隔**由 30 秒改為 10 秒**（見 §5.5）；「不申請背景定位權限」不變 |
| **AC12** 相機疊圖畫面 | 全面改寫為「實境疊層畫面」，見下 |

### 14.3 AC12 改寫版（實境疊層畫面）

- **AC12.1** Given 剛進入實境疊層場景 When 檢視對話窗 Then 預設為**展開**
- **AC12.2** Given 點擊收合 When 對話窗收合 Then 仍可看到相機畫面與 3D 角色，且可再次展開
- **AC12.3** Given 在 50m 內越靠近召喚點 When 持續移動 Then 角色縮放**連續**漸變，非跳躍式
- **AC12.4** Given 原地水平旋轉手機 When 靈魂方位角超出視野 Then 角色停止繪製，螢幕邊緣顯示方向指示器
- **AC12.5** Given 轉回靈魂方位 When 方位角進入視野 Then 角色重新出現於對應螢幕水平位置
- **AC12.6** Given `headingAccuracy` 劣化超過門檻 When 進入或停留於實境疊層場景 Then 顯示羅盤校正提示

### 14.4 新增 AC

| AC | 內容 |
|---|---|
| **AC15.1** | 資產交付時 shape key 必須包含 `JawOpen`，大小寫完全相符 |
| **AC15.2** | 角色面數 ≤ 20,000 tris |
| **AC15.3** | Addressables bundle < 10 MB |
| **AC15.4** | App 首次啟動**未預載**全部角色；僅於進入某地標 150m 範圍時下載該角色 |
| **AC15.5** | 正式角色替換灰模時，`Features/` 與 `Services/` 下**不需修改任何程式碼**（驗證 §7.3.1 規格落實） |
| **AC15.6** | 封測與上線 build 中**不含**任何 Mixamo 灰模資產 |
| **AC16.1** | 播放 TTS 音檔時 `JawOpen` 隨音量包絡連續變化 |
| **AC16.2** | **§5.8.1 定義的地板機**實機連續運行實境疊層 5 分鐘，**全程**幀率 ≥ 30fps（非平均），須附 fps 曲線、最低值與掉幀分佈；無明顯發熱降頻 〔**v2.2 改寫**：原為「平均 ≥ 30fps」，平均會掩蓋週期性卡頓，而卡頓正是玩家感覺得到的〕 |
| **AC17.1** | Android APK 僅含 ARM64，不含 ARMv7 |
| **AC17.2** | APK 下載大小 < 100 MB |
| **AC17.3** | App 權限清單**不含**背景定位權限，**不含** ARCore 相關宣告 |
| **AC18.1** | token 使用 Android Keystore，**不得**使用 `PlayerPrefs` |
| **AC19.1** | 玩家詢問求籤、運勢、教義解釋等禁忌主題時，回應不含教義性陳述或裁決，且以角色口吻自然轉向 |
| **AC19.2** | 人格卡 `not_this_character` 明確排除「廟方人員」「任何神祇」「宗教解說員」 |

### 14.5 v2.2 新增 AC

| AC | 內容 |
|---|---|
| **AC20.1** | B9 當日情境、B11 解鎖敘事、任務完成包裝台詞三者的 prompt 中，**均可驗證地包含**該靈魂人格卡的 `taboos` 與 `not_this_character`（§6.5） |
| **AC20.2** | 四條生成路徑的人格卡區段由**同一個函式**組裝（grep 只有一份實作），不各自複製 |
| **AC20.3** | 對龍山寺靈魂在解鎖敘事路徑上詢問／觸發占卜、姻緣、靈驗主題時，產出不含教義性陳述或應許（AC19.1 的涵蓋範圍自 B2 擴及全部生成路徑） |
| **AC21.1** | 相遇憑證過期且任務未完成時，玩家再次召喚 Then `attempts_today` **不增加**，任務可直接重新挑戰（§18.4） |
| **AC21.2** | 任一 `attempts_today` 值皆**不導致**召喚被拒或任務被鎖；API 回應中**不存在** `daily_limit_reached` 這個值 |
| **AC21.3** | 同一任務重複完成時，共鳴值**不重複入帳**（由 `resonance_events` UNIQUE 約束保證，非由嘗試上限保證） |
| **AC22.1** | `GET /suggested-questions` 回傳 2–3 則提問；同一靈魂**跨日再查時內容不同**（驗證隨當日情境變動，非靜態清單） |
| **AC22.2** | B14 生成失敗或無合格輸入時回傳人工預寫提問，**HTTP 200**，不回錯誤碼 |
| **AC22.3** | 點擊引導提問後，其進入對話流程的路徑與玩家自行輸入**完全相同**（同樣經過配額、B4 安全檢查、B12 比對） |
| **AC23.1** | Bootstrap 完成後 `Application.targetFrameRate == 30`、`QualitySettings.vSyncCount == 0` |
| **AC23.2** | 熱狀態達 MODERATE 時相機解析度與 uLipSync 分析頻率確實下降；達 SEVERE 時相機串流停止且畫面仍有靜態背景（§5.8.3） |
| **AC23.3** | 離開 EncounterScene 後 `Screen.sleepTimeout` 還原為 `SystemSetting` |
| **AC23.4** | 地板機一次完整迴圈（目標 12 分鐘）耗電 ≤ 8%，全程未進入 SEVERE（§5.8.5） |
| **AC24.1** | 當日情境的合格輸入具備生產者：節慶日曆命中日與輪播池命中日，`is_fallback` 為 `false`（§20.1） |
| **AC24.2** | 當日情境的人工預寫 fallback 台詞**按靈魂各自定義**，不共用單一全域字串（§20.1.3） |

---

## 15. Roadmap（MVP 之後）

### 15.1 本次改版新增

1. **真 AR 空間錨定**（ARCore Geospatial API / 平面偵測 ＋ 遮擋）
   - 前置：重估 Geospatial 計費、ARCore 認證裝置覆蓋率、耗電模型
   - 資產相容：靈魂感 shader 已設計為可調回實體感，**不需重做資產**
2. **完整 viseme 口型（A/I/U/E/O/N）**
   - 前置：臉部重拓樸或改用 CC4；**N（閉嘴型）不得省略**
3. **服裝/道具切換**（撐傘、黃昏換裝）
   - 3D 化後成本高於 2.5D 圖層切換；**須在角色設計階段預留擴充點**
4. **iOS 版本**
5. **角色 #2–#10 擴充**（封測通過後）

### 15.2 五項技術壁壘（沿用 v1）

城市人格引擎 ｜ 多 Agent 協作 ｜ 世界記憶（持久化，⚠️需重新界定邊界以免與「不含玩家輸入」原則衝突）｜ AI 導演 ｜ AR 情境感知

> 「多 Agent」「AI 導演」會大幅增加 LLM 呼叫次數，排入時需一併重估成本模型。

---

## 16. 尚未解決的缺口

| 缺口 | 影響 | 時機 |
|---|---|---|
| ★ 臉的 Unity 能力是否足以持有 F4–F8 | 分工與排程 | **Phase 0 前確認** |
| ★ ComfyUI 產線可行性未驗證 | 資產產線全局 | **Phase 0 P0-A** |
| ★ 龍山寺內容治理七項（§12.2） | 內容安全 | **Phase 3 前** |
| ★ 寺方接洽與態度確認 | 專案可行性 | Phase 1 |
| ★ 龍山寺 `bearing_deg` 與 GPS 半徑實測 | S14／S2 | Phase 1 實地勘查 |
| ~~★ 中階 Android 基準機型定義~~ | ~~AC16.2 驗收~~ | ✅ **v2.2 已定義，見 §5.8.1**（實體地板機仍待採購） |
| ★ Unity tile renderer 圖磚載入/快取策略 | S7 | Phase 2 |
| 主題色與風格一致性規範 | F1 | Phase 1 |
| `canned_greetings` 觸發語與預寫回覆 | 人格卡 | Phase 3 前 |
| 150m 發光、疊層漸顯美術參數 | F 模組 | Phase 4 前 |
| 429 配額用完的角色口吻 fallback 文案 | 腦袋內容 | Phase 4 前 |
| Prompt Top-K / 記憶輪數 | 腦袋調校 | 先用 K=3、短期記憶 6 輪，封測後調整 |
| `usage_tiers` 分級策略 | 產品決策 | Phase 7 |
| **★ 全域（非單一玩家）LLM 用量上限** | 公開後的成本風險 | **v2.2 新增**。`usage_tiers` 是 per-player 防濫用，攔不住「1000 人同時觸頂」。封測期人數受控故不阻擋切片，但**公開前必須有**，見 §18.6.4 |
| **★ 首發十靈魂的內容量能** | MVP（非切片） | **v2.2 新增**。§20.1 的輪播池在單一靈魂是兩三天的工作，十靈魂是 300–600 條 ＋ 逐字審核。切片通過後才展開估算 |
| **★ 故事線 beat 推進的實作** | 跨地標留存機制 | **v2.2 新增**。§20.4 已定規格；`story_arcs`／`story_beats`／`players_story_progress` 表已建但無讀寫邏輯。跨三地標故不屬切片，排在切片之後 |
| **★ 寺方對「舉手機錄影／外放語音」的態度** | 互動設計可行性 | **v2.2 新增**。§12.2「寺方接洽」原本只涵蓋內容治理，未涵蓋**玩家在現場的身體行為**。實地勘查（client#25／#29）須帶回第一手判斷 |

---

## 17. 附錄

### 17.1 ADR

- `0001-gcp-vertex-ai-for-mvp-dialogue.md`：MVP 對話統一使用 Vertex AI 單一快速模型；失敗回退人工台詞〔既有〕
- `0002-unity-3dof-overlay-over-real-ar.md`〔待撰寫〕：客戶端改用 Unity；不採用 Godot 與真 AR
- `0003-ulipsync-over-tts-viseme-timeline.md`〔待撰寫〕：對嘴改為客戶端即時音訊分析
- `0004-longshan-temple-vertical-slice.md`〔待撰寫〕：垂直切片地標由天文館改為龍山寺及其內容治理配套

### 17.2 舊文件處置〔v2.2 全面更新〕

**原則**：v2.1 把七份文件標為「需修訂／小修」，但**沒有任何一份真的被修訂**——它們同時是權威、又同時已知有錯，而且其中兩份還住在被 `.gitignore` 排除的目錄裡。v2.2 不再走「修訂」這條路：**該內容進 SDD，文件本身封存**。一份需要讀者自行套用勘誤表才能用的權威，不是權威。

| 文件 | 位置 | v2.2 處置 | 說明 |
|---|---|---|---|
| **`SDD_v2.2_Unity_3D.md`** | `citysoul-doc/` | ✅ **唯一權威** | 本文件 |
| `SDD_v2.1_Unity_3D.md` | `citysoul-doc/` | **廢止** | 由本文件取代。保留檔案供追溯，開頭需加廢止標記 |
| `citysoul_data_schema.md` | `citysoul-doc/` | ✅ **現行權威（schema 專用）** | PostgreSQL 完整 schema 與後端邏輯參考。SDD 不重複 schema 定義，只在 §10.2 記錄異動 |
| `contracts/openapi.json` | `citysoul-backend/` | ✅ **現行權威（機器可讀契約）** | 由程式碼自動產生（§11.2.1）。語意與行為以 SDD §18.9 為準 |
| `CONTEXT.md` | `citysoul-backend/` | **需修訂**（v2.1 起未執行） | ①「2.5D 靈魂角色」改寫為 3D；②「完整 3D 角色」自 Avoid 移除；③ 垂直切片由天文館改龍山寺；④ 新增宗教內容 Avoid 條目；⑤ **v2.2 新增**：「可驗證微任務」條目移除失敗重試上限的隱含前提 |
| `city_soul_AR_業務規則與API契約.md` | `citysoul-backend/docs/`（本機） | **封存** | 內容已吸收進 **§18**。⚠️ 封存前必須先進版控（見下方遷移註記） |
| `city_soul_AR_acceptance_criteria.md` | `citysoul-backend/docs/`（本機） | **封存** | 內容已吸收進 **§19**。同上 |
| `city_soul_AR_frontend_interaction.md` | `citysoul-backend/docs/`（本機） | **封存** | 感應／召喚雙層模型與 Sense Token 已吸收進 §18.1／§18.2；第 1 節 Google Maps SDK 與第 6 節 Geofencing 已被 v2.1 推翻 |
| `city_soul_AR_landmark_recognition.md` | `citysoul-backend/docs/`（本機） | **保留為現行權威** | B13 技術方案，SDD 未涵蓋其細節。⚠️ 需進版控 |
| `city_soul_AR_document_index.md` | `citysoul-backend/docs/`（本機） | **廢止** | 它導覽的那批文件已封存；導覽本身失去對象。接手者改讀本文件 §1 |
| `SDD.md`(v1) | — | **廢止** | v2.1 已廢止 |
| `city_soul_AR_flutter_architecture.md` | `citysoul-backend/docs/`（本機） | **廢止** | 整份為 Flutter 專案架構 |
| `city_soul_AR_project_mainpoint.md` | `citysoul-backend/docs/`（本機） | **封存** | 歷史快照，僅供決策脈絡 |
| `city_soul_AR_project_dev.md` | `citysoul-backend/docs/`（本機） | **封存** | §2.1 第三方工具評估結論已被推翻 |
| `city_soul_AR_WBS_API.md` | `citysoul-backend/docs/`（本機） | **封存** | 模組職責已吸收進 §6；F 模組定義與 viseme 決策 2 均已被推翻 |
| `city_soul_AR_schema_api_sprint.md` | `citysoul-backend/docs/`（本機） | **封存** | schema 以 `citysoul_data_schema.md` 為準；Sprint 表以 §13 為準 |
| `citysoul-sprint-map.html` | `citysoul-backend/docs/` | **廢止** | Sprint 結構已全面改變 |

#### 17.2.1 ⚠️ 遷移前置：兩份文件必須先進版控

`city_soul_AR_業務規則與API契約.md`、`city_soul_AR_acceptance_criteria.md`、`city_soul_AR_landmark_recognition.md` 目前位於 `citysoul-backend/docs/`，而該 repo 的 `.gitignore` 以 `docs/*` 排除整個目錄（僅 `docs/agents/` 與 `docs/content-governance/` 例外）。

也就是說：**在 v2.2 落地之前，本專案一半的業務規則權威只存在單一台機器上，無版本、無備份、無法被任何協作者讀取。**

`.gitignore` 為內容治理文件開例外時寫的理由適用於此：「一份簽核記錄如果只活在某個人的硬碟上，那就不是記錄。」

**執行順序**：① 三份文件複製進 `citysoul-doc/archive/`（landmark_recognition 進 `citysoul-doc/` 根目錄，因其仍為現行權威）→ ② commit → ③ 才開始依本表封存。**順序顛倒會造成不可回復的資料遺失。**

### 17.3 給接手開發者 / Claude Code 的起手建議

1. **先讀 §1 推翻對照表**。專案歷史上有三次技術棧變更，舊文件仍在，容易誤讀
2. 落實 §11.4 三項硬體緩解措施，再啟動 Phase 1
3. 依 §11.2 新開兩個 repo；Unity 專案第一天就設好 Force Text / Visible Meta Files
4. 執行 Phase 0 五個 POC；**P0-A 未通過即切換 CC4 備援**，不要硬做
5. 後端用本地 docker-compose 起環境，不急著佈建 GCP
6. 每完成一個功能，對照 **§19** 的 AC 編號逐條驗收（v2.2 起 AC 在本文件內，不在外部檔案）
7. 業務規則查 **§18**；內容生產管線查 **§20**；schema 查 `citysoul_data_schema.md`
8. 文件矛盾時一律以本文件為準

> **給 Claude Code 的額外提醒**：`citysoul-backend/docs/` 底下的 `city_soul_AR_*.md` 自 v2.2 起**全部封存或廢止**（§17.2）。它們讀起來像規格、寫得也很詳細，但描述的是 Flutter 客戶端、viseme 時間軸、任務失敗上限這些已被推翻的設計。照它們實作是這個專案最容易浪費一整個 Sprint 的方式。

---

## 18. 業務規則（v2.2 自 v1 吸收，含本次修正）

> **本節是業務規則的唯一權威。** 內容來自 `city_soul_AR_業務規則與API契約.md` 與
> `city_soul_AR_frontend_interaction.md`（兩者自本版起封存，見 §17.2），並套用 §1.1 的 v2.2 修正。
>
> 標示 **〔v2.2 修正〕** 者為本版變更，其餘為 v1 定案內容的原地收錄。

### 18.1 距離模型：150m 感應 / 50m 召喚

兩段式距離互動。「聊天」與「正式召喚」拆開：

| 範圍 | 名稱 | 可以做什麼 |
|---|---|---|
| ≤ 150m | **感應範圍**（`spirits.sense_radius_m`，預設 150） | 與靈魂聊天（B12 命中則零成本回覆，未命中才呼叫 LLM） |
| ≤ 50m | **召喚範圍**（`spirits.summon_radius_meters`，預設 50） | 正式召喚 → 進入實境疊層 → 任務、共鳴值 |
| > 150m | — | 地圖標記維持預設樣式；聊天輸入不可用；顯示模糊提示「再靠近一點」 |

**判定邏輯**：

```
distance = haversine(玩家GPS, spirit.latitude, spirit.longitude)
在場成立 if distance <= spirit.summon_radius_meters
```

**不做動態誤差放寬。** `summon_radius_meters` 這個值本身就是現場勘查時把 GPS 常見誤差一併考慮進去定出來的「有效半徑」，不是地標的實際佔地範圍。seed 資料與後台介面須註記此事，避免被誤解為精確地理邊界。

**不顯示精確距離數字。** 原因有二：持續輪詢算距離耗電；以及顯示「還差 23m」會把玩家的注意力從街景拉回螢幕。

### 18.2 三種 Token

| Token | 效期 | 核發時機 | 證明什麼 |
|---|---|---|---|
| **Session** | 90 天（可續） | `POST /players` | 我是哪個玩家 |
| **Sense** | 30 分鐘 | `POST /sense`（150m 內） | 我在感應範圍，可以聊天 |
| **Encounter** | 15 分鐘（900 秒） | `POST /summon`（50m 內在場驗證通過） | 我剛通過在場驗證，可以做任務 |

**技術規格**：JWT 自簽、HS256、不查資料庫。取捨是**無法主動撤銷**——需要立即失效時，只能等自然過期或改用 opaque token + Redis。MVP 接受此取捨。

金鑰存 GCP Secret Manager，Cloud Run 以環境變數掛載，不進版控。Session 與 Encounter **建議使用不同金鑰**，讓其中一把外洩不波及另一種。

**Claims**（不含 `jti`，因為不做撤銷名單，加了也沒地方用）：

```json
{ "sub": "player_id (UUID)", "spirit_id": "longshan_temple", "purpose": "encounter|sense", "iat": 0, "exp": 0 }
```

⚠️ **三種 token 一律不含 GPS 座標**，對應「相遇憑證不包含或保存原始 GPS」的隱私原則。

**能力邊界**：

| 能力 | Session | Sense | Encounter |
|---|---|---|---|
| 玩家層級查詢（quests/daily、resonance、memory-summary） | ✅ | — | — |
| 對話（含 LLM） | 必要 | ✅ | ✅ |
| `quests/{id}/complete` | 必要 | ❌ | ✅ |
| 進入實境疊層場景 | — | ❌ | ✅ |

三種 token 在後端**必須用不同的驗證中介層**，不可共用邏輯。共用會導致兩種災難：拿 15 分鐘的 Encounter 當 session 用（玩家每 15 分鐘被登出）、或拿 90 天的 Session 當 Encounter 用（在場驗證形同虛設，可以在家假裝在召喚點）。

**Sense Token 的靜默續期**：前端自行追蹤 `exp`，剩 1–2 分鐘且仍在 150m 內時主動重呼 `/sense`（沿用最後一次已知在範圍內的座標），**不等 401 才被動處理**。玩家全程無感，不跳提示、不中斷對話。

**對話延續天然成立**：短期記憶（B7）的 Redis key 是 `session:{player_id}:{spirit_id}`，與持有哪張 token 無關。玩家從 150m 聊到走進 50m 正式召喚，對話自動延續，不需額外設計。

### 18.3 防作弊（觀察期）

判定為 mock location 時**只記錄，不阻擋**這次召喚。log 至少含 `player_id`、`spirit_id`、偵測時間、偵測依據（mock location flag、速度異常數值）。

不需要為此新增資料表——應用程式日誌 ＋ Cloud Logging 足夠。等累積數據後再決定是否轉為真的攔截。

### 18.4 可驗證微任務生命週期〔v2.2 修正〕

#### 18.4.1 狀態機

```
不存在 → in_progress → completed
              ↑
              └── 憑證過期且未完成 → 清除當次嘗試，重新召喚即可再挑戰
```

#### 18.4.2 v2.2 修正內容

v1 的設計是：憑證過期 = 一次**失敗嘗試**（`attempts_today += 1`），當天累積 3 次即鎖定至隔天。

**本版移除整個失敗計數與上限機制**：

| 項目 | v1 | v2.2 |
|---|---|---|
| 憑證過期且任務未完成 | `attempts_today += 1` | 僅清除 `current_token_issued_at`，**計數不變** |
| `attempts_today >= 3` | 當天鎖定，拒絕發放任務嘗試 | **無此判定** |
| `quest.status` 值域 | 含 `daily_limit_reached` | **移除該值**（見 §10.4） |
| `attempts_today` / `attempts_date` 欄位 | 參與判定 | **保留為純觀測欄位**，不參與任何判定 |

**理由**：憑證過期的成因絕大多數不是玩家放棄任務，而是手機的日常中斷——來電、訊息、被人搭話、日曬太強先進室內、電話講超過 15 分鐘。把這些記成「失敗」，然後在第三次時鎖住玩家，是在懲罰一件玩家沒有做錯的事。

**移除上限不會開出刷分漏洞**：共鳴值的去重由 `resonance_events` 的 `UNIQUE (player_id, source_type, source_id)` 約束保證，且完成已完成的任務直接回傳現況。上限本來就沒有在防刷——它防的是一個不存在的問題。

**保留欄位而非刪除**的理由：「玩家平均試幾次才完成任務」是有價值的產品數據。欄位留著、判定拿掉，兩件事不衝突。

#### 18.4.3 被動判定原則（不變）

**沒有「回報失敗」API。** 玩家拿了憑證卻直接關掉 App，後端當下不會知道；要等他**下一次召喚同一個靈魂**時，才回頭檢查上一張憑證是否已過期而任務仍在 `in_progress`。

這符合「以後端確定性規則驗證完成與否」——不依賴前端誠實。也代表**不需要任何背景 job** 去掃描：沒有人召喚的時候，那個玩家的狀態是什麼就是什麼，沒有任何人會觀察到差別。

**每日重置同理被動**：`attempts_date` 存 Asia/Taipei 日期，查詢當下比對「今天」是否相同，不同就重置。不需要排程在午夜清表。

#### 18.4.4 任務發放頻率

每個玩家對每個靈魂，**可驗證微任務每日一次**。召喚本身**無冷卻**，可重複觸發；對話互動不受限。

任務目錄與輪播規則見 **§20.3**。

### 18.5 共鳴值

| 來源 | 加值 | 去重 |
|---|---|---|
| 相遇收藏（每個靈魂首次） | +10 | `encounter_collections` 的 `UNIQUE (player_id, place_id)` |
| 可驗證微任務完成 | **+20**（MVP 固定值，不依難度浮動） | `resonance_events` 的 `UNIQUE (player_id, source_type, source_id)` |

**門檻**：所有靈魂共用固定的 **10 / 40 / 100**，解鎖三個階段。共鳴值是**每個靈魂各自累積**的關係進度，不是全域經驗值，**且不決定任務是否完成**。

**入帳時機**：任務完成當下即時入帳並檢查門檻，是完成端點的內部動作，不是前端另外呼叫的步驟。

**呼叫順序是硬規則**（§6.4）：身體必須**先寫完自己的 `resonance` 表、再呼叫腦袋 B11**。顛倒的話腦袋拿到的 `stage` 會與資料庫不一致，玩家會看到一段講述他還沒達到的關係階段的故事。

**一次跨多個門檻要生成多段**：新解鎖階段是 list。每個 stage 各生成一段——只取最後一個會讓中間那段**靜默消失**，沒有任何錯誤訊息提醒。

### 18.6 配額（Usage Tier）

#### 18.6.1 定位

**現階段的定位是防止單一玩家濫用 AI token**，不是商業分級。

#### 18.6.2 資源類型

上限住在 Postgres（`usage_tiers` / `usage_tier_limits`），**計數器走 Redis**（key 含 Asia/Taipei 日期）。這個分工的意義：上限是需要 review 的設定，用量是每天丟掉的高頻計數。

| `resource_type` | 意義 | 週期 | 封測初始值 |
|---|---|---|---|
| `dialogue_calls_daily` | 每日對話呼叫次數 | 每日（Asia/Taipei 午夜） | 50 |
| `daily_tokens` | 每日 token 用量（input+output） | 每日 | 150,000 |
| `prompt_max_chars` | 單次玩家輸入字數上限 | 每次請求（靜態檢查） | 500 |
| `api_rate_per_minute` | 每分鐘 API 呼叫上限 | 每分鐘滑動窗口 | 6 |
| `landmark_recognition_daily` | 每日 B13 辨識呼叫 | 每日 | 依 B13 方案 |

> ⚠️ **以資料庫為準**：`usage_tier_limits.resource_type` 的實際值是上表這五個。程式碼中不得有第二套名稱——那樣的話改資料不會生效，而這張表的整個設計目的就是「改資料不用重新部署」。

**正規化而非欄位式**：`(tier_id, resource_type, limit_value)` 三欄一列。加一種新資源只是多寫一筆資料，不用 `ALTER TABLE`、不用重新部署。

#### 18.6.3 執行層

- 位置：對話端點的**第一道關卡**，排在 B4 安全檢查**之前**——被擋下的請求不該讓下游付出任何成本，而 B4 自己就要呼叫一次模型
- 原子性：**Lua script**，不是先累加後比對。累加之後才發現超額的話，那次被擋下的請求已經記帳了（用量從 50 變成 51），違反「擋下的請求不記帳」
- **不可「先查再寫」**：兩個併發請求會同時查到「還有額度」然後雙雙通過
- `prompt_max_chars` 是唯一不用計數器的一種——單次靜態檢查，且**必須在配額扣除之前**執行
- 150m 感應聊天與 50m 召喚後對話**共用同一組配額**，不分開算

#### 18.6.4 ★ 未解決：全域上限〔v2.2 標記〕

上述機制全部是 **per player per day**。攔不住「1000 個玩家同時觸頂」——帳單是 1000 倍，而**沒有任何一個閘門會攔**。

封測期人數由發碼控制，故不阻擋垂直切片。但**公開前必須有**：一顆全域日計數器（同一支 Lua，key 不含 `player_id`），超過即整條對話路徑走 fallback 台詞。

排入 §16 缺口清單，時機為封測通過、公開之前。

### 18.7 生成內容的安全與回退

#### 18.7.1 三層邊界

| 層 | 負責 | 涵蓋 |
|---|---|---|
| **B4 角色安全邊界** | 輸入端過濾 | 玩家自由輸入中的不適合、危險、偏離主題內容 |
| **B5 史實邊界** | prompt 注入 | 已知史實／民間傳說／角色想像的分界；不對敏感歷史武斷定論 |
| **人格卡 `taboos`** | prompt 注入 | 該靈魂特有的禁忌，含 §12.2 的宗教安全下限 |

⚠️ **三層不可互相替代。** B5 不涵蓋宗教邊界，人格卡不涵蓋輸入端過濾。§6.5 的注入硬規則就是為了確保第三層出現在**每一條**生成路徑上。

#### 18.7.2 安全下限只能往上加

人格卡的 `taboos` 中，來自 `CONTEXT.md` 與 §12.2 的條目是**安全下限**：敘事審核可以往上加，**不能移除或改寫**。新版本少了任何一條，審核應視為不通過；匯入器須逐條比對前一版，缺項直接拒絕匯入。

#### 18.7.3 回退一律不阻擋玩家進度

| 情境 | 處置 | HTTP |
|---|---|---|
| LLM 呼叫失敗／逾時 | 人工預寫 fallback 台詞 | **200**（非錯誤碼） |
| TTS 失敗 | 不回音檔，文字照常顯示；客戶端維持靜止口型 | 200 |
| 當日情境無合格輸入 | 該靈魂的人工預寫台詞（§20.1.3） | 200 |
| 解鎖敘事生成失敗 | 該階段的人工預寫台詞 | 200 |
| 任務包裝台詞生成失敗 | 人工預寫台詞 | 200 |
| 引導提問生成失敗 | 人工預寫提問 | 200 |

**共同原則**：這些內容是**包裝**，玩家的進度是真的。一次生成失敗不該讓整個流程看起來像出錯。回退台詞本來就寫得像角色會說的話，因此 `is_fallback` 旗標**僅供觀測生成成功率**，不建議用它改變玩家看到的東西。

### 18.8 引導提問（B14）〔v2.2 新增〕

#### 18.8.1 定位

**引導提問是主體驗，自由文字輸入是進階功能。**

理由：玩家站在戶外、單手持機、日照下看螢幕、用中文輸入法打字。降低開口門檻的元件不是輔助，是主要路徑。

⚠️ 但這**不等於**把 LLM 換成罐頭回覆。引導提問作用在**輸入端**，點擊之後仍然走完整的 LLM 對話。玩家覺得「這是機器人」的來源是**重複**，不是「有提問」——固定三句提問才是露餡的地方。

#### 18.8.2 生成規則

| 項目 | 規格 |
|---|---|
| 數量 | 2–3 則 |
| 輸入 | 人格卡（含 `quest_themes`、`taboos`）＋ 當日情境 ＋ 玩家共鳴階段 ＋ 當前 story beat（若有） |
| 變動頻率 | **每日**。同一靈魂跨日再查時內容必須不同 |
| 快取 | 每靈魂每日一次生成，快取當日；成本可忽略（十靈魂 = 每天 10 次呼叫） |
| 回退 | 人工預寫提問，HTTP 200 |
| 模組邊界 | 同 B9：**生成不自己抓資料**，合格輸入由呼叫端提供 |

#### 18.8.3 客戶端行為（S9）

- 點擊後**如同玩家自己打字送出**進入對話流程——不是填入輸入框讓玩家再按一次，那就沒有降低門檻
- 因此點擊送出的提問**同樣經過**配額計數、B4 安全檢查、B12 比對。這條路徑不繞過任何關卡
- 無可用提問時**完全隱藏**元件（不佔版面、不顯示空殼），對話仍可正常打字
- 同一個 prefab 被 MapScene（sense 模式）與 EncounterScene（encounter 模式）共用，只有一份實作

### 18.9 API 契約總表

> 機器可讀的真相是 `citysoul-backend/contracts/openapi.json`（自動產生）。本表定義的是**驗證要求與語意**。

| Method | Path | 驗證 | 說明 |
|---|---|---|---|
| POST | `/api/v1/players` | 無 | 建立／取回匿名身分，核發 Session Token |
| GET | `/api/v1/spirits` | 無 | 所有召喚點，供地圖繪製 |
| GET | `/api/v1/spirits/{placeId}` | 無 | 地標基本資料 ＋ `orientation`（§10.2） |
| POST | `/api/v1/sense` | Session | 150m 感應驗證，核發 Sense Token，**不觸發任務** |
| POST | `/api/v1/summon` | Session | 50m 在場驗證，核發 Encounter Token ＋ 任務狀態 |
| POST | `/api/v1/spirits/{placeId}/dialogue` | Session ＋（Sense 或 Encounter） | 對話。回應含 `suggested_questions`（§10.5） |
| GET | `/api/v1/spirits/{placeId}/suggested-questions` | Session ＋（Sense 或 Encounter） | 引導提問（§10.5，v2.2 新增） |
| GET | `/api/v1/spirits/{placeId}/daily-event` | 無 | 當日情境（公開世界狀態） |
| GET | `/api/v1/quests/daily` | Session | 任務列表 |
| POST | `/api/v1/quests/{questId}/complete` | Session ＋ Encounter | 完成判定 ＋ 共鳴入帳 ＋ 敘事包裝 |
| POST | `/api/v1/quests/{questId}/landmark-photo` | Session ＋ Encounter | B13 辨識，即用即丟 |
| GET | `/api/v1/resonance/{spiritId}` | Session | **唯讀**查詢共鳴進度 |
| GET | `/api/v1/players/me/memory-summary` | Session | Profile 直查身體自己的表，不呼叫腦袋 |
| GET | `/api/v1/assets/{avatarId}` | 無 | Addressables catalog／bundle URL |

**錯誤碼慣例**：

| 碼 | 語意 |
|---|---|
| 401 | Token 無效或過期 |
| 403 | 不在範圍內；或 token 的 `spirit_id` 與路徑不符 |
| 404 | `spirit_id` 不存在**或 `is_active = false`**（下架的靈魂對玩家而言就是不存在） |
| 409 | 任務今日已完成，不可重複完成 |
| 422 | 格式不合法；或後端確定性規則判定未達成完成條件（**且不呼叫 LLM**） |
| 429 | 配額超過。回應須說明哪一種配額用完與重置時間，並以角色口吻呈現 |

⚠️ **404 與「沒有內容」的分界**：地標不存在 → 404；地標存在但今天沒有內容 → **200 ＋ 保底內容**。前者是玩家問錯了東西，後者是我們還沒準備好，兩件事不該給同一個答案。

---

## 19. 驗收標準總表（v2.2 自 v1 吸收，含本次修正）

> **本節是驗收標準的唯一權威**，取代 `city_soul_AR_acceptance_criteria.md`（封存，見 §17.2）。
> 格式為 Given-When-Then。每一條都應該有對應的測試（形式不拘）能證明它成立。
> **時區**：所有「每日一次」「隔天重置」統一以 **Asia/Taipei（UTC+8）午夜**為基準。
>
> §14 保留 v2.1 的異動說明作為歷史脈絡；**逐條驗收請用本節**。

### 19.1 在場驗證與召喚（S2）

- **AC1.1** Given 玩家 GPS 與地標距離 = `summon_radius_meters`（含邊界值） When 呼叫 `/summon` Then 在場驗證通過
- **AC1.2** Given 距離 > `summon_radius_meters`（例如 +1m） When 呼叫 `/summon` Then 回 `403`，不核發 encounter_token
- **AC1.3** Given `spirit_id` 不存在或 `is_active = false` When 呼叫 `/summon` Then 回 `404`
- **AC1.4** Given Session Token 無效或過期 When 呼叫 `/summon` Then 回 `401`
- **AC1.5** Given 在場驗證通過 When 處理完成 Then encounter_token 的 `exp - iat` **精確等於 900 秒**
- **AC1.6** Given 任意一次成功的 `/summon` When 檢查 encounter_token payload Then **不含**任何 GPS 座標欄位

### 19.2 感應驗證（`/sense`）

- **AC2.1** Given 距離 ≤ `sense_radius_m`（150m） When 呼叫 `/sense` Then 核發 sense_token
- **AC2.2** Given 距離 > `sense_radius_m` When 呼叫 `/sense` Then 回 `403`
- **AC2.3** Given 任意一次成功的 `/sense` When 檢查回應 Then **不包含**任何任務欄位（感應不觸發任務發放）
- **AC2.4** Given 成功核發的 sense_token When 檢查 `exp - iat` Then **精確等於 1800 秒**
- **AC2.5** Given sense_token 將於 1–2 分鐘內過期、玩家仍在 150m 內對話 When 前端偵測到 Then 自動靜默換發，玩家 UI **無任何中斷或提示**

### 19.3 Mock Location 防作弊（S3，觀察期）

- **AC3.1** Given `is_mock_location = true` 隨 `/summon` 送出 When 後端處理 Then 寫入一筆 log（含 player_id、spirit_id、時間、依據），**且該次召喚正常通過**
- **AC3.2** Given 同上 When 檢查回應 Then HTTP 狀態碼與正常通過時**相同**

### 19.4 可驗證微任務生命週期（S4）〔v2.2 改寫〕

- **AC4.1** 〔**v2.2 改寫**〕Given 玩家的 encounter_token 已過期、且任務仍是 `in_progress` When 對同一 `spirit_id` 再次呼叫 `/summon` Then `attempts_today` **不增加**，任務可直接重新挑戰
- **AC4.2** 〔**v2.2 改寫**〕Given 任意 `attempts_today` 數值 When 呼叫 `/summon` Then 一律正常核發 encounter_token 與任務嘗試，**不因次數被拒絕**
- **AC4.3** 〔**v2.2 刪除**〕~~`attempts_today >= 3` 時回傳 `daily_limit_reached`~~ → 見 §18.4.2；API 回應中**不得出現**該值
- **AC4.4** Given 跨越 Asia/Taipei 午夜 When 玩家再次查詢 Then `attempts_today` 重置為 0（觀測值的重置，不影響可玩性）
- **AC4.5** Given 任務已於今日完成過 When 再次呼叫完成端點 Then 回 `409`
- **AC4.6** Given `completion_evidence` 格式不符、或確定性規則判定未達成 When 呼叫完成端點 Then 回 `422`，**且不呼叫 LLM**
- **AC4.7** 〔**v2.2 新增**〕Given 玩家在一天內因中斷而反覆讓憑證過期 5 次 When 第 6 次召喚 Then 仍可正常挑戰任務（驗證上限確實移除）

### 19.5 共鳴值（S5）

- **AC5.1** Given 任務完成判定通過 When 完成端點處理完成 Then `resonance_value` 在**同一個請求內**即時 +20
- **AC5.2** Given 某地標的相遇收藏尚未建立 When 建立 Then +10，且**只加這一次**（UNIQUE 約束）
- **AC5.3** Given 共鳴值跨過 10/40/100 任一門檻 When 完成端點內部檢查 Then 呼叫 B11，回應中 `unlock_story` 不為 null
- **AC5.4** Given 未跨過任何新門檻 When 任務完成 Then `unlock_story` 為 null
- **AC5.5** Given 同一 `source_type` + `source_id` 已入過帳 When 嘗試重複入帳 Then 被資料庫約束擋下，不重複加值
- **AC5.6** 〔**v2.2 新增**〕Given 一次入帳同時跨過兩個門檻 When 生成解鎖敘事 Then 產出**兩段**，不是一段

### 19.6 Token 邊界

- **AC6.1** Given encounter_token 內 `spirit_id` 與路徑 `placeId` 不一致 When 驗證 Then 回 `403`
- **AC6.2** Given 只帶 session_token、無 encounter/sense token When 呼叫對話 Then 回 `401`
- **AC6.3** Given 只帶有效 sense_token When 呼叫任務完成端點 Then 被拒絕
- **AC6.4** Given 只帶有效 sense_token When 嘗試進入實境疊層場景 Then 前端阻擋跳轉，維持在地圖畫面

### 19.7 混合式聊天（B12）

- **AC7.1** Given 玩家輸入**完全命中** `canned_greetings.trigger_phrases` 任一詞 When 呼叫對話 Then 直接回預寫台詞，**不呼叫 LLM**（可用呼叫次數／延遲驗證）
- **AC7.2** Given 未命中 When 呼叫對話 Then 正常走 B4→B2→B1→B10 完整流程
- **AC7.3** 〔**v2.2 新增**〕Given 玩家輸入為「你好，龍山寺什麼時候蓋的？」（含招呼語但實為提問） When 比對 Then **不命中**招呼語（比對為完全相等，非子字串）

### 19.8 配額（Usage Tier）

- **AC8.1** Given 今日 `dialogue_calls_daily` 已達上限 When 再次呼叫對話（不論持哪張 token） Then 回 `429`，說明哪種配額用完與重置時間
- **AC8.2** Given 已透過 150m 聊天消耗部分配額 When 走進 50m 召喚後繼續對話 Then 消耗**同一組**計數器
- **AC8.3** Given 輸入字數超過 `prompt_max_chars` When 呼叫對話 Then 回 `422`，**且不消耗**次數配額
- **AC8.4** Given 60 秒內呼叫次數超過 `api_rate_per_minute` When 呼叫受限 API Then 回 `429`
- **AC8.5** 〔**v2.2 新增**〕Given 兩個併發請求同時抵達且僅剩 1 次額度 When 兩者同時檢查 Then 僅一個通過，計數器**不超賣**

### 19.9 Fallback / 服務降級

- **AC9.1** Given LLM 呼叫失敗或逾時 When 對話處理中 Then 回 **HTTP 200**，內容為人工預寫台詞
- **AC9.2** Given 今日尚未產生當日情境快取 When 呼叫當日情境端點 Then 回前一天的內容作為保底，**不回 404 空畫面**
- **AC9.3** Given B9 無合格受控輸入 When 排程觸發生成 Then 使用人工預寫台詞，**不讓 LLM 自由發揮**
- **AC9.4** 〔**v2.2 新增**〕Given 完全沒有任何當日情境快取（新地標剛上線） When 查詢 Then 回該靈魂的人工預寫保底，仍為 200

### 19.10 地圖標記狀態機（前端）

- **AC10.1** Given 距離 > 150m When 檢視地圖 Then 標記維持預設樣式
- **AC10.2** Given 距離 ≤ 150m 且 > 50m When 檢視地圖 Then 標記顯示「可聊天」提示
- **AC10.3** Given 距離 ≤ 50m When 檢視地圖 Then 標記完全點亮、成為可點擊的召喚按鈕
- **AC10.4** Given 玩家點擊已點亮的召喚按鈕 When 點擊 Then 呼叫 `/summon` 並在成功後跳轉；**不會**在僅僅進入 50m、尚未點擊時自動跳轉
- **AC10.5** Given 玩家在 150m 外嘗試開啟聊天 When 互動 Then 顯示模糊提示，**不含精確距離數字**

### 19.11 距離偵測（省電）

- **AC11.1** 〔v2.1 改寫〕Given App 在地圖畫面 When 檢查 GPS 存取模式 Then 為前景區間輪詢、間隔約 **10 秒**，**不要求背景定位權限**
- **AC11.2** Given 已進入實境疊層場景 When 檢查 GPS 存取模式 Then 允許前景持續讀取供漸顯插值
- **AC11.3** Given 輪詢延遲導致 UI 已亮起但點擊當下實際距離超過 50m When 呼叫 `/summon` Then 後端仍回 `403`，**不因 UI 已亮起而略過後端驗證**

### 19.12 實境疊層畫面

- **AC12.1** Given 剛進入實境疊層場景 When 檢視對話窗 Then 預設為**展開**
- **AC12.2** Given 點擊收合 When 收合後 Then 仍可看到相機畫面與 3D 角色，且可再次展開
- **AC12.3** Given 在 50m 內越靠近召喚點 When 持續移動 Then 角色縮放**連續**漸變，非跳躍式
- **AC12.4** Given 原地水平旋轉手機 When 靈魂方位角超出視野 Then 角色停止繪製，螢幕邊緣顯示方向指示器
- **AC12.5** Given 轉回靈魂方位 When 方位角進入視野 Then 角色重新出現於對應螢幕水平位置
- **AC12.6** Given `headingAccuracy` 劣化超過門檻 When 進入或停留於場景 Then 顯示羅盤校正提示

### 19.13 開發環境

- **AC13.1** Given 開發環境啟動 When 檢查資料庫連線目標 Then 指向本地 Postgres/Redis，而非雲端受管服務
- **AC13.2** Given 要切換到雲端 When 檢查程式碼異動範圍 Then **僅需更換連線字串／環境變數**，不需修改 schema 或業務邏輯

### 19.14 地標視覺辨識（B13）

- **AC14.1** Given 上傳有效圖片 When 呼叫辨識端點 Then 回 `200` 且 `landmark_recognized` 為布林值
- **AC14.2** Given 任意一次辨識已完成 When 檢查後端所有儲存體（DB、Cloud Storage、log） Then **找不到**該張照片的任何持久化檔案
- **AC14.3** Given 辨識呼叫失敗或逾時 When 處理 Then 回 `200` 且 `landmark_recognized: false`，**不阻擋**後續任務完成
- **AC14.4** Given 今日辨識配額已達上限 When 呼叫 Then 回 `429`
- **AC14.5** Given `landmark_recognized: false` 但其餘完成條件皆滿足 When 完成任務 Then **仍判定完成**，只是不核發特別徽章
- **AC14.6** Given 上傳非圖片或超過大小限制 When 呼叫 Then 回 `422`，**且不消耗**辨識配額

### 19.15 資產與體積（AC15）

見 §14.4：AC15.1–AC15.6（shape key、面數、bundle 體積、按需下載、灰模可替換、上線 build 不含灰模）。

### 19.16 對嘴與效能（AC16）

- **AC16.1** Given 播放 TTS 音檔 When 觀察 `JawOpen` Then 隨音量包絡**連續**變化
- **AC16.2** 〔**v2.2 改寫**〕Given §5.8.1 定義的**地板機** When 連續運行實境疊層 5 分鐘 Then **全程**幀率 ≥ 30fps（非平均），須附 fps 曲線、最低值與掉幀分佈，且無明顯發熱降頻

### 19.17 建置與隱私（AC17–AC18）

見 §14.4：AC17.1–AC17.3（ARM64 only、APK < 100MB、權限清單不含背景定位與 ARCore 宣告）、AC18.1（Keystore，不得用 `PlayerPrefs`）。

### 19.18 宗教場域內容治理（AC19）

- **AC19.1** Given 玩家詢問求籤、運勢、教義解釋等禁忌主題 When 回應 Then 不含教義性陳述或裁決，且以角色口吻自然轉向
- **AC19.2** Given 檢視人格卡 When 檢查 `not_this_character` Then 明確排除「廟方人員」「任何神祇」「宗教解說員」

### 19.19 v2.2 新增（AC20–AC24）

見 **§14.5**：

- **AC20** 人格卡注入（四條生成路徑、單一組裝函式、宗教邊界擴及全路徑）
- **AC21** 任務失敗語意（不計失敗、無上限、去重仍成立）
- **AC22** 引導提問（每日變動、失敗回退 200、與手動輸入同路徑）
- **AC23** 效能與熱治理（幀率基線、降級階梯、sleepTimeout 還原、完整迴圈耗電）
- **AC24** 內容管線（合格輸入有生產者、fallback 按靈魂各自定義）

### 19.20 尚未涵蓋

以下項目因對應設計尚未定案，暫時無法展開驗收標準：

1. UI 視覺細節（美術呈現、動畫參數）
2. Prompt 組裝的 Top-K／記憶輪數（屬品質調校，非功能對錯）
3. 故事線 beat 推進（§20.4 已定規格，實作後補 AC）

---

## 20. 內容生產管線〔v2.2 新增〕

> **本節補的是同一個結構性缺口**：規格寫了、資料表建了、生成邏輯也寫了，但**沒有人定義資料從哪裡進來**。
>
> 這個缺口不會有錯誤訊息。系統會安靜地一直走 fallback 路徑，測試也會通過——因為 fallback 本來就是正確行為。它只會表現為「玩家隔天回來，什麼都沒變」。

### 20.1 當日情境的合格輸入

#### 20.1.1 問題

B9 定義了三類白名單輸入（日期／節日、地標官方公開活動、人工審核素材），設計乾淨且刻意不讓生成端自己抓資料——那正是「輸入來源限定」這條規則唯一守得住的方式。

但**三類都沒有生產者**：沒有節慶日曆、沒有素材匯入路徑、沒有排程呼叫。結果是白名單恆為空，每一天、每一個靈魂都走人工預寫台詞。

核心迴圈寫著「有理由隔天回來查看變化」。那個變化目前不存在。

#### 20.1.2 生產者定義

| 輸入類別 | 生產者 | 形式 |
|---|---|---|
| 日期／節日 | **節慶日曆表** | 每靈魂一份。含國曆固定節日、農曆節日（初一、十五、佛誕、中元、除夕）、該地標特有的年度祭典（如霞海城隍廟農曆五月十三迎城隍） |
| 地標官方公開活動 | **人工錄入** | 敘事負責人依廟方／館方官網公告錄入，含起訖日期 |
| 人工審核素材 | **預寫輪播池** | 每靈魂 30–60 條短素材，經審核後入池，由排程依日期輪播 |

**排程**：Cloud Scheduler 每日一次（Asia/Taipei 清晨），對每個 `is_active` 的靈魂各觸發一次生成。

**重複觸發是正常的**（重試、多實例、手動補跑），去重靠 `(place_id, event_date)` 主鍵：先寫、撞到就改成更新。不靠排程自己記得。

#### 20.1.3 fallback 按靈魂各自定義

現行的人工預寫保底是**單一全域字串**，且措辭帶香火語彙。在龍山寺切片階段恰好正確，但套到天文館、當代館、松菸會立刻壞掉。

**規格**：fallback 台詞移入**人格卡**，每靈魂各自定義。無該欄位時才退到通用句。

**寫法要求**：寫得像「今天很平常」，不是「系統沒有資料」。大多數日子本來就很平常，而玩家不需要知道我們的內容管線今天是空的。

#### 20.1.4 量能估算

| 階段 | 需求 | 估計工時 |
|---|---|---|
| **垂直切片**（龍山寺 1 隻） | 節慶表 1 份 ＋ 輪播池 30 條 ＋ 排程 | 一人 2–3 天 |
| **MVP**（十靈魂） | 節慶表 10 份 ＋ 輪播池 300–600 條 ＋ 逐字審核 | ★ 見 §16 缺口，切片通過後估算 |

> 十靈魂的量能是**真正的瓶頸**，且瓶頸在審核者而非工程。但它不阻擋垂直切片——這是把切片跑完再擴張的具體理由之一。

### 20.2 引導提問的內容來源

規格見 §18.8。此處補與 §20.1 的關係：

引導提問的輸入之一是**當日情境**。因此 §20.1 的管線接通之後，引導提問自動獲得每日變動的能力——兩者是同一條管線的兩個出口，不是兩套內容。

這也是為什麼引導提問的成本可以忽略：真正花錢的是內容生產（人工），不是生成呼叫（每靈魂每天一次）。

### 20.3 任務目錄

#### 20.3.1 現況與限制

現行實作為「每個靈魂一個固定任務」，`quest_id` 由 `spirit_id` 直接推導。這在垂直切片階段是**正確的簡化**——切片只有一隻靈魂，任務內容也只有一種。

但配上共鳴值規則會算出一個短命的體驗：任務 +20，門檻 100，相遇 +10，因此**單一靈魂 5 次任務完成即封頂**。切片若跑超過一週，玩家在第五天之後就沒有東西可做。

#### 20.3.2 規格

| 項目 | 規格 |
|---|---|
| 來源 | 人格卡的 `quest_themes`（每張卡 3–4 條，內容已存在但目前無任何程式讀取） |
| 輪播 | 每日依日期輪播一條主題，同一玩家同一靈魂每日一個任務（§18.4.4） |
| 判定 | **確定性規則**，不經 LLM（不變） |
| 敘事包裝 | 由 LLM 生成，須注入人格卡（§6.5） |
| 目錄表 | 需要時才引入。在只有輪播的階段，`quest_themes` 即目錄 |

⚠️ **不要把日期編進 `quest_id`**。`quest_progress` 已有 `attempts_date` 欄位；若 `quest_id` 自己帶日期，每天都是新的一列，那個欄位就沒有存在的必要。欄位的存在本身就說明了 `quest_id` 應該跨日穩定。

### 20.4 故事線推進

#### 20.4.1 現況

`story_arcs`、`story_beats`、`resonance_unlockables`、`players_story_progress` 四張表已建立、ORM 已定義，但**沒有任何讀寫邏輯**：沒有 beat 推進、沒有 `prerequisite_beat_ids` 因果鏈判定、沒有對外端點、prompt 組裝不知道玩家踩在哪個 beat 上。

內容面同樣是零：`story/wanhua_district_storyline_aming_v0.2.md` 仍是草案，未有任何一筆進資料庫。

#### 20.4.2 為什麼不排進垂直切片

萬華主線跨**三個地標**（龍山寺、西門紅樓、剝皮寮），而切片只有龍山寺。故事線在單一靈魂上跑不起來——這不是延後，是前提不成立。

切片階段的留存機制是共鳴值三階段，這已經足夠驗證「玩家會不會回來」這個問題。

#### 20.4.3 規格（供切片之後實作）

| 項目 | 規格 |
|---|---|
| 真相來源 | `players_story_progress`。`dialogue_turns.story_beat_id` 只是日誌，**不是判斷依據** |
| 推進判定 | `prerequisite_beat_ids` 構成因果鏈；判定為確定性規則，不由 LLM 決定玩家踩到哪一拍 |
| Prompt 注入 | 當前 beat 須進入 B2 的 User Turn。劇本標記為「全劇情最重的一拍」者（如剝皮寮的反轉時刻）須給予較高敘事優先權 |
| 條件未滿足 | 走該靈魂的 `contingency_notes`（「想不起來」狀態），**不另開劇情節點** |
| 路線設計 | 故事**帶路線**，不強制收集。玩家沒有義務走完所有地標 |

#### 20.4.4 世界觀邊界（已在劇本內處理，此處記錄為規則）

- 城市靈魂**不扮演真人**。劇本中的歷史人物只透過靈魂的**轉述**存在，不得有直接台詞
- 靈魂**不轉述神明的話語**。需要解釋「靈魂怎麼知道某件事」時，一律用靈魂自己的觀察力，不用神明告知（§12.2 taboo）
- 這兩條在 `story_arc_writing_sop.md` 有完整說明，撰寫新劇本前必讀

### 20.5 內容審核負擔（Lead 的實際上限）

| 內容類型 | 切片（龍山寺） | MVP（十靈魂） | 是否直達玩家 |
|---|---|---|---|
| 人格卡 | 1 份（已過審 v3） | 10 份 | 否（經 LLM） |
| `canned_greetings` | 4 條（已審） | 約 30 條 | ✅ **是，逐字讀** |
| `quest_themes` | 4 條 | 約 30 條 | 否 |
| 當日情境輪播池 | 30 條 | 300–600 條 | 否（經 LLM 改寫） |
| 當日情境 fallback | 1 條 | 10 條 | ✅ **是，逐字讀** |
| 引導提問 fallback | 3 條 | 30 條 | ✅ **是，逐字讀** |
| 解鎖敘事 fallback | 3 條 | 30 條 | ✅ **是，逐字讀** |

⚠️ 標示「直達玩家」者**不經 LLM，原樣送到玩家眼前**——措辭就是成品，不是給模型的指令。審核時要逐字讀。

**這張表是首發日期的實際上限，不是後端進度。** 目前 `reviewed_by` 只有一個人。十靈魂展開前，這件事需要一個答案：增加審核人力，或分批上線。
