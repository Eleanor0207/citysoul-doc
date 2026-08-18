# Meshy Blender 外掛程式教學與 3D 列印整合全攻略
版本：v0.6.0+ | 更新：2026 年最新版

## 01. 外掛程式簡介
了解安裝流程、系統需求與核心介面導覽


### 1.1 什麼是 Meshy Blender 外掛程式？

**Meshy Blender Add-on** 是一款將 **Meshy WebApp AI 3D 生成能力** 與 **Blender 專業 3D 建模/3D 列印預備工具** 深度結合的官方插件。透過此外掛，您不僅可以一鍵將網頁上生成的 3D 模型傳送至 Blender 本地場景，還能利用內建的 **模型分析 (Analyze)**、**模型清理 (Clean Up)** 與 **模型編輯 (Model Editing)** 工具，快速解決 AI 幾何模型常見的「非流形 (Non-Manifold)」、「破面漏洞」、「重疊內部面」與「過厚過重」等問題，達到 100% 可順利 3D 列印的「水密 (Watertight)」品質。

---

### 1.2 系統與軟體需求

| 項目 | 最低需求 | 推薦配置 |
| :--- | :--- | :--- |
| **Blender 版本** | Blender 3.6 LTS 或更高 | **Blender 4.0 / 4.1 / 4.2+** |
| **作業系統** | Windows 10/11, macOS 12+, Ubuntu 22.04+ | 64-bit 最新系統 |
| **Meshy 帳號** | 免費版帳號 (支援 Bridge 同步) | Pro / Max 帳號 (享高解析度資產與 PBR 材質) |
| **切片軟體相容** | Bambu Studio, PrusaSlicer, OrcaSlicer, Cura, Chitubox, Lychee | 支援 STL / 3MF 直接匯入 |

---

### 1.3 外掛程式安裝步驟 (兩種方法)

> 💡 **提示**：安裝前無須解壓縮下載的 .zip 檔案！

#### 方法 A：標準選單安裝 (通用方法)
1. 前往 Meshy 官方網站或 GitHub 下載最新的 meshy-blender-addon.zip。
2. 開啟 Blender，點擊頂部選單的 **「編輯 (Edit)」 -> 「偏好設定 (Preferences)」**。
3. 選擇左側 **「外掛程式 (Add-ons)」** 頁籤，點擊右上角的 **「安裝 (Install...)」** 按鈕。
4. 選擇下載好的 .zip 檔案並點擊「安裝外掛程式」。
5. 在搜尋欄輸入 Meshy，勾選開啟 **「3D View: Meshy for Blender」**。

#### 方法 B：Blender 4.x 拖曳安裝 (快捷方法)
* 在 Blender 4.0+ 視圖中，直接將 meshy-blender-addon.zip 檔案拖放入 3D Viewport 視窗內，系統將自動跳出安裝確認。

---

### 1.4 開啟側邊欄 (N-Panel) 介面

安裝並啟用外掛後：
1. 在 Blender 3D 視圖中按下快捷鍵 **N** 鍵開啟側邊欄 (N-Panel)。
2. 找到並點擊 **Meshy** 頁籤。
3. 您將看到四大核心功能面板：
   - 🌉 **Bridge 面板**：管理帳號同步與資產一鍵匯入。
   - 🔍 **Analyze (分析) 面板**：檢查 3D 列印幾何結構缺陷。
   - 🧹 **Clean Up (清理) 面板**：一鍵 Make Manifold 自動修復流形水密。
   - ✂️ **Model Editing (編輯) 面板**：掏空 (Hollow)、底座對齊 (Align XY)、精確縮放與卡榫設計。
   - 📦 **Export (匯出) 面板**：專用 3D 列印格式匯出 (STL / 3MF)。


---

## 02. Bridge 到 Blender
快速將 WebApp 創作的 AI 3D 資產無縫傳送至 Blender 視圖


### 2.1 Bridge (資產橋接) 核心優勢

以往在 Web 端生成 3D 模型後，使用者必須經歷：[下載 GLB] -> [打開 Blender] -> [檔案選單] -> [匯入] -> [重新連結 PBR 材質節點] -> [重新設定尺寸] 的繁瑣流程。

Meshy Bridge 為您提供真正的 **一鍵即時同步**：
* ⚡ **零檔案傳輸等待**：透過雲端 API 或本地通訊，一鍵完成模型載入。
* 🎨 **自動建立 Shader 節點**：自動將 Albedo (顏色)、Normal (法線)、Roughness (粗糙度) 與 Metallic (金屬度) 貼圖連結至 Blender 的 Principled BSDF 節點。
* 📐 **比例單位自動校正**：預設校正為 3D 列印標準公制單位 (Millimeters / Meters)。

---

### 2.2 如何使用 Bridge 傳送資產？

#### 步驟 1：登入與帳號綁定
1. 在 N-Panel 的 **Bridge** 面板中，點擊 **「Login with Meshy」**。
2. 瀏覽器將自動開啟授權頁面，點擊「Allow / 授權」即可完成連結。
   *(註：v0.6.0+ 版本後，基礎模型載入已無需複雜 API Key 設置)*

#### 步驟 2：選擇資產與載入
1. 登入後，面板將列出您在 Meshy WebApp 中生成的最新模型列表 (包含 Text-to-3D、Image-to-3D 與 AI Textures)。
2. 點擊目標模型旁邊的 **「Import to Scene (載入至場景)」**。
3. 可選擇模型精細度版本：
   - **Quad Mesh (四邊形重構版)**：適合後續 Blender 編輯與雕刻。
   - **Tri Mesh (高細節三邊形版)**：適合直接進行 3D 列印分析與修復。

---

### 2.3 材質與貼圖自動處理機制

Bridge 匯入時會自動建立結構完整的 Shader Node Network：
* **Base Color** -> Principled BSDF Color Input (Color Space: sRGB)
* **Normal Map** -> Normal Map Node -> Principled BSDF Normal (Color Space: Non-Color)
* **Roughness / Metallic** -> Roughness & Metallic Inputs (Color Space: Non-Color)

> 💡 **3D 列印小貼士**：如果是準備進行彩色 3D 列印 (例如使用 Bambu Lab 噴頭多色列印或 Full-Color 樹脂列印)，Bridge 匯入的 UV 與 Vertex Color 資料可直接導出為 **3MF** 格式！


---

## 03. 模型分析 (Model Analysis)
專為 3D 列印設計的全方位幾何結構檢查工具


### 3.1 為什麼 AI 生成模型需要分析？

AI 3D 生成模型 (如 Image-to-3D 或 Text-to-3D) 在生成初期是由深度預測與 Marching Cubes 算法構成，經常會產生以下 **3D 列印致命缺陷**：
1. ❌ **Non-Manifold Edges (非流形邊緣)**：一條邊被 3 個以上的面共享，導致切片軟體無法判斷內外部。
2. ❌ **Open Boundary Holes (破洞與未封閉邊界)**：模型內部為空，切片軟體無法計算體積 (Volume)。
3. ❌ **Inverted Normals (法線反轉)**：部分網格法線朝內，導致切片軟體誤以為模型內部是外部。
4. ❌ **Self-Intersecting Faces (自相交面)**：模型網格互相穿透交錯，易引發切片路徑亂碼。
5. ❌ **Loose Geometry (離散浮空碎片)**：生成過程中產生的漂浮雜點點線面。

---

### 3.2 Analyze 面板功能詳解與操作

在 Meshy 側邊欄點擊 **Analyze** 面板，勾選要檢視的項目並按下 **「Run Analysis (執行分析)」**：

| 檢測項目 | 中文名稱 | 說明與危害等級 |
| :--- | :--- | :--- |
| **Non-Manifold** | 非流形檢測 | ⚠️ **高風險**：若 >0 則大多數切片機將拒絕處理或生成錯誤切片 |
| **Bad Contiguity** | 連續性異常 (法線) | ⚠️ **高風險**：法線方向不一致，列印時壁厚會出錯 |
| **Intersect Face** | 自相交面檢測 | 🟡 **中風險**：可能導致列印時局部過度擠出或空打 |
| **Zero Faces / Edges** | 零面積面 / 零長度邊 | 🟡 **中風險**：退化幾何體，影響布林運算 |
| **Loose Vertices** | 孤立頂點與雜點碎片 | 🟢 **低風險**：浮空點會造成列印空打絲線 |

---

### 3.3 視圖即時高亮標記 (Visual Highlighting)

點擊 Analyze 後，外掛程式會在編輯模式 (Edit Mode) 下自動：
* 🔴 **紅色高亮**：標示非流形邊緣 (Non-Manifold Edges) 與破洞。
* 🟡 **黃色高亮**：標示自相交面與退化網格。
* 藍色箭頭：顯示法線 (Normals) 向量方向。

> 📌 **推薦工作流**：在載入模型後，**第一步** 請務必執行「Run Analysis」，確定問題數量後再進入下一階段進行自動清理！


---

## 04. 模型清理 (Model Cleanup)
自動修復常見幾何缺陷，打造 100% 水密 (Watertight) 幾何體


### 4.1 核心修復神器：Make Manifold (一鍵流形化)

**Make Manifold** 是 Meshy 外掛程式中最核心的 3D 列印預備工具。針對 AI 模型的雜亂網格，一鍵點擊即可自動執行以下 **5 大修復步驟**：

1. **Delete Loose Geometry (刪除孤立點線面)**：自動清除所有懸空漂浮的離散碎片。
2. **Merge Duplicate Vertices (依距離合併重複頂點)**：自動融合過度密集的重疊頂點。
3. **Fill Holes (自動封閉孔洞)**：對未閉合的邊界環自動補面。
4. **Delete Internal Faces (刪除內部隱藏面)**：自動清理交錯在模型內部的無用幾何面，大幅減輕檔案大小。
5. **Recalculate Normals Outside (重新計算外部法線)**：強制所有網格法線朝外。

---

### 4.2 Clean Up 面板工具矩陣

* **Make Manifold (一鍵流形修復)**：自動全方位防破面水密修復。
* **Delete Small Pieces (清理離散雜點浮塊)**：當 AI 生成模型邊緣帶有飄浮的散落微點時使用。可設定 **Threshold Volume (體積閾值)**，低於該體積的孤立島嶼將被一次清除。
* **Recalculate Normals (重新計算法線)**：若模型在切片軟體中呈現部分黑影或透光，點擊此按鈕可修復法線翻轉問題。
* **Merge by Distance (距離頂點融合)**：依特定距離閾值融合微小重疊頂點。

---

### 4.3 修復後驗證流程 (Verification)

修復完成後，請依照以下步驟驗證：
1. 再次回到 **Analyze 面板**。
2. 點擊 **「Run Analysis」**。
3. 確認 **Non-Manifold Count 為 0** 且 **Open Boundaries 為 0**。
4. 恭喜！模型已順利轉化為可列印的「水密實體 (Watertight Solid)」。


---

## 05. 模型編輯 (Model Editing)
掏空、底座平貼、精確尺寸縮放與分割卡榫設計


### 5.1 Hollow Tool (模型掏空與放藥/排水孔)

針對 **SLA / DLP 光固化樹脂列印** 與高密度 FDM 列印，實心列印不僅浪費昂貴耗材，還會因為吸盤效應 (Suction Cup Effect) 導致拉軸列印失敗。

#### Hollow 工具參數說明：
* **Offset Distance (壁厚設定)**：建議設定為 **1.5 mm - 2.5 mm** (光固化樹脂標準壁厚)。
* **Offset Direction (掏空方向)**：
  - Inside (向內掏空)：保持模型外觀尺寸 100% 不變 (最常用)。
  - Outside (向外外擴)：用於製作外殼套件。
* **Voxel Size (體繪素解析度)**：控制掏空內壁的精細度，預設 0.5mm。
* **Hollow Duplicate (複製掏空)**：保留原版實心模型，建立備份。
* **Add Drainage Holes (添加排水孔/放藥孔)**：在模型底座隱蔽處放置直徑 3mm - 5mm 的打孔圓柱，使未固化的液態樹脂能在列印後順利流出與清洗。

> 💡 **節省效益**：掏空後可節省約 **60% - 85%** 的樹脂消耗量！

---

### 5.2 Align XY (自動底座壓平與定位)

3D 列印成功的第一要素是「良好的第一層附著 (First Layer Adhesion)」。
1. 在編輯模式下選擇模型底部希望貼平列印床的面 (Faces)。
2. 點擊 Model Editing 面板的 **「Align XY (對齊 XY 平面)」**。
3. 外掛程式會自動旋轉模型，使選取面 100% 水平貼合在 Z=0 平面上。

---

### 5.3 Scale & Bounding Box (精確列印尺寸控制)

不要在切片軟體中憑感覺拉伸！在 Blender 中使用 Meshy Scale 工具：
* 支援輸入真實世界物理尺寸 (如 X: 120mm, Y: 85mm, Z: 150mm)。
* 具備 **Lock Aspect Ratio (等比例鎖定)** 避免模型變形。
* **Volume Check (體積與耗材重量預估)**：預估列印所需的印料克數 (g) 與 Milliliters (ml)。

---

### 5.4 Cut & Pinning (模型分割與卡榫設計)

當模型尺寸超出列印機工作體積 (Build Volume)，或為了避免過多支撐結構時：
1. **Plane Cut (平面切割)**：指定切面位置將模型分割為上半部與下半部。
2. **Auto Pin / Key Joint (自動生成榫卯卡榫)**：外掛程式自動在切割面上生成公母卡榫 (Male/Female Pegs)，列印後無需複雜定位即可精準黏合！


---

## 06. 模型匯出 (Model Export)
多格式匯出與切片軟體一鍵串接


### 6.1 3D 列印匯出格式對比與選擇指南

| 格式副檔名 | 適用場景 | 單位/色彩支援 | 推薦指數 |
| :--- | :--- | :--- | :--- |
| **.3MF** | 現代主流切片機 (Bambu Studio, PrusaSlicer, OrcaSlicer) | 包含公制單位、多色 Vertex Color / 貼圖、切片設定與零件層級 | ⭐⭐⭐⭐⭐ **最高推薦** |
| **.STL** | 傳統單色列印機、光固化切片機 (Chitubox, Lychee) | 僅包含純網格幾何體 (預設 Millimeter) | ⭐⭐⭐⭐ **標準格式** |
| **.OBJ** | 帶有 UV 貼圖的彩色列印機 | 幾何網格 + .MTL 貼圖檔案 | ⭐⭐⭐ **彩印適用** |
| **.GLTF / .GLB** | Web 3D 展示、AR 預覽與數位資產保存 | PBR 完整材質與動畫 | ⭐⭐⭐ **數位轉存** |

---

### 6.2 匯出前的 CheckList 檢查清單

為確保 100% 成功列印，匯出前請核對以下 5 項指標：
- [ ] **1. 全體法線向外**：無黑色翻轉面。
- [ ] **2. 水密流形 (Watertight)**：Analyze 顯示 Non-Manifold = 0。
- [ ] **3. 套用變換 (Apply Transformations)**：在 Blender 中對模型按下 **Ctrl + A -> 「All Transforms」**，確保 Scale 為 1.0, 1.0, 1.0。
- [ ] **4. 單位設定 (Units)**：Blender 長度單位設為 Millimeters (公釐)。
- [ ] **5. Z 軸朝上 (Z-Up Orientation)**：符合 3D 列印機垂直升降坐標系。

---

### 6.3 Meshy Export 面板一鍵匯出

在 N-Panel 的 **Export** 面板中：
1. 選擇匯出目標：Selected Objects (僅選取物件)。
2. 勾選 **「Apply Modifiers (自動套用修飾器)」**。
3. 點擊 **「Export 3MF for 3D Print」** 或 **「Export Binary STL」**。
4. 檔案即刻儲存，可直接拖入 Bambu Studio / PrusaSlicer 開始切片！


---

