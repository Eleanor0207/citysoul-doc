# Meshy → Blender → Unity 角色產線（完整版）

> 依 `spirit_palace_v0`（故宮靈魂）在 2026-08-15 從 Meshy 下載跑到 Unity 相遇場景
> 可播放的完整過程整理，中途也用 `bopiliao_v0` 與 `spirit_ximen_v0` 交叉驗證過。
>
> 這份跟 `meshy-blender-character-pipeline.md` 的關係：那份只寫到 Blender 匯出，
> 而且它 §10 記的「Humanoid retarget 塌陷，跨角色的系統性問題，原因未知」**已經
> 查出來了**——見本文 §7.1。兩份都留著，那份是當時的記錄，這份是現在的做法。
>
> 規格數字一律以 `SDD_v2.1_Unity_3D.md` §7.3 / §7.3.1 為準。本文只講怎麼做。

---

## 0. 全流程

```
Meshy 下載 fbx
  → Blender：動作清理 → 身高原點 → 節點結構 → 材質分離 → shape key
  → 貼圖：8192 縮 2048、打包 ORM
  → 匯出 fbx（不內嵌貼圖）
  → Unity：匯入設定 → 材質 → prefab → Animator → 場景接線
  → 驗證：量骨骼世界座標
```

每一段都有「照著做會過」與「不照著做會安靜地壞掉」兩種結果。標 ⚠️ 的是實際踩過的。

---

## 1. Meshy 下載

- 模型類型選 **智慧拓撲**，不要選標準（標準是幾百萬面的原始掃描網格）
- 幀率選 **30**
- 動作庫搜「待機」「說話」「轉身」，**不要搜走路**——站立對話的 NPC 用不到，
  硬套的話 `Idle` 會變成一直走位
- 下載時選 **「全部已新增」**，會打包成單一 fbx（骨架＋多動作）。分開下載會得到
  每個動作各一個檔、每個都帶完整網格（實測 6 個檔各 61 MB）

**四組必要 clip**：`Idle`、`Talking`、`Wave`、`Turn`。其中 `Talking` 規格要求
**≥ 10 秒**，Meshy 的說話動作通常只有 3～5 秒，見 §2.5 的接法。

⚠️ **坐姿的待機不能用**。動作庫裡叫 `Chair_Sit_Idle` 之類的是坐著的，名字有
「Idle」但角色是坐的。

---

## 2. Blender：動作

### 2.1 只留四組，改成純名稱

Meshy 的命名很亂（`Idle_12`、`Talk_with_Right_Hand_Open`、`Idle_Turn_Right`）。
改成 `Idle` / `Talking` / `Wave` / `Turn`，其餘刪掉。

⚠️ Unity 匯入後 clip 名稱會變成 `Armature|Idle`（Blender 匯出器的 take 命名規則），
要在 Unity 的匯入設定裡改回純名稱，見 §6.2。

### 2.2 清掉物件層級的 fcurve

Meshy 匯出的動作，**物件層級**（不是骨頭）被烘了 location / rotation / scale 關鍵幀。

症狀：Blender 裡把 `armature.scale` 改成 1 並 apply 過了，**只要切換時間軸幀數，
縮放又跳回原本的錯誤數值**，怎麼修都「修不好」。

```python
for layer in action.layers:
    for strip in layer.strips:
        for slot in action.slots:
            bag = strip.channelbag(slot)
            for fcurve in [f for f in bag.fcurves
                           if f.data_path in ("location", "rotation_euler", "scale")]:
                bag.fcurves.remove(fcurve)
```

改完切幾個不同幀確認 `armature.scale` 穩定維持 `(1,1,1)` 才算修好。

### 2.3 移除根骨的位移通道

Meshy 把角色的世界擺位烘進了 `Hips` 的 location。原本的 rest 位置也在同一個偏移上，
兩者互相抵消才看起來正常；一旦把 rest 歸零（§3），那段偏移就浮出來。

實測未處理時的漂移：`Idle` 0.49 m、`Talking` 1.80 m、`Wave` 1.51 m、
**`Turn` 7.64 m**。

站著對話的 NPC 不需要位移，把**所有** `.location` 通道整條移除，只留旋轉：

```python
for fcurve in [f for f in bag.fcurves if f.data_path.endswith(".location")]:
    bag.fcurves.remove(fcurve)
```

處理後：四組動作全程腳底在 ±0.01 m 內、最大漂移 0.114 m（自然的重心轉移）。
順帶免掉 Unity 端 Root Transform 的 Bake Into Pose 設定。

### 2.4 動作長度

實測 Meshy 動作的長度（故宮那次）：`Idle` 6.00 s、`Turn` 1.10 s、`Wave` 4.10 s、
`Talking` **3.77 s**。

### 2.5 `Talking` 接到 ≥ 10 秒

它本身是 loop，把關鍵幀複製接三次即可，接點不會有斷差：

```python
length = end - start
for cycle in range(1, 3):
    shift = length * cycle
    for frame, value, handle_left, handle_right, interpolation in originals:
        if frame == start:
            continue          # 每段的第一幀＝上一段的最後一幀，跳過不要疊
        point = fcurve.keyframe_points.insert(frame + shift, value)
        point.handle_left = (handle_left[0] + shift, handle_left[1])
        point.handle_right = (handle_right[0] + shift, handle_right[1])
        point.interpolation = interpolation
```

3.77 s × 3 = 11.30 s，通過。

---

## 3. Blender：身高與原點

目標：身高 **1.60 m**、腳底中心在 **(0,0,0)**、所有物件的 scale 都是 **1**。

### 3.1 ⚠️ `animation_data_clear()` 不會重置姿勢

**這是最會讓人白做工的一個坑。** 清掉 action 之後，pose bone 仍然停在最後評估的
姿勢上，之後量到的身高、bounds、原點**全部是假的**。

我們第一次量到「1.60 m、腳底歸零」並回報完成，實際上是在殘留姿勢下量的；
真實的靜止身高是 1.92 m。

```python
armature.animation_data_clear()
for bone in armature.pose.bones:
    bone.matrix_basis = mathutils.Matrix.Identity(4)   # 這一行才是重點
bpy.context.view_layer.update()
```

### 3.2 量在資料層，不要量在世界座標

蒙皮網格的 `bound_box` 是 rest 資料的框，跟骨架把它擺到哪裡無關；而
`matrix_world` 有沒有包含父層、`matrix_parent_inverse` 有沒有殘留，都會讓
世界座標的量測失真。物件矩陣清乾淨之後，直接量頂點座標最可靠：

```python
lo = Vector((inf,) * 3); hi = Vector((-inf,) * 3)
for piece in meshes:
    for vertex in piece.data.vertices:
        for axis in range(3):
            lo[axis] = min(lo[axis], vertex.co[axis])
            hi[axis] = max(hi[axis], vertex.co[axis])
scale = 1.60 / (hi.z - lo.z)
offset = Vector(((lo.x + hi.x) / 2, (lo.y + hi.y) / 2, lo.z))
```

⚠️ 也要確認自己量的是**哪一根軸**。第一次做故宮時量到 Z 是 0.59——那是深度不是
身高，縮放完變成「深度 1.6 公尺的人」。

### 3.3 ⚠️ 改 edit bone 前先解開 `use_connect`

相連的子骨，head 綁在父骨的 tail 上。逐骨改座標時 Blender 會同步拖動子骨，
位移互相疊加，結果角色炸成 **14～46 公尺**。

```python
connected = [b.name for b in armature.data.edit_bones if b.use_connect]
for bone in armature.data.edit_bones:
    bone.use_connect = False
for bone in armature.data.edit_bones:
    bone.head = (bone.head - offset) * scale
    bone.tail = (bone.tail - offset) * scale
for name in connected:
    armature.data.edit_bones[name].use_connect = True
```

網格頂點與 edit bone 要套**同一組** `offset` / `scale`，任何一邊漏掉，蒙皮就對不上。

### 3.4 ⚠️ 空物件 apply 不了縮放

`transform_apply` 對 EMPTY 沒有作用。如果 `spirit_root` 帶著 0.01 縮放，
你把 armature 與 mesh 都 apply 成 1 之後再重設 root 為單位矩陣，那 0.01
會被推給子物件——縮放只是搬家，沒有消除。

正確順序：**先解除父子關係 → apply → 最後才重建層級**（此時 root 已是單位矩陣）。

剝皮寮那隻交來時是 `Armature` 100 倍、`Body` 與 `spirit_root` 各 0.01 倍互相抵消。

---

## 4. Blender：節點結構

規格（§7.3.1）要求：

```
spirit_root
├── Armature/
├── Body    (SkinnedMeshRenderer，含 shape key)
├── Hair    (SkinnedMeshRenderer)
└── Eyes    (SkinnedMeshRenderer)
```

三個網格是 **`Armature` 的兄弟，不是它的子物件**，各自獨立物件而不是同一個 mesh
的三個材質槽——因為 F5（uLipSync）與 F8（Animator）綁的是節點路徑字串。

網格不靠父子關係變形，靠 **Armature modifier**：

```python
modifier = piece.modifiers.new("Armature", "ARMATURE")
modifier.object = armature
```

⚠️ **Blender 的 fbx 匯入器會把蒙皮網格重新掛到 armature 底下**。所以拿 Blender
重新匯入自己的 fbx 來檢查結構，看到的會是 `Armature/Body`，那是匯入器的行為，
不代表 fbx 內容錯了。**以 Unity 匯入的結果為準**——實測 Unity 那邊是正確的四項並排。

---

## 5. Blender：材質分離與 shape key

### 5.1 只有一個材質時怎麼切

Meshy 常常整隻只有一個材質，`separate(type="MATERIAL")` 用不上。做法是先依貼圖
顏色把面指派到三個材質槽，再 separate。

- **頭髮**：取樣 basecolor 的亮度，`luma < 0.22` 且屬於 Head 骨骼權重的面
- **髮飾等配件**：用色相判斷（故宮的翡翠是 `g > r + 0.04 且 g > b + 0.04`）
- **眼睛**：先在深色、正面、眼睛高度的面裡找出**左右兩團**（中間會有鼻樑造成的
  空隙），取兩團的中心，再用**距離半徑**選取。單純用 x 範圍會把太陽穴一起吃進去

取樣 8192 貼圖不要整張讀進 Python（8192²×4 個 float ≈ 1 GB）。複製一份縮到 1024
再取樣就夠分類用。

### 5.2 補破洞

每個面只取樣一個 UV 點，髮際線會出現零星的膚色破洞。跑幾輪鄰面多數決即可：
被頭髮包圍的孤立面補成頭髮，四周都不是頭髮的孤立面還原。

### 5.3 shape key

三組硬性要求：`JawOpen`、`Blink_L`、`Blink_R`，**大小寫敏感**。

- `JawOpen`：把唇線以下、屬於 Head 權重的頂點，繞著耳際高度的軸旋轉約 11°，
  用 smoothstep 做衰減，讓接縫柔和。實測位移最大 25 mm
- `Blink_L` / `Blink_R`：眼睛中心半徑內、且高於中心的頂點往下移約 11 mm

⚠️ **角色的左右**：Blender 裡角色面向 −Y 時，它自己的左手在 **+X**。用骨骼位置
確認（`LeftArm` 的 x 是正的），不要憑畫面猜。

眼球已經是獨立物件時，Body 上的 blink 只能帶動眼瞼、蓋不滿眼球——規格本來就把
眨眼的備援放在貼圖切換，灰模階段夠用。

---

## 6. 貼圖

規格：Albedo / Normal / ORM 各一張 **2048×2048**。ORM 的 R = 遮蔽、G = 粗糙度、
B = 金屬度；Meshy 不出遮蔽圖，那個通道填 1.0。

### 6.1 ⚠️ Blender 影像 API 的三個地雷

一次踩齊，順序錯了就會拿到 `does not have any image data`：

| 行為 | 後果 |
|---|---|
| 剛 `load()` 的影像直接 `scale()` | 失敗。要先**完整讀過一次 pixels** 才會真的載入 |
| 存檔前改 `colorspace_settings` | **緩衝作廢**。要在讀取前就設好，或改用新建的影像資料塊 |
| 呼叫 `reload()` | `has_data` 由 True 變 **False**，反而把資料清掉 |

而且法線／粗糙度／金屬度的來源檔常被標成 sRGB，**要在讀取前改成 Non-Color**，
否則讀到的是被線性化過的錯誤數值。

### 6.2 ⚠️ 影像緩衝不會跨 MCP 呼叫存活

透過 blender-mcp 分幾次執行時，上一次縮好的影像到下一次呼叫就沒有資料了。
整批貼圖處理要在**同一次執行**內做完，或乾脆用無頭 Blender 跑獨立腳本。

### 6.3 不要用 `embed_textures`

它會把 blend 裡**所有**被參照的影像都塞進去，包含原始的 8192 那張——實測
fbx 從 2.44 MB 膨脹到 65 MB。貼圖分開交付即可。

---

## 7. 匯出

```python
bpy.ops.export_scene.fbx(
    filepath=path,
    use_selection=True,
    object_types={"EMPTY", "ARMATURE", "MESH"},
    use_mesh_modifiers=False,          # 保留骨架綁定，不要把 modifier 烘進網格
    mesh_smooth_type="FACE",
    add_leaf_bones=False,
    primary_bone_axis="Y", secondary_bone_axis="X",
    bake_anim=True,
    bake_anim_use_all_actions=True,
    bake_anim_use_nla_strips=False,
    bake_anim_simplify_factor=0.0,
    axis_forward="-Z", axis_up="Y",
    path_mode="AUTO",
)
```

⚠️ **`bake_anim_use_all_actions` 只對「已經有 `animation_data` 的物件」生效。**
量測階段如果把 `animation_data` 清掉了忘了補回來，匯出會**一組動作都沒有**，
而且不會有任何錯誤訊息——檔案照樣產出，只是 0.96 MB 而不是 2.44 MB。

### ⚠️ 它也可能少匯出剛好一組——而且通常是 `Idle`

`bake_anim_use_all_actions` 的做法是把每組 action 輪流暫時指定給物件再烘焙。
實測會漏掉**匯出當下已經指定給 Armature 的那一組**。

2026-08-18 的 `ximen_red_house_0818` 就是這樣：`.blend` 裡四組齊全，`Idle` 是
`ASSIGNED ACTION`，匯出的 fbx 只有 `Talking` / `Turn` / `Wave`。沒有錯誤訊息，
檔案大小也看不出少了東西。

**`Idle` 是最容易中招的那一組**，因為它是預設狀態、通常就停在時間軸上。而它偏偏
是最不能少的——Animator 沒有 `Idle` 就沒有預設狀態，角色召喚出來會定在綁定姿勢
不動，`Talking` 播完也沒有東西可以回。

判斷方式：`.blend` 有四組、fbx 只有三組，就是這個。用 `verify_delivery.py` 跑
一次立刻看得出來（`動作組` 那一項會列出缺哪一個）。

修法是重匯出——**不需要重做動作**。匯出前把 `animation_data` 的指定換成別組，
或乾脆先清掉再重新指定一組，讓輪詢不會跟當前狀態撞在一起。

匯出後務必重新匯入驗一次：動作數量、長度、shape key、面數、骨名、身高。
`citysoul-client/Tools/character/verify_delivery.py` 就是在做這件事。

---

## 8. Unity 匯入

### 8.1 ⚠️ 匯出時沒開 `bake_space_transform` 的話，兩邊只能對一邊

**這一節在 2026-08-16 整個改寫過。先前的處方（把 −90 移到 `Armature`）是錯的**
——它解決的是編輯模式的顯示，Humanoid 播放時照樣躺平。完整說明見
`fbx-export-axis-conversion.md`。

Blender 匯出器預設把 Z-up → Y-up 的轉換寫成**根節點的 −90° X 旋轉**，而匯入器把
骨骼的 local transform 寫成剛好補償它的值。Animator 播 Humanoid 動畫時 retarget
會重寫那些 local transform，補償消失，角色躺下。

所以同一個資產：根節點保留 −90 則編輯模式正確、Play Mode 躺平；根節點改成 0 則
反過來。**兩邊都對的唯一方法是匯出時就把轉換烘進資料**：

```python
bpy.ops.export_scene.fbx(..., bake_space_transform=True)
```

**不要用 Unity 匯入器的 `Bake Axis Conversion`**——名字像但位置不同，實測會弄壞
Avatar 的 bind pose，Humanoid 直接扭曲，編輯模式還會變成頭下腳上。

⚠️ 舊參數匯出的角色在遊戲裡看起來正常，是因為 `SpiritPlacement.LateUpdate` 用
`LookRotation` 每幀覆寫根節點旋轉，那 −90 活不過第一幀。**「在遊戲裡好好的」不
代表資產是對的。**

### 8.2 匯入設定

| 欄位 | 值 | 說明 |
|---|---|---|
| `animationType` | `3` | Humanoid |
| `avatarSetup` | `1` | Create From This Model |
| `clipAnimations` | 四組，`name` 用純名稱、`takeName` 用 `Armature|Xxx` | 規格要求 clip 叫 `Idle` 不是 `Armature|Idle` |
| `internalID` | **自己指定固定值** | 見下 |
| `globalScale` | 身高不對時用它修 | 例：交來 1.70 m → `1.6/1.7 = 0.9411765` |
| `bakeAxisConversion` | `0` | 見 §8.1 |

⚠️ **`internalID` 要自己指定。** Animator Controller 是靠 clip 的 fileID 參照
動畫的，留 `0` 的話 Unity 自己生成、外部無法預測，controller 就沒辦法用手寫的方式
接上去。指定一組固定值（例如 `7710001`～`7710004`），Unity 會照用，重新匯入也保留。

### 8.3 ⚠️ 材質要變成明確的資產，不要留給匯入器決定

預設是 `materialLocation: 1`（內嵌材質）＋ `externalObjects: {}`，材質是匯入時
即時生成的。實測**同一個 fbx 在三台機器上得到不同結果**：兩台有顏色、一台沒有。

解法是把貼圖抽成專案資產、建明確的 URP 材質、用 `externalObjects` 重新對應：

```yaml
  externalObjects:
    - first:
        type: UnityEngine:Material
        assembly: UnityEngine.CoreModule
        name: Body_ReferenceConsistency_v1     # ← fbx 裡真正的材質名
      second: {fileID: 2100000, guid: <材質 guid>, type: 2}
```

貼圖內嵌在 fbx 裡的話，用 Blender 開檔後 `image.save()` 抽出來即可。

#### ⚠️ `name` 要以 fbx 實際的材質名為準，不能假設是 Body / Hair / Eyes

上面那個 `name` 是**查找鍵**。對不上就是那一條靜默失效——該材質退回匯入器自動
生成的版本，而那份通常指向一個不存在的貼圖路徑，結果就是「其他部位有顏色、
某一個部位沒有」，**Console 一個字都不會說**。

實際踩過：故宮的新版把 Body 的材質改名成 `Body_ReferenceConsistency_v1`，
`Hair` 與 `Eyes` 維持原名。照慣例寫 `name: Body` 的結果是身體無貼圖、頭髮與
眼睛正常。

先讀出來再寫：

```python
bpy.ops.import_scene.fbx(filepath=path)
for obj in bpy.data.objects:
    if obj.type == "MESH":
        print(obj.name, [m.name for m in obj.data.materials])
```

寫完之後**重新匯入再回讀一次 meta**，確認那幾條還在（Unity 會清掉對不上的
條目），最後用材質資產的 `_BaseMap` 確認它指到的貼圖檔案真的存在。

「缺色」目前遇過三種成因，症狀相似但原因無關：

| 症狀 | 成因 |
|---|---|
| 同一個 fbx，有的機器有色有的沒有 | 材質留給匯入器即時生成，結果依機器而異 |
| 整隻是無貼圖的單色 | fbx 裡的貼圖路徑指向製作者本機，檔案沒隨檔交付 |
| 只有某一個部位沒色 | `externalObjects` 的 `name` 對不上 fbx 的材質名 |

### 8.4 prefab 必須跟 fbx 保持連動

prefab 要是 fbx 的 prefab 實例（檔案裡有 `m_SourcePrefab`），不是解包後的獨立副本。
**不要 Unpack，也不要把場景物件拖回 Project 蓋掉它**——連結一斷，之後重新匯出
fbx 就不會傳到 prefab，而那正好牴觸 §7.3.1「正式角色直接替換、不改程式碼」的前提。

要改 prefab 的內容，用 Inspector 上方的 **Overrides ▾ → Apply All**。

⚠️ Apply 會把**當下實例的所有覆寫**寫回去，包含你為了觀察而挪動的位置。Apply 完
記得檢查 `m_LocalPosition`。

### 8.5 Unity 端該量什麼

不要只看 Avatar 的 `isValid` / `isHuman`——那只驗證骨骼命名對應，不代表播放正常。
量骨骼的**世界座標**最直接：

| 量什麼 | 正常值（1.60 m 角色） |
|---|---|
| `Head` 骨骼 y | 約 1.40～1.45 |
| `HeadTop_End` y | 約 1.59 |
| 根節點 `localRotation` | 單位旋轉（−90 應該在 `Armature` 上） |
| `Animator.isHuman` | true |
| `Animator.hasBoundPlayables` | true（controller 真的解析成功） |
| 各物件 `lossyScale` | 1（不是 0.01 或 100） |

頭在 y ≈ 0 而 z ≈ −1.4，就是躺著。

---

## 9. Animator 與場景接線

### 9.1 Controller

三個狀態就夠起步：`Idle`（預設）→ `Talking`（`Reply` 觸發）→ `Wave`（`Greet` 觸發），
`Talking` / `Wave` 播完自動回 `Idle`。參數用 Trigger。

`Turn` 的 clip 在 fbx 裡，但 F8 的完整定義是四個狀態都要，之後補。

### 9.2 誰去打觸發器（F8）

角色是 `SpiritPlacement` 在**執行期 Instantiate** 出來的，場景裡的 UnityEvent
沒辦法預先指向一個還不存在的物件。所以由角色這側的元件
（`Assets/_Project/Characters/SpiritAnimatorDriver.cs`）在 `OnEnable` 找到場上的
`DialogueController` 並掛上監聽。

依賴方向是「臉」讀「對話」的事件，不是反過來——身體那側不需要知道有動畫這回事，
資料夾分工的邊界保住。

### 9.3 對照表

`SpiritPlacement.spiritModels` 一列一隻，key 是後端的 `spirit_id`。
`spiritPrefab`（預設回退）**建議留空**：漏接的召喚點會出現佔位膠囊，一眼看得出
「這個地標還沒有模型」；填了某一隻的話，會安靜地讓所有地標都出現那一隻。

⚠️ 客戶端的 `SpiritCatalog` 是另一道閘。模型接好、後端也有靈魂，但這張表沒放行的
召喚點，按下去仍然顯示「還沒有靈魂進駐」。

---

## 10. 交付自檢

匯出後重新匯入 fbx 跑一次，逐項對 §7.3.1：

```
□ 節點結構 spirit_root / Armature / Body / Hair / Eyes
□ 所有物件 scale 為 1
□ 身高 1.60 m、腳底 z = 0、xy 置中
□ 四組 clip：Idle / Talking / Wave / Turn，30 fps，Talking ≥ 10 s
□ 各動作全程腳底不離地、漂移小
□ shape key：JawOpen / Blink_L / Blink_R（大小寫一致）
□ 面數 15,000–20,000
□ 材質三個，各自獨立物件
□ 骨名 Mixamo 相容
□ Unity：Humanoid 無警告、isHuman、hasBoundPlayables
□ Unity：實際播 Idle，肉眼確認站姿正常
```

最後一項不能省。前面全過、Idle 一播就塌的情況真的發生過（原因見 §3.4 與 §8.1）。

---

## 11. 三隻角色的實測對照

| | `spirit_palace_v0` | `bopiliao_v0`（交來時） | `spirit_ximen_v0`（交來時） |
|---|---|---|---|
| 面數 | 14,663 | 12,835 | 15,532 |
| 身高 | 1.60（處理後） | 1.58 | 1.70（用 Scale Factor 修） |
| 物件縮放 | 1 | **Armature 100、mesh 0.01** | **Armature 0.01** |
| 節點結構 | 四項並排 | **Body 掛在 Armature 下** | 四項並排 |
| 材質 | 3 個獨立物件 | 1 網格 2 材質槽 | 3 個獨立物件 |
| shape key | 三組齊 | **無** | 只有 `JawOpen` |
| 根節點旋轉 | **−90（要搬到 Armature）** | 0 | 0 |
| 貼圖 | 自製 2048 三張 | **指向製作者本機路徑，遺失** | 內嵌 2048 Albedo |
| `Talking` | 3.77 s → 接成 11.30 s | 5.17 s | 10.33 s |

**殘留縮放是最常見的問題，也是塌陷的主因。** 剝皮寮那隻的 README 把症狀記成
「Humanoid retarget 壞掉、原因未知、跨角色的系統性問題」，實際上是那組
100 倍 / 0.01 倍互相抵消的縮放——apply 進資料層之後，模型立刻正常站立。

---

## 12. 還沒解決的

- **面數不足**要重新生成模型，不是後製能補的（剝皮寮 12,835 < 15,000）
- **貼圖遺失**只能跟原作者要（fbx 裡存的是製作者本機的絕對路徑）
- **Normal / ORM**：Meshy 出的是分離的 metallic / roughness / normal，可以打包；
  完全沒出的話灰模階段「不論貼圖」，正式資產要補
- **`Turn` 狀態**還沒進 Animator 狀態機
- **口部與眨眼的拓樸精修**屬於美術工作，不在這條產線裡
