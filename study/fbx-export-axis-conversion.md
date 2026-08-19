# 匯出 FBX 一定要開 `bake_space_transform`

> 2026-08-16。這一條是 `meshy-to-unity-character-pipeline.md` §7 與 §8.1 的更正，
> 起因是 `spirit_bopiliao_v0` 的「Humanoid 播放塌陷」（見
> `bopiliao-humanoid-collapse-fix.md`）。追到最後發現那不是單一角色的問題，是
> **每一隻用舊參數匯出的角色都有**，只是被別的東西掩蓋著。

## 一句話

匯出 FBX 時加 `bake_space_transform=True`。不加的話，角色在編輯模式與 Play Mode
只能有一邊是站的。

## 症狀

不加這個參數，會遇到下面其中一種，取決於 prefab 根節點的旋轉怎麼設：

| 根節點旋轉 | 編輯模式 | Play Mode（Humanoid） |
|---|---|---|
| 保留匯入的 −90° | 站立 | **躺平** |
| 手動改成 0 | **躺平、腳陷進地板** | 站立 |

兩邊都對是做不到的——因為「那個 −90 該不該存在」，兩邊的答案相反。

## 為什麼

1. Blender 是 Z-up、Unity 是 Y-up。匯出器預設把這個轉換寫成**根節點的一個
   −90° X 旋轉**，網格與骨架的資料本身維持 Z-up
2. 匯入器同時把骨骼的 local transform 寫成**剛好補償那個 −90** 的值，所以靜態
   看起來是對的
3. Animator 播 Humanoid 動畫時，retarget 以「根節點是正的」為前提**重寫所有骨骼
   的 local transform**，那份補償被覆蓋
4. −90 現形，角色躺下

`bake_space_transform=True` 把轉換烘進**資料本身**，節點全部是單位旋轉，骨骼
local transform 不必補償任何東西，被 retarget 覆寫也不會有東西消失。

## 正確的匯出參數

```python
bpy.ops.export_scene.fbx(
    filepath=path,
    use_selection=True,
    apply_scale_options="FBX_SCALE_ALL",
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
    bake_space_transform=True,         # ← 這一行
)
```

Blender 介面上它叫 **「!EXPERIMENTAL! Apply Transform」**，在匯出對話框的
Transform 區塊。名字上的 experimental 標記從 2.8 掛到現在，對這個用途是穩定的。

## ⚠️ 不要跟 Unity 的 `Bake Axis Conversion` 搞混

那是**匯入端**的選項，用途看起來一樣，但實測會把轉換烘進網格與骨架資料的同時
改掉 Avatar 參考的 bind pose，**Humanoid 直接扭曲**，而且編輯模式會變成頭下腳上。

| | 位置 | 結果 |
|---|---|---|
| `bake_space_transform` | Blender 匯出器 | 正解 |
| `Bake Axis Conversion` | Unity 匯入器 | 弄壞 Humanoid，不要開 |

## 驗證方法

量骨骼世界座標，不要只看畫面。**編輯模式與 Play Mode 都要量**——這個 bug 的
特徵就是只有一邊會露出來。

| 檢查 | 正常值（1.60 m 角色） |
|---|---|
| prefab 根節點 `localRotation` | 單位旋轉（w=1, x=y=z=0） |
| `Head` 世界 y（編輯模式） | ≈ +1.44（相對根節點） |
| `Head` 世界 y（Play Mode 播 `Idle`） | ≈ +1.44 |
| `LeftToeBase` 世界 y | ≈ +0.03（腳趾骨在腳掌內，略高於地面是正常的） |

躺平的特徵是 `Head` 的 y ≈ 0 而 |z| ≈ 身高。

`spirit_palace_v0` 用新參數重匯出後的實測：編輯模式 `Head` y = 1.4416、
Play Mode y = 1.4438、`LeftToeBase` y = +0.035。

## 為什麼以前沒發現

`SpiritPlacement.LateUpdate` 每幀執行：

```csharp
_spirit.rotation = Quaternion.LookRotation(toPlayer);   // 讓角色面向玩家
```

它整個換掉根節點的旋轉，那 −90 活不過第一幀。**所以透過 `SpiritPlacement` 生成的
角色在遊戲裡是站的**，症狀只有在把 prefab 直接拖進場景測試時才會出現。

也就是說：**「在遊戲裡好好的」不代表資產是對的**，只代表執行期有東西蓋掉了它。

## 現況

| 角色 | 狀態 |
|---|---|
| `spirit_palace_v0` | 已用新參數重匯出並驗證 |
| `spirit_ximen_v0` | 舊參數。靠 `LookRotation` 才站得住，請作者用新參數重匯出 |
| `spirit_bopiliao_v0` | 同上 |

拿 fbx 往返轉一手也能修，但 shape key 與動作要重新驗一次，不如請作者從自己的
`.blend` 重匯出乾淨。
