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
| **`SDD_v2.1_Unity_3D.md`** | **系統設計文件，單一權威。** 與任何其他文件衝突時以此為準 |
| `SDD-v2.1-Overview.html` | 架構總覽（視覺化版本，用瀏覽器開） |
| `Phase1_交付清單與驗收標準.md` | Phase 1 各模組的交付項目與驗收指令 |
| `SDD_v2.1_Phase排程與Issue對應表.md` | §13 七階段排程對到兩個 repo 的 issue。**索引性質，狀態以 GitHub 為準** |
| `citysoul_data_schema.md` | PostgreSQL 完整 schema 與後端邏輯參考 |
| `landmark/` | 地標史實研究記錄範本（11 份，多數待填），含建立流程 SOP |
| `character/` | 角色設定與人格卡內容（待撰寫） |
| `story/` | 主線劇情（story arc）內容與撰寫 SOP |
| `event/` | B9 當日情境素材（待撰寫）。主線劇情不放這裡，見 `story/` |

## ⚠️ 接手前先讀 SDD §1

專案已經換過**三次**技術棧：Unity → Flutter 2.5D → **Unity 3D**。舊文件仍散落在
`citysoul-backend/docs/`，照著它們實作是這個專案最容易浪費一整個 Sprint 的方式。

SDD §1 是決策推翻對照表，先讀它。幾個已經害過人的地方：

| 看起來像 | 實際上 |
|---|---|
| `docs/SDD.md`（v1） | **已廢止**，描述的是不存在的 Flutter + 2.5D 客戶端 |
| F3「Rive/Lottie 動畫實作」 | **作廢**，改為 Mixamo 綁骨 ＋ Blender shape key |
| F5「播放後端給的 viseme 時間軸」 | **職責反轉**，改為客戶端 uLipSync 即時分析（Google Cloud TTS 不提供 viseme 時間軸，原設計不可實作） |
| B10「TTS ＋ viseme 時間軸」 | **縮減**，只回 `audio_url` |

## 為什麼文件獨立成一個 repo

先前 SDD 同時存在 `citysoul-backend` 根目錄與本機的 `docs/`，結果**兩份真的分歧了**——
一份有輪詢間隔改成 10 秒，另一份有 §7.3.1 灰模交付規格，各自都缺對方的內容。

現在只有這裡這一份。`citysoul-backend` 與 `citysoul-client` 都只放指標，不放副本。

## 這裡不放什麼

- **API 契約** → 在 `citysoul-backend/contracts/openapi.json`（由程式碼自動產生，見 SDD §11.2.1）
- **GitHub issue** → 在各自的 repo，不在文件庫
- **美術原始檔**（`.blend`、ComfyUI 輸出、4K 貼圖）→ 走雲端硬碟，不進任何 git（SDD §11.2）
