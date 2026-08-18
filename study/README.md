# 學習與考察文檔（Study Directory）

本目錄用於記錄《城市靈魂 CitySoul》專案在 **3D 角色資產產線、美術視覺風格考察、地標史實轉譯** 過程中的研究成果與技術規格對齊。

---

## 1. 3D 角色資產產線研究 (Asset Pipeline & Meshy AI)

對應技術規範文件：[SDD_v2.1_Unity_3D.md](file:///Users/doudou/Desktop/%E5%9C%98%E5%B0%88%E8%B7%AF%E5%BE%91/citysoul-doc/SDD_v2.1_Unity_3D.md) §7

詳細標準作業手冊：[meshy-to-unity-character-pipeline.md](file:///Users/doudou/.gemini/antigravity-ide/scratch/citysoul-doc/study/meshy-to-unity-character-pipeline.md)

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
  - 專案檔：[`NewCulture_SDD_Final.blend`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/NewCulture_SDD_Final.blend)
  - Unity FBX：[`Spirit_NewCulture_SDD_Compliant.fbx`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Spirit_NewCulture_SDD_Compliant.fbx)（1.82 MB，11 項規格全數通過）
  - ORM 貼圖：[`Spirit_NewCulture_ORM.png`](file:///Users/doudou/Downloads/meshyai_%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95/Spirit_NewCulture_ORM.png)
- **`Meshy_AI_The_Archivist_s_Appre_biped`**（檔案員靈魂角色）
  - 專案檔：[`Archivist_SDD_Final.blend`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Archivist_SDD_Final.blend)
  - Unity FBX：[`Spirit_Archivist_SDD_Compliant.fbx`](file:///Users/doudou/Downloads/Meshy_AI_The_Archivist_s_Appre_biped/Spirit_Archivist_SDD_Compliant.fbx)

---

## 2. 地標史實與設定檔轉換 (Landmark Research to YAML)

對應地標研究範本：[臺灣新文化運動紀念館_史實研究記錄範本.md](file:///Users/doudou/Desktop/%E5%9C%98%E5%B0%88%E8%B7%AF%E5%BE%91/citysoul-doc/landmark/%E8%87%BA%E7%81%A3%E6%96%B0%E6%96%87%E5%8C%96%E9%81%8B%E5%8B%95%E7%B4%80%E5%BF%B5%E9%A4%A8_%E5%8F%B2%E5%AF%A6%E7%A0%94%E7%A9%B6%E8%A8%98%E9%8C%84%E7%AF%84%E6%9C%AC.md)

### 完成進度：
- [x] 完成史實研究與表格填寫（1921年臺灣文化協會成立、1933年臺北北警察署落成、扇形拘留所與水牢遺跡）。
- [x] Lead 審核通過中立措辭與敏感政治語意隔離。
- [x] 成功將草稿轉譯為正式 YAML 設定檔：[`landmarks/newculture.yaml`](file:///Users/doudou/Desktop/%E5%9C%98%E5%B0%88%E8%B7%AF%E5%BE%91/citysoul-doc/landmarks/newculture.yaml)。
- [x] 已將「蔣渭水關押於上一代木造北署（已拆除）」寫入 `common_misconceptions` 修正對話庫。
