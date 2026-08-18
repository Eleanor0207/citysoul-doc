# 城市靈魂 — 文件庫

規格與交付文件的**單一真相來源**。程式碼在另外兩個 repo：

| Repo | 內容 |
|---|---|
| [`citysoul-backend`](https://github.com/city-soul-taipei/citysoul-backend) | FastAPI 後端（🧠 腦袋 ＋ 🦾 身體的後端部分） |
| [`citysoul-client`](https://github.com/city-soul-taipei/citysoul-client) | Unity 6 客戶端（🦾 身體的客戶端部分 ＋ 🎭 臉） |

---

## 文件

| 檔案 | 用途 |
|---|---|
| **`SDD_v2.2_Unity_3D.md`** | **系統設計文件，單一權威。** 與任何其他文件衝突時以此為準 |
| ~~`SDD_v2.1_Unity_3D.md`~~ | **已廢止**，由 v2.2 取代。章節編號未變，僅供追溯 |
| `SDD-v2.1-Overview.html` | 架構總覽（視覺化版本，用瀏覽器開）。⚠️ 內容仍為 v2.1，未含 §18–§20 |
| `citysoul_data_schema.md` | PostgreSQL 完整 schema 與後端邏輯參考（schema 專用權威） |
| `city_soul_AR_landmark_recognition.md` | B13 地標視覺辨識技術方案（SDD 未涵蓋其細節，現行權威） |
| `dev_status/` | Phase 交付清單、排程對應表、進度快照 |
| `landmark/` | 地標史實研究記錄（11 個地標，內容已填並經 Lead 覆核），含建立流程 SOP 與決策摘要 |
| `character/` | 角色設定與人格卡內容 |
| `story/` | 主線劇情（story arc）內容與撰寫 SOP |
| `event/` | B9 當日情境素材（見 SDD §20.1，待撰寫）。主線劇情不放這裡，見 `story/` |
| `archive/` | **已封存的 v1 文件。** 內容已吸收進 SDD §18／§19，僅供追溯，**不得作為實作依據** |

## v2.2 有什麼不一樣

**業務規則與驗收標準搬進 SDD 了。** 在此之前它們住在 `citysoul-backend/docs/`，而那個目錄被 `.gitignore` 排除——等於一半的權威只存在單一台機器上。現在在 SDD §18（業務規則）與 §19（驗收標準）。

其餘變更見 SDD §0.1 與 §1.1。四項會影響實作：任務失敗語意（憑證過期不再算失敗）、人格卡注入硬規則（安全）、引導提問 B14、內容生產管線 §20。

## ⚠️ 接手前先讀 SDD §1

專案已經換過**三次**技術棧：Unity → Flutter 2.5D → **Unity 3D**。舊文件仍散落在
`citysoul-backend/docs/` 與本 repo 的 `archive/`，照著它們實作是這個專案最容易浪費一整個 Sprint 的方式。

SDD §1 是決策推翻對照表，先讀它。幾個已經害過人的地方：

| 看起來像 | 實際上 |
|---|---|
| `docs/SDD.md`（v1） | **已廢止**，描述的是不存在的 Flutter + 2.5D 客戶端 |
| F3「Rive/Lottie 動畫實作」 | **作廢**，改為 Mixamo 綁骨 ＋ Blender shape key |
| F5「播放後端給的 viseme 時間軸」 | **職責反轉**，改為客戶端 uLipSync 即時分析（Google Cloud TTS 不提供 viseme 時間軸，原設計不可實作） |
| B10「TTS ＋ viseme 時間軸」 | **縮減**，只回 `audio_url` |
| `archive/` 裡的「當天最多失敗重試 3 次」 | **v2.2 移除**，見 SDD §18.4.2 |

## 為什麼文件獨立成一個 repo

先前 SDD 同時存在 `citysoul-backend` 根目錄與本機的 `docs/`，結果**兩份真的分歧了**——
一份有輪詢間隔改成 10 秒，另一份有 §7.3.1 灰模交付規格，各自都缺對方的內容。

現在只有這裡這一份。`citysoul-backend` 與 `citysoul-client` 都只放指標，不放副本。

## 這裡不放什麼

- **API 契約** → 在 `citysoul-backend/contracts/openapi.json`（由程式碼自動產生，見 SDD §11.2.1）
- **GitHub issue** → 在各自的 repo，不在文件庫
- **美術原始檔**（`.blend`、ComfyUI 輸出、4K 貼圖）→ 走雲端硬碟，不進任何 git（SDD §11.2）
