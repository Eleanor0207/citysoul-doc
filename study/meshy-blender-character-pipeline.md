# Meshy → Blender → Unity 角色產線 SOP

> 依 `bopiliao_v0`（剝皮寮靈魂）實際跑過一次的流程整理，給每個人各自負責的角色參考。
> 標「⚠️」的地方是實際踩過的坑，照著做可以直接避開，不用重踩一次。
> 對應規格見 `citysoul-doc` repo 的 `SDD_v2.1_Unity_3D.md` §7.3 / §7.3.1，本文件只講
> 「怎麼做」，規格數字以 SDD 為準（SDD 有異動時，以 SDD 為主，這份文件可能沒同步更新）。

---

## 0. 整體流程

```
GPT 生成參考圖 → Meshy Image to 3D → Meshy 動作庫 → Blender 整理 → 匯出單一 .fbx → Unity 驗證
```

---

## 1. Meshy 生成模型

- **模型類型選「智慧拓撲」，不要選「標準」**。標準模式會生出幾百萬面的原始掃描網格，是設計給你自己手動 retopology 用的，不是能直接用的最終模型；智慧拓撲才是面數合理（1~2 萬）、適合綁骨的版本。
- 面數設定在規格範圍內（見 SDD §7.3.1）。
- 生完網格後另外點「貼圖」按鈕生成材質，不要只看沒貼圖的灰模階段判斷好壞——沒貼圖時五官/頭髮看起來都會很粗糙，貼圖上色後才是真實效果。

## 2. Meshy 動作庫

- 用「骨骼綁定」功能自動綁骨（不用手動上傳去 Mixamo）。
- 動作庫搜尋時，找 **「待機」「說話」** 這類關鍵字，**不要搜「走路」**——NPC 站立對話用不到走路循環，硬套的話 Idle 會變成一直在走位。
- 選好幾個動作後（右上角打勾加進「已新增」清單），下載時選 **「全部已新增」**，會直接打包成一個含骨架+多動作的 fbx，不用分開存四次再自己合併。
- 每秒幀數選 **30**（跟 Mixamo 動作庫預設一致，避免混用時速度對不起來）。

## 3. Blender：清理匯入

1. 匯入 Meshy 下載的合併動作 fbx
2. 動作命名很亂（例如 `Idle_11`、`Stand_and_Chat`），改名成標準的 `Idle` / `Talking` / `Wave` / `Turn`
3. 刪掉不需要的多餘動作（走路、跑步之類的）

## 4. Blender：身高校正 ⚠️ 這裡最容易出錯

**目標身高見 SDD §7.3.1（1.60m）。**

- ❌ **不要直接覆蓋 `armature.scale`**（例如 `armature.scale = (0.94, 0.94, 0.94)`）。Meshy 匯出的骨架物件本身通常已經帶了一個很小的縮放值（例如 0.01，是匯出時的單位轉換），直接覆蓋會蓋掉這個既有縮放，結果差 100 倍。
- ✅ **正確做法：用「乘」不是「蓋」**——`new_scale = armature.scale * factor`，用目標身高除以目前身高算出 `factor`。
- **改完一定要 Apply Transform（Scale）**，把縮放值烘焙進資料本身，不要留著非 1 的 object scale。骨架跟網格（mesh）都要各自 apply，兩個分開處理，不要假設其中一個 apply 了另一個就自動對。
- Apply 完，`armature.scale` 跟 `mesh.scale` 應該都回到 `(1, 1, 1)`，這是檢查有沒有做對的簡單方法。

## 5. Blender：spirit_root 節點結構 ⚠️ 重要更正

**規格要求（SDD §7.3.1）**：

```
spirit_root
├── Armature/
├── Body    (SkinnedMeshRenderer)
├── Hair    (SkinnedMeshRenderer)
└── Eyes    (SkinnedMeshRenderer)
```

**`Body`／`Hair`／`Eyes` 必須是三個獨立的物件（各自有自己的 SkinnedMeshRenderer），不是同一個 mesh 裡的三個材質槽。** 這點很容易做錯（材質槽比較好做，物件分離要多一步），但規格明確要求要獨立物件，原因是 **F5（uLipSync）跟 F8（Animator）是靠這個節點路徑字串去綁定的**，材質槽沒辦法提供這種路徑。

正確做法：
1. 網格分材質區域（可以用貼圖顏色/UV 分析抓範圍，或手動框選）
2. 用 `bpy.ops.mesh.separate(type="MATERIAL")`（或 Edit Mode 手動 P 鍵分離）把 Hair、Eyes 各自分離成獨立物件
3. 分離出來的新物件記得**重新綁定骨架**（Parent → Armature Deform，或確認 Armature modifier 有正確指到同一副骨架），並改名成 `Hair`、`Eyes`
4. 三個物件都要掛在 `spirit_root` 底下（跟 `Armature` 同一層），不是互相巢狀

**⚠️ 重新掛父層（parent）時的地雷**：如果物件之前掛在別的父物件底下（例如 mesh 原本掛在 Armature 底下），改掛到 `spirit_root` 之前，一定要先把 `matrix_parent_inverse` 重置成單位矩陣（`mathutils.Matrix.Identity(4)`），否則舊的父子關係殘留資料會讓縮放莫名其妙跑掉，且不會馬上發現（要等切換時間軸幀數或重新整理才會爆出來）。用 Blender 內建的 `parent_set(type='OBJECT', keep_transform=True)` 操作比手動兜矩陣可靠。

## 6. Blender：Shape Key

- `JawOpen` 是硬性要求（見 SDD §7.3.1），`Blink_L`／`Blink_R` 也要，命名大小寫要完全一致。
- 建議用手動在 Blender 視窗裡選取頂點、Blend From Shape 重置＋位移的方式做，比程式碼猜座標範圍準。範圍抓太大容易牽動不該連動的部位（例如眼皮選到眉毛、下巴選到整個嘴巴附近五官）。

## 7. ⚠️ 動作烘焙了物件層級動畫的地雷（很隱蔽，一定要檢查）

Meshy 匯出的動作 fbx，**物件層級（不是骨頭）可能被烘焙了 location / rotation / scale 關鍵幀**。症狀：Blender 裡明明已經把 `armature.scale` 改成 1 並 apply 過了，**只要切換時間軸幀數（或進 Unity Play Mode 觸發動畫播放），縮放值又會跳回原本的錯誤數值**，怎麼修都「修不好」。

**檢查方法**（Blender 5.x 新版 Action API）：
```python
action = bpy.data.actions.get("Wave")
for layer in action.layers:
    for strip in layer.strips:
        for slot in action.slots:
            cb = strip.channelbag(slot)
            if cb:
                for fc in cb.fcurves:
                    if fc.data_path in ("location", "rotation_euler", "scale"):
                        print(fc.data_path, fc.array_index)  # 有印出東西就代表中招
```

**修法**：把這幾個物件層級的 fcurve 從**每一組動作**裡刪掉（`cb.fcurves.remove(fc)`），只留骨頭本身（`pose.bones.*`）的動畫。改完後在時間軸切幾個不同幀數確認 `armature.scale` 都穩定維持 `(1,1,1)`，不會再跳動，才能確定真的修好了。

## 8. 匯出

- 全選 `spirit_root` 底下所有物件（含子物件）
- 匯出單一 `.fbx`，勾選 `bake_anim` 跟 `bake_anim_use_all_actions`（把所有動作都包進同一個檔案，符合 SDD §7.3「格式：.fbx（含骨架、動畫、shape key）」的要求）

## 9. Unity 端驗證

1. Import 後 Rig 分頁選 **Humanoid**，Apply
2. **不要只看 Avatar 是否 `isValid`/`isHuman` 就當作過關**——這只驗證了骨骼命名對應，不代表動畫播放正常。**一定要實際進 Play Mode 播放 Idle 動畫，肉眼確認角色站姿正常、沒有扭曲塌陷**，這個問題在灰模階段的靜態檢查完全查不出來。
3. ⚠️ **Root Motion 地雷**：如果動畫一播放角色就瞬間移動到很遠的座標，去 Model Importer 的 Animation 分頁，每個 clip 都把 `Root Transform Rotation` / `Root Transform Position (Y)` / `(XZ)` 設成 **Bake Into Pose**（對應程式碼是 `lockRootRotation` / `lockRootHeightY` / `lockRootPositionXZ` 設 `true`），避免動畫自帶的根位移把角色物件本身的座標推走。

## 10. 已知未解決問題（跨角色都出現，優先查）

**Humanoid retarget 播放 Idle 動畫時角色會扭曲塌陷成一團**，不是正常站姿。兩個不同角色（Mixamo 來源、Meshy 來源）都出現一樣的症狀，代表這很可能是產線本身的系統性問題，不是單一角色的意外。骨骼命名對應、Avatar 有效性都檢查過沒問題，問題出在更底層的 retarget 計算。

排查建議（還沒驗證，下次先查）：
1. 把 Animation Type 從 Humanoid 改成 **Generic**，看播放是否正常，藉此判斷問題是不是真的出在 Humanoid retarget 這一層
2. 檢查 Avatar 的 T-pose 參考姿勢跟 mesh 實際 bind pose 是否一致
3. 檢查 Blender 匯出時骨架的 rest pose 有沒有跟 mesh 的 skin binding 對齊

如果你的角色也遇到同樣的塌陷問題，不用重新排查，直接在 issue/群組裡說一聲，一起查會比較快找到根因。
