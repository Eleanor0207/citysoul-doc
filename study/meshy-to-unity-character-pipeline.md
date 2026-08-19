# Meshy to Unity 3D 角色資產標準化產線規格與實作手冊

> **文檔版本**：v1.0  
> **適用規格**：[`SDD_v2.1_Unity_3D.md §7.3.1`](../SDD_v2.1_Unity_3D.md)  
> **目標引擎**：Unity 6 (Humanoid Rig, uLipSync, Addressables)

---

## 1. 產線總覽與架構

本產線用於將 Meshy AI 生成之 3D Biped 模型，轉換為完全相容於 Unity 專案架構（uLipSync 對嘴、Animator 狀態機、Shader 靈魂感、Addressables 熱更新）的標準化資產。

```
Meshy AI (.fbx / PBR Textures)
  ↓
[Blender 產線自動化處理]
  1. 動作清理 (移除位移通道、刪除多餘動作、標準命名、迴圈長度校正)
  2. 座標與尺度校正 (Y-up 轉 Z-up、等比縮放至 1.60m、腳底貼齊原點)
  3. 網格拆分 (Body / Hair / Eyes 獨立物件)
  4. 節點階層建立 (spirit_root Empty 根節點綁定)
  5. 臉部 Blend Shapes 補全 (JawOpen、Blink_L、Blink_R)
  6. PBR 材質與 ORM 貼圖通道合併
  ↓
匯出交付資產 (Spirit_{ID}_SDD_Compliant.fbx + ORM 貼圖 + .blend)
```

---

## 2. SDD v2.1 §7.3.1 硬性規格要求

| 項目 | 規格標準 | 判定說明 |
| :--- | :--- | :--- |
| **面數 (Tris)** | 15,000 – 20,000 tris | 總三角面數，不得過高或過低 |
| **身高與位置** | 身高 1.60m，腳底 (0,0,0) | Unity 距離縮放插值基準 |
| **面向** | 面向 +Z 軸 | Unity 標準角色朝向 |
| **Transform Scale** | 全部為 `(1.0, 1.0, 1.0)` | Armature 與 Mesh 物件皆須 Apply |
| **Prefab 節點路徑** | 嚴格固定 4 個子節點 | 供 uLipSync (F5) 與 Animator (F8) 字串綁定 |
| **網格獨立性** | Body, Hair, Eyes 三個獨立物件 | 必須為獨立 Mesh 物件，而非單一網格多 Slot |
| **Shape Keys** | `JawOpen`, `Blink_L`, `Blink_R` | 建於 Body 物件上，大小寫敏感 |
| **動畫 Clips** | `Idle`, `Talking`, `Wave`, `Turn` | 移除 Location 位移通道；Talking 長度 ≥ 10.0 秒 |
| **貼圖規格** | Albedo / Normal / ORM (2048×2048) | ORM = R:Occlusion, G:Roughness, B:Metallic |

---

## 3. Prefab 節點結構標準

```
spirit_root                 (Empty 根節點，掛載 Animator + uLipSync)
├── Armature/               (24 根 Mixamo 標準命名骨架)
├── Body                    (SkinnedMeshRenderer，含 JawOpen, Blink_L, Blink_R)
├── Hair                    (SkinnedMeshRenderer)
└── Eyes                    (SkinnedMeshRenderer)
```

---

## 4. 關鍵處理解析與常見踩坑點

### 4.1 網格物件拆分 vs. 材質槽
- **常見錯誤**：在單一 `char1` 網格上建立 3 個 Material Slots。
- **正確做法**：必須在編輯模式下依材質區域選取面，執行 `separate(type='MATERIAL')`，將其拆分為 `Body`、`Hair`、`Eyes` 三個獨立的物件。

### 4.2 座標軸向校正 (Y-up to Z-up)
- Meshy 輸出的 FBX 預設為 Y-up（Y 軸為高度方向），在 Blender 中 Apply Transform 後會使 Y 變成身高軸。
- **正確做法**：Apply 變換後，繞 X 軸旋轉 -90° 轉為 Z-up，再進行 1.60m 等比縮放與腳底歸零。

### 4.3 動畫位移通道移除 (Root Motion Drift)
- Meshy 動畫帶有 Hip 的 Location 位移軌道，在 Unity 播放時會導致角色漂離原點。
- **正確做法**：反向遍歷並刪除 Action 中的所有 `location` fcurves，僅保留旋轉與縮放軌道。
- **Blender 5.2+ API 注意**：新版採用 Layered Actions，需透過 `action.layers[].strips[].channelbags[].fcurves` 存取。

### 4.4 ORM 貼圖合併
- **R 通道 (Red)**：Ambient Occlusion（預設填入 1.0 白色）
- **G 通道 (Green)**：Roughness 粗糙度貼圖
- **B 通道 (Blue)**：Metallic 金屬度貼圖

---

## 5. 驗收完成資產紀錄

### 臺灣新文化運動紀念館靈魂角色
- **資產目錄**：[`meshyai_新文化運動/`](../meshyai_新文化運動/)
- **FBX**：[`Spirit_NewCulture_SDD_Compliant.fbx`](../meshyai_新文化運動/Spirit_NewCulture_SDD_Compliant.fbx) (1.75 MB)
- **Base Color 貼圖**：[`Spirit_NewCulture_Albedo.png`](../meshyai_新文化運動/Spirit_NewCulture_Albedo.png) (2048×2048)
- **Normal 貼圖**：[`Spirit_NewCulture_Normal.png`](../meshyai_新文化運動/Spirit_NewCulture_Normal.png) (2048×2048)
- **ORM 貼圖**：[`Spirit_NewCulture_ORM.png`](../meshyai_新文化運動/Spirit_NewCulture_ORM.png) (2048×2048)
- **交付檢驗結果**：11 項規格全數通過 ✅

