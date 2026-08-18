# 學習與考察文檔（Study Directory）

本目錄用於記錄《城市靈魂 CitySoul》專案在 **3D 角色資產產線、美術視覺風格考察、地標史實轉譯** 過程中的研究成果與技術規格對齊。

---

## 1. 3D 角色資產產線研究 (Asset Pipeline & Meshy AI)

對應技術規範文件：[`SDD_v2.1_Unity_3D.md`](../SDD_v2.1_Unity_3D.md) §7

詳細標準作業手冊：[`meshy-to-unity-character-pipeline.md`](meshy-to-unity-character-pipeline.md)

### 產線標準流程 (Pipeline Step)
```
① AI 3D Mesh 生成 (Meshy / Hunyuan3D)
   ↓
② Blender 動作清理 (移除 Location 位移軌道、刪除多餘動作、長度校正 Talking ≥ 10s)
   ↓
③ Blender 座標與尺度校正 (Y-up 轉 Z-up、等比縮放至 1.60m、腳底貼齊原點)
   ↓
④ Blender 網格分離 (separate 拆出 Body / Hair / Eyes 三個獨立物件)
   ↓
⑤ Prefab 階層綁定 (建立 spirit_root Empty 根節點)
   ↓
⑥ Blender Shape Keys 補全 (JawOpen 對嘴, Blink_L, Blink_R 眨眼)
   ↓
⑦ PBR 貼圖與 ORM 合併 (Albedo / Normal / ORM 各一，2048×2048)
   ↓
⑧ 匯出 Unity 相容 Humanoid FBX 檔 (< 20,000 tris, 1.60m 身高)
```

### 已驗收資產與對齊紀錄
- **`meshyai_新文化運動`**（臺灣新文化運動紀念館靈魂角色）
  - **狀態**：✅ **驗收通過（11 項 SDD §7.3.1 硬性標準全數合格）**
  - **交付資產**：
    - 專案檔：[`NewCulture_SDD_Final.blend`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/NewCulture_SDD_Final.blend) (15.36 MB)
    - Unity FBX：[`Spirit_NewCulture_SDD_Compliant.fbx`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Spirit_NewCulture_SDD_Compliant.fbx) (1.82 MB)
    - ORM 貼圖：[`Spirit_NewCulture_ORM.png`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Spirit_NewCulture_ORM.png) (2048×2048)
    - Albedo 貼圖：[`Meshy_AI_Clockwork_Academy_Sch_biped_texture_0.png`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Meshy_AI_Clockwork_Academy_Sch_biped_texture_0.png) (2048×2048)
    - Normal 貼圖：[`Meshy_AI_Clockwork_Academy_Sch_biped_texture_0_normal.png`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Meshy_AI_Clockwork_Academy_Sch_biped_texture_0_normal.png) (2048×2048)
  - **11 項實測合規檢驗表**：
    | # | 檢查項目 | 實測值 | 規格標準 | 判定 |
    |---|---|---|---|---|
    | 1 | 面數 (Tris) | 15,248 tris | 15,000 – 20,000 tris | ✅ 合格 |
    | 2 | 身高尺度 | 1.60 m | 1.60 m (Z-up) | ✅ 合格 |
    | 3 | 腳底位置 | (0.0, 0.0, 0.0) | 腳底貼齊原點 | ✅ 合格 |
    | 4 | Transform Scale | (1.0, 1.0, 1.0) | (1.0, 1.0, 1.0) | ✅ 合格 |
    | 5 | Prefab 根節點 | `spirit_root` (Empty) | 必須存在根節點 | ✅ 合格 |
    | 6 | 網格獨立性 | `Body` / `Hair` / `Eyes` | 3 個獨立 Mesh 物件 | ✅ 合格 |
    | 7 | 材質配置 | 各物件單一材質槽 | 獨立材質槽掛載 | ✅ 合格 |
    | 8 | Blend Shapes | `JawOpen`, `Blink_L`, `Blink_R` | 位於 Body 物件上 | ✅ 合格 |
    | 9 | 動畫命名與內容 | `Idle`(6s), `Talking`(10s), `Wave`(3s), `Turn`(3s) | 標準 4 組，Talking ≥ 10s | ✅ 合格 |
    | 10 | 動畫位移通道 | 0 條 Location FCurves | 剔除 Location 位移 | ✅ 合格 |
    | 11 | 貼圖合流 | Albedo / Normal / ORM (2048×2048) | R:AO, G:Rough, B:Metal | ✅ 合格 |
- **`Meshy_AI_The_Archivist_s_Appre_biped`**（檔案員靈魂角色）
  - **狀態**：✅ **產線重跑完成（11 項 SDD §7.3.1 硬性標準全數驗收合格）**
  - **交付資產**：
    - 專案檔：[`Archivist_SDD_Final.blend`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Archivist_SDD_Final.blend) (18.78 MB)
    - Unity FBX：[`Spirit_Archivist_SDD_Compliant.fbx`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Spirit_Archivist_SDD_Compliant.fbx) (1.76 MB)
    - ORM 貼圖：[`Spirit_Archivist_ORM.png`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Spirit_Archivist_ORM.png) (2048×2048)
    - Albedo 貼圖：[`Meshy_AI_The_Archivist_s_Appre_biped_texture_0.png`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Meshy_AI_The_Archivist_s_Appre_biped_texture_0.png) (2048×2048)
    - Normal 貼圖：[`Meshy_AI_The_Archivist_s_Appre_biped_texture_0_normal.png`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Meshy_AI_The_Archivist_s_Appre_biped_texture_0_normal.png) (2048×2048)
  - **11 項實測合規檢驗表**：
    | # | 檢查項目 | 實測值 | 規格標準 | 判定 |
    |---|---|---|---|---|
    | 1 | 面數 (Tris) | 15,282 tris | 15,000 – 20,000 tris | ✅ 合格 |
    | 2 | 身高尺度 | 1.60 m | 1.60 m (Z-up) | ✅ 合格 |
    | 3 | 腳底位置 | (0.0, 0.0, 0.0) | 腳底貼齊原點 | ✅ 合格 |
    | 4 | Transform Scale | (1.0, 1.0, 1.0) | (1.0, 1.0, 1.0) | ✅ 合格 |
    | 5 | Prefab 根節點 | `spirit_root` (Empty) | 必須存在根節點 | ✅ 合格 |
    | 6 | 網格獨立性 | `Body` / `Hair` / `Eyes` | 3 個獨立 Mesh 物件 | ✅ 合格 |
    | 7 | 材質配置 | 各物件單一材質槽 | 獨立材質槽掛載 | ✅ 合格 |
    | 8 | Blend Shapes | `JawOpen`, `Blink_L`, `Blink_R` | 位於 Body 物件上 | ✅ 合格 |
    | 9 | 動畫命名與內容 | `Idle`(6s), `Talking`(10s), `Wave`(3s), `Turn`(3s) | 標準 4 組，Talking ≥ 10s | ✅ 合格 |
    | 10 | 動畫位移通道 | 0 條 Location FCurves | 剔除 Location 位移 | ✅ 合格 |
    | 11 | 貼圖合流 | Albedo / Normal / ORM (2048×2048) | R:AO, G:Rough, B:Metal | ✅ 合格 |

---

## 2. 地標史實與設定檔轉換 (Landmark Research to YAML)

對應地標研究範本：[`landmark/臺灣新文化運動紀念館_史實研究記錄範本.md`](../landmark/臺灣新文化運動紀念館_史實研究記錄範本.md)

### 完成進度：
- [x] 完成史實研究與表格填寫（1921年臺灣文化協會成立、1933年臺北北警察署落成、扇形拘留所與水牢遺跡）。
- [x] Lead 審核通過中立措辭與敏感政治語意隔離。
- [x] 成功將草稿轉譯為正式 YAML 設定檔：[`landmarks/newculture.yaml`](../landmarks/newculture.yaml)。
- [x] 已將「蔣渭水關押於上一代木造北署（已拆除）」寫入 `common_misconceptions` 修正對話庫。
