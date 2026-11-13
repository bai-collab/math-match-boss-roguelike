# 04｜隨機爬塔地圖：10 層、六角路徑、可重播

## 1. 直接參考的既有實作

來源：[`bai-collab/pyes_Composition_game/game/index.html`](https://github.com/bai-collab/pyes_Composition_game/blob/main/game/index.html) 中 `randomFrom(seed)`、`generateMap(seed)`、`startRun(...)`，以及 [`spec.md`](https://github.com/bai-collab/pyes_Composition_game/blob/main/spec.md) 的「爬塔、成長與經濟」。已實際閱讀這些內容。

其核心設計：第 0～8 層在 0..4 欄六角格生成每層 2～5 個節點（起點 2～4），單數層位移半格；按欄位相鄰規則連到下一層；第 9 層整排 Boss；用 seed RNG 保持可重播。這是**參考架構**，不是要求複製原始程式碼。

## 2. 數學遊戲初版地圖（提案）

| 層 | 節點 | 特殊要求 |
|---|---|---|
| 0 | 2～4 普通怪 | 提供基本入門，弱點數字較小 |
| 1 | 2～5 混合節點 | 普通怪／營火／事件 |
| 2～4 | 2～5 混合節點 | 至少在生成地圖中有營火；部分路線可以選擇避戰 |
| 3～5 | 2～5 混合節點 | 至少在地圖中有商店 |
| 4～6 | 2～5 混合節點 | 至少在地圖中有菁英 |
| 7～8 | 2～5 混合節點 | 高風險／補給選擇 |
| 9 | 1 個 Boss | 必須走完各破防階段 |

初版一般節點權重（**非已實測最佳**）：普通怪 43%、營火 17%、商店 14%、菁英 12%、事件 9%、數學試煉 5%。若改變權重，保持在設定檔而非寫死在 UI。

### 節點定義

- `monster`：一般戰鬥，獲得局內金幣／遺物選項。
- `elite`：有額外弱點限制的菁英，風險高、獎勵較好。
- `camp`：回復生命、重整道具或選一項短期強化。
- `shop`：花本局金幣購買遺物、補血或重排技能。
- `event`：故事或非戰鬥二選一（初版可只做簡單事件）。
- `math_trial`：非戰鬥限次目標總和挑戰；提供數學資源或局內獎勵。
- `boss`：本局最後的多階段數學怪物。

## 3. Map Generator 明確規約

```text
function generateMap(seed,config): MapLayers
  rng = seeded(seed)
  layers[0] = chooseUniqueColumns(0..4, count 2..4)
  for f in 1..8:
      # 確保每一個新節點至少存在一個前一層的可到達前驅
      # 每一個既有節點至少有一條下一層連線
      columns[f] = chooseReachableUniqueColumns(columns[f-1], count 2..5)
  layers[9] = [bossNode]
  for f in 0..8:
      connectOnlyAdjacentHexColumns(layers[f], layers[f+1])
  assignNodeTypesWithSeed()
  enforceGuaranteedNodePresence(camp floors 2..4, shop 3..5, elite 4..6)
  assertAllNodesReachableFromSomeStart()
  assertEveryStartHasPathToBoss()
  return layers
```

注意 `有節點` 不等於 `玩家任意選的路線必定經過該類節點`；如果要保證每位玩家一定走到營火，需再做路線覆蓋驗證，不要混淆。初版僅保證地圖中出現即可。

## 4. 地圖狀態

```ts
type NodeType='monster'|'elite'|'camp'|'shop'|'event'|'math_trial'|'boss';
type MapNode={id:string;floor:number;col:number;type:NodeType;targets:string[]};
type MapLayers=MapNode[][];
type MapProgress={seed:number;currentNodeId:string|null;availableIds:string[];visitedIds:string[]};
```

- 開局 `availableIds` 是所有第 0 層節點。
- 只能點 `availableIds` 中的節點；不能跳層／選未連通的路。
- 節點完成後將 `visitedIds` +1，`availableIds` 改為其 `targets`。
- 畫面顯示已走路線、可走節點、不可走暗色節點；節點資訊不應暗示其實不存在的可達分支。
- 存檔儲存完整 `map`、`seed`、`current`、`visited`、`available`、玩家資源和 RNG state。

## 5. 地圖測試

- 相同 seed 地圖完全一致；不同 seed **通常**有差異（非保證所有兩種 seed 都不同）。
- 每層節點合法、id 唯一，沒有環、沒有跨兩層邊。
- 每個非 Boss 可達節點至少有一條通往下一層的路；第 9 層一定是 Boss。
- 不存在從未到達的節點直接開戰或一格重複領獎。
- 保證指定區間中至少存在一個 `camp/shop/elite`（不是每條路線保證）。
- 地圖 seed 驗證至少 1,000 次，自動測試每局可以由某個初始節點抵達 Boss。
- 存檔恢復後剩餘節點與重載前完全一致。

## 6. 與作文遊戲的區別

僅借鏡路線拓樸與隨機狀態模型。數學遊戲的每個 `monster/elite` 節點必須綁定 `weaknessProfile` 與 `mathDifficulty`，Boss 有自己的三階段弱點；不使用作文段落、句子牌、文體與墨水系統名稱。
