# 城市靈魂 — Phase 1 交付清單與驗收標準

> **對應文件**：`SDD_v2.1_Unity_3D.md` §13.3
> **階段**：Phase 1「骨架（上）」
> **期程**：3 個 Sprint × 3 天 = **9 個工作天**
> **驗收人**：🧭 Lead
> **前置條件**：Phase 0 五項 POC 全數通過，且 §11.4 三項硬體緩解措施已落實

---

## 0. 本階段的定位

Phase 1 **不產出可玩的東西**。目標是把三條軸線各自的地基打好，讓 Phase 2 能在 12 天內把端到端打通。

判斷標準只有一句話：

> **Phase 1 結束時，三個模組是否都能「獨立跑起來並被驗證」，且彼此的介面已對齊？**

不要求互相串接（那是 Phase 2 的事），但要求**介面契約已凍結、假資料可替換**。

---

## 1. 🧠 腦袋 Brain 交付清單

### 1.1 程式碼交付

| #     | 交付物                               | 路徑                                  | 說明                                                                 |
| ----- | ------------------------------------ | ------------------------------------- | -------------------------------------------------------------------- |
| BR-1  | FastAPI 專案骨架                     | `citysoul-backend/app/`             | 分層結構：`api/` `services/` `models/` `core/`               |
| BR-2  | docker-compose 開發環境              | `docker-compose.yml`                | Postgres + Redis，一鍵起                                             |
| BR-3  | Alembic migration                    | `alembic/versions/`                 | `players`、`spirits`、`personas` 三張表                        |
| BR-4  | `spirits` 含新欄位                 | 同上                                  | `bearing_deg`、`height_offset_m`（SDD §10.2）                   |
| BR-5  | **Token 簽發與驗證中介層**     | `app/core/tokens.py`                | 三種 token**各自獨立**的驗證邏輯，不得共用                     |
| BR-6  | `POST /api/v1/players`             | `app/api/v1/players.py`             | 匿名建號 → 回 session_token（90 天）                                |
| BR-7  | `POST /api/v1/sense`               | `app/api/v1/sense.py`               | S2 在場驗證：GPS 在 150m 內 → 回 sense_token（30 分鐘）             |
| BR-8  | **B3 人格卡載入器**            | `app/services/persona_loader.py`    | 從 YAML/JSON 載入、含版本欄位、可熱替換                              |
| BR-9  | **B7 短期記憶存取層**          | `app/services/short_term_memory.py` | Redis key =`{player_id}:{spirit_id}`，可存取對話輪次（尚未接 LLM） |
| BR-10 | 錯誤碼統一處理                       | `app/core/errors.py`                | 沿用 v1 定義的錯誤碼                                                 |
| BR-11 | pytest 測試                          | `tests/`                            | BR-6/7/8/9 各至少一組正常與異常案例                                  |
| BR-12 | **`contracts/openapi.json`** | repo 根目錄                           | ★ 契約凍結用，見 §4.1                                              |

### 1.2 龍山寺人格卡（內容交付）

| #     | 交付物                        | 說明                                                                                                          |
| ----- | ----------------------------- | ------------------------------------------------------------------------------------------------------------- |
| BR-13 | `personas/longshan_v0.yaml` | 骨架版本，可載入即可。**內容由 Lead 提供**，腦袋負責 schema 定義與載入                                  |
| BR-14 | 人格卡 schema 文件            | 明列必填欄位：`character_core`、`taboo_topics`、`not_this_character`、`canned_greetings`、`version` |

> ⚠️ `taboo_topics` 的**實際內容**屬 §12.2 龍山寺內容治理，Phase 3 前完成即可。Phase 1 只需欄位存在且可載入。

### 1.3 驗收方式

```bash
# ① 環境
docker-compose up -d
# 預期：Postgres + Redis 起來，無錯誤

# ② migration
alembic upgrade head
# 預期：三張表建立，spirits 含 bearing_deg / height_offset_m

# ③ 建號
curl -X POST localhost:8000/api/v1/players
# 預期：回 session_token，DB 出現一筆匿名 player

# ④ 在場驗證（成功）
curl -X POST localhost:8000/api/v1/sense \
  -H "Authorization: Bearer <session_token>" \
  -d '{"place_id":"longshan","lat":25.0372,"lng":121.4999}'
# 預期：回 sense_token

# ⑤ 在場驗證（失敗，距離超過 150m）
# 預期：回對應錯誤碼，不核發 token

# ⑥ Token 邊界
# 用 sense_token 去呼叫需要 session_token 的端點
# 預期：拒絕（驗證邏輯確實獨立）

# ⑦ 契約
curl localhost:8000/openapi.json > /tmp/o.json && diff /tmp/o.json contracts/openapi.json
# 預期：無差異

# ⑧ 測試
pytest
# 預期：全綠
```

### 1.4 完成定義（DoD）

```
□ 上述 8 項驗收指令全部通過
□ README 寫明如何在乾淨機器上從零起環境
□ 三種 token 的驗證邏輯確實分屬不同函式/中介層（Lead 逐行看過）
□ 人格卡可在不重啟服務的情況下替換版本
□ 短期記憶可寫入、讀取、過期
□ openapi.json 已 commit
```

### 1.5 明確不在 Phase 1 範圍

`POST /summon`、Gemini 串接（B1）、TTS（B10）、長期記憶 pgvector（B6）、Prompt 組裝（B2）、防作弊（S3）、任務（S4）、共鳴值（S5）、GCP 佈署。

---

## 2. 🦾 身體 Body 交付清單

### 2.1 Unity 專案設定（★第一天就要做完）

| #    | 交付物                                                   | 說明                                                        |
| ---- | -------------------------------------------------------- | ----------------------------------------------------------- |
| BO-1 | Unity 6 專案，**URP**                              |                                                             |
| BO-2 | **Force Text serialization ＋ Visible Meta Files** | ⚠️ 事後改會產生大量無意義 diff                            |
| BO-3 | Git LFS 設定                                             | `*.fbx` `*.png` `*.wav` `*.bundle`                  |
| BO-4 | Build target：**Android ARM64 only**               | 不勾 ARMv7                                                  |
| BO-5 | 套件安裝                                                 | Addressables、`com.unity.nuget.newtonsoft-json`、uLipSync |
| BO-6 | 資料夾結構                                               | 依 SDD §5.3，`Characters/` 留給臉                        |

### 2.2 程式碼交付

| #     | 交付物                        | 路徑                                       | 說明                                                                                    |
| ----- | ----------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------- |
| BO-7  | **ApiClient**           | `Services/ApiClient.cs`                  | `UnityWebRequest` 封裝；統一錯誤碼處理；逾時與重試策略                                |
| BO-8  | **TokenManager**        | `Services/TokenManager.cs`               | 三種 token 生命週期；session 自動換發；**sense 靜默續期骨架**（可先不觸發）       |
| BO-9  | 安全儲存                      | `Services/SecureStorage.cs`              | Android Keystore 封裝。**不得使用 `PlayerPrefs`**                               |
| BO-10 | **S1 LocationService**  | `Services/LocationService.cs`            | 10 秒前景輪詢；150m / 50m 門檻狀態機；離開場景即停止；**不申請背景定位權限**      |
| BO-11 | Bootstrap 場景                | `Features/Bootstrap/`                    | 啟動 → 有 token 就帶出、沒有就呼叫`POST /players` → 進下一場景。**無登入 UI** |
| BO-12 | **EncounterScene 雛形** | `Features/Encounter/`                    | 相機背景（`WebCamTexture`）＋ 灰模顯示 ＋ 佔位對話 UI（可收合）                       |
| BO-13 | 環境設定檔                    | `Resources/env.json` 或 ScriptableObject | 後端 base URL 可切換（local / dev）                                                     |

### 2.3 雛形驗收：EncounterScene 應呈現什麼

Phase 1 的 EncounterScene **不需要 3DoF 定向、不需要對嘴、不需要真對話**。只要：

```
✅ 相機畫面鋪滿背景
✅ 灰模站在畫面中央，播放 Idle 動畫
✅ 對話窗預設展開，可收合／再展開（AC12.1 / AC12.2）
✅ 對話窗內顯示寫死的假訊息
✅ 有返回按鈕可離開場景
```

### 2.4 驗收方式

| # | 操作                       | 預期                                                         |
| - | -------------------------- | ------------------------------------------------------------ |
| 1 | 從 CI 下載 APK，安裝到實機 | 可安裝，無崩潰                                               |
| 2 | 首次啟動                   | 背景建號成功，**無任何登入畫面**                       |
| 3 | 殺掉 App 重開              | 不重新建號，帶出既有身分                                     |
| 4 | 用 adb 檢查權限清單        | **無**背景定位權限、**無** ARCore 宣告（AC17.3） |
| 5 | 用 adb 檢查 APK 架構       | 僅 ARM64（AC17.1）                                           |
| 6 | 檢查 token 儲存位置        | Keystore，非`PlayerPrefs`（AC18.1）                        |
| 7 | 手動進 EncounterScene      | 相機起、灰模出現、對話窗可收合                               |
| 8 | 把後端關掉再操作           | 有可讀的錯誤提示，不白畫面、不崩潰                           |
| 9 | 觀察 LocationService log   | 每 30 秒一次，切背景後停止                                   |

### 2.5 完成定義（DoD）

```
□ 上述 9 項驗收全部通過
□ APK 由 GitHub Actions / GameCI 自動產出（非本機手動 build）
□ Assets/_Project/Characters/ 未被身體修改（分工邊界）
□ README 寫明如何切換後端 base URL
```

### 2.6 明確不在 Phase 1 範圍

3DoF 定向（S14）、tile renderer 與地圖（S7）、真對話串接、uLipSync、Animator 狀態機、任務/Profile UI、Addressables 遠端下載。

---

## 3. 🎭 臉 Face 交付清單

### 3.1 灰模交付（主交付物）

完整規格見 SDD **§7.3.1**。此處為交付檢查清單。

| #    | 交付物            | 路徑                                                               | 說明                                                                     |
| ---- | ----------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| FA-1 | 灰模`.fbx`      | `citysoul-client/Assets/_Project/Characters/spirit_longshan_v0/` | 含骨架、四組動畫、shape key                                              |
| FA-2 | Prefab            | 同上                                                               | 節點結構依 §7.3.1 定死                                                  |
| FA-3 | 材質 ×3          | 同上                                                               | `Body` / `Hair` / `Eyes`，棋盤格貼圖                               |
| FA-4 | Addressables 設定 |                                                                    | key =`spirit_longshan`，**本地 group 即可**（remote 是 Phase 5） |

**灰模驗收檢查表**：

```
□ Unity 匯入後 rig type = Humanoid，無警告
□ Blend Shapes 面板可見 JawOpen / Blink_L / Blink_R（大小寫完全相符）
□ 手動拉 JawOpen 至 1.0，嘴部明顯開合且不破面
□ 手動拉 Blink_L / Blink_R，眼皮闔上
□ 四組 clip 皆可 loop：Idle / Talking / Wave / Turn
□ Talking clip 長度 ≥ 10 秒
□ 面數 15,000–20,000 tris（★不得偷懶做低面數）
□ 身高量測 1.60m，腳底貼齊 y=0，面向 +Z
□ 材質 slot 為 Body / Hair / Eyes 三個
□ prefab 節點路徑符合 §7.3.1 結構
□ Addressables key = spirit_longshan
```

### 3.2 F1 概念設計交付

| #    | 交付物                           | 說明                                                               |
| ---- | -------------------------------- | ------------------------------------------------------------------ |
| FA-5 | 龍山寺靈魂概念設計稿             | 至少 3 個方向的草案，供 Lead 選擇                                  |
| FA-6 | **風格一致性規範（草案）** | 供後續 9 隻角色沿用：比例基準、色彩範圍、材質語彙、服裝語彙        |
| FA-7 | 龍山寺主題色定義                 | 對應 SDD §9.1 的 rim / dissolve 配色                              |
| FA-8 | ComfyUI 生成規範                 | 固定 prompt 模板、參考構圖、seed/LoRA 設定（風格一致性的技術基礎） |

> ⚠️ 概念設計須考量 §12.2：龍山寺為宗教場所，角色造型**不得**近似神祇、廟方人員或宗教職事者形象。此為造型硬約束，非美感偏好。

### 3.3 產線紀錄（★容易被忽略但很重要）

| #     | 交付物                     | 說明                                                          |
| ----- | -------------------------- | ------------------------------------------------------------- |
| FA-9  | **灰模製作工時紀錄** | 各步驟實際耗時。用於校準 P0-A 的「單隻 < 4 小時」門檻是否成立 |
| FA-10 | 產線步驟文件               | 一份可重複執行的 SOP，讓第 2 隻不用重新摸索                   |

> Mixamo 免費角色不含任何 blend shape，因此灰模必須實走一次「Blender 加 shape key → 分離眼睛材質 → 匯出」，即正式產線第 ④ 步的預演。**這份工時紀錄是判斷 ComfyUI 路線可行性的關鍵數據。**

### 3.4 完成定義（DoD）

```
□ 灰模檢查表 11 項全數通過
□ 身體確認可在 EncounterScene 正常載入並播放 Idle
□ 概念設計稿已交付且完成一輪 Lead 回饋
□ 工時紀錄與 SOP 已提交
```

### 3.5 明確不在 Phase 1 範圍

正式角色建模、靈魂感 shader（F4）、uLipSync 整合（F5）、Animator 狀態機（F8）、Addressables 遠端打包（F6/F7）、角色 #2–#10。

---

## 4. 跨模組事項

### 4.1 API 契約凍結（Lead 主辦，腦袋配合）

**時點**：Phase 1 第 2 個 Sprint 結束前。

| 步驟                                         | 負責 |
| -------------------------------------------- | ---- |
| 腦袋產出`contracts/openapi.json` 並 commit | 腦袋 |
| Lead 逐條核對是否符合 SDD §10               | Lead |
| 凍結宣告，之後變更需走流程                   | Lead |
| 身體據此手寫或生成 DTO                       | 身體 |

> ⚠️ **CI 契約閘門（SDD §11.2.1）在 Phase 2 才建置**，Phase 1–2 期間契約變更**靠人講**。腦袋若需改動已凍結欄位，必須先告知 Lead 與身體。

### 4.2 阻塞矩陣

| 誰等誰       | 內容                                    | 何時解除             | 風險                           |
| ------------ | --------------------------------------- | -------------------- | ------------------------------ |
| 身體 ← 腦袋 | `POST /players`、`POST /sense` 可用 | Sprint 1.2           | 🟠 身體的 Bootstrap 會卡住     |
| 身體 ← 臉   | 灰模可載入                              | Sprint 1.2           | 🟠 EncounterScene 雛形會卡住   |
| 臉 ← Lead   | 概念設計方向回饋                        | Sprint 1.2           | 🟡                             |
| 全員 ← Lead | repo、CI、龍山寺座標                    | **Sprint 1.1** | 🔴**最高優先，不得延遲** |

**緩解**：身體在腦袋 API 就緒前，用本機 mock server（或寫死 JSON）先開發 ApiClient 與 TokenManager；灰模未到前用 Unity 內建 Capsule 佔位。**兩者都不得成為延期理由。**

### 4.3 Lead 在 Phase 1 的對應交付（給三模組的前置）

| #    | 交付                                   | 時點                     |
| ---- | -------------------------------------- | ------------------------ |
| LD-1 | 兩個 repo 建立、權限開通、LFS 設定     | Sprint 1.1 第 1 天       |
| LD-2 | GitHub Actions / GameCI 可產出 APK     | Sprint 1.1               |
| LD-3 | 龍山寺召喚點座標、`bearing_deg` 初值 | Sprint 1.2（實地勘查後） |
| LD-4 | 龍山寺人格卡 v0 內容                   | Sprint 1.2               |
| LD-5 | 中階 Android 基準機型定義              | Sprint 1.1               |
| LD-6 | 角色身高基準確認（暫定 1.60m）         | Sprint 1.1               |
| LD-7 | 契約凍結宣告                           | Sprint 1.2               |

### 4.4 龍山寺實地勘查（全員同行，Sprint 1.2）

| 待取得                           | 用途                               |
| -------------------------------- | ---------------------------------- |
| 召喚點精確座標                   | S2 在場驗證                        |
| 站在召喚點時，靈魂應出現的方位角 | `bearing_deg`                    |
| 現場 GPS 誤差實測（多點多時段）  | 判斷`summon_radius_m` 是否需調整 |
| 現場羅盤`headingAccuracy` 實測 | S14 可行性再確認                   |
| 人潮密度與現場動線觀察           | 現場行為指引設計                   |
| 現場照片                         | 概念設計參考、圖磚風格參考         |

> 建議攜帶兩台不同 SoC 的實機同時量測。

---

## 5. Phase 1 驗收會議

**時點**：Sprint 1.3 最後一天。

**議程**：

1. 腦袋 demo：跑一次 §1.3 的 8 項指令
2. 身體 demo：實機安裝 APK，跑一次 §2.4 的 9 項操作
3. 臉 demo：灰模在 Unity 內展示 shape key 與四組動畫；提交工時紀錄
4. **三方對介面**：確認契約凍結內容、灰模規格、資料夾邊界無爭議
5. **Phase 2 風險盤點**：見下

**Phase 1 未通過的處理**：不順延 Phase 2 開始時間，改為將未完成項併入 Phase 2 第一個 Sprint，並在會議上明確記錄誰欠什麼。

---

## 6. ⚠️ 已知風險

| 風險                                | 說明                                                        | 建議                                                                      |
| ----------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------- |
| **身體的 Phase 1 工作量偏重** | 9 天內要完成 Unity 專案初始化 ＋ 4 個 Service ＋ 2 個場景   | 若第 2 個 Sprint 結束仍落後，優先砍 EncounterScene 雛形（可併入 Phase 2） |
| **臉的 Unity 能力缺口未確認** | SDD §6.3【待確認】。Phase 1 的灰模交付會第一次暴露這個問題 | 灰模若卡在 Unity 端（prefab／Addressables），立即啟動補人力評估           |
| **契約閘門尚未建立**          | Phase 1–2 靠人講，容易漏                                   | Lead 每個 Sprint 結束時手動比對一次 openapi.json                          |
| **實地勘查可能改變技術參數**  | 若龍山寺 GPS 誤差過大或羅盤不穩，S2/S14 需調整              | 勘查排在 Sprint 1.2，留一個 Sprint 的反應時間                             |
| **人格卡內容為 Lead 單點**    | LD-4 若延遲，腦袋的 B3 無實際內容可測                       | 可先用天文館或虛構地標的假人格卡驗證載入器                                |
