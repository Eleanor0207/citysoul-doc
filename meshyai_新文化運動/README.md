# meshyai_新文化運動（臺灣新文化運動紀念館靈魂角色）

> SDD §7.3.1 合規交付資產

## 交付檔案

| 檔案 | 說明 | 大小 |
|---|---|---|
| `Spirit_NewCulture_SDD_Compliant.fbx` | Unity Humanoid FBX（15,248 tris, 1.60m, 4 組原地動畫, 3 個 Shape Keys） | 1.75 MB |
| `Spirit_NewCulture_Albedo.png` | Base Color 基礎色貼圖 | 2048×2048 |
| `Spirit_NewCulture_Normal.png` | 法線貼圖 | 2048×2048 |
| `Spirit_NewCulture_ORM.png` | ORM 融合貼圖（R:AO, G:Roughness, B:Metallic） | 2048×2048 |

## 11 項 SDD §7.3.1 合規檢驗結果

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
| 10 | 動畫位移通道 | 所有 location 值 = 0.000；4 組動作最大漂移 ≤ 0.0005m | 原地動作無漂移 | ✅ 合格 |
| 11 | 貼圖合流 | Albedo / Normal / ORM (2048×2048) | 三張齊備，ORM 通道融合 | ✅ 合格 |

## 動畫原地驗證數據

| Clip | 地板高度 | 最大漂移 | 判定 |
|---|---|---|---|
| Idle | +0.0146 m | 0.0005 m | ✅ |
| Talking | +0.0144 m | 0.0005 m | ✅ |
| Turn | +0.0143 m | 0.0005 m | ✅ |
| Wave | +0.0146 m | 0.0005 m | ✅ |
