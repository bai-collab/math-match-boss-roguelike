# 02｜數學規則：唯一正確性判定與傷害公式

> 本文 `weakpointMultiplier` 與 `lengthMultiplier` 是**待平衡的建議起始值**。精確計算應用整數驗證，不得使用 LLM 判定數學正確與否。

## 1. 數學操作定義

- 盤面 `board[r][c] = { id, color: 'red'|'blue'|'green', value: integer 1..9 }`。
- 已選取合法路徑 `p=[(r0,c0),...,(rn-1,cn-1)]`。
- 路徑長度：`n = p.length`；目前預設 `3 ≤ n ≤ 6`。
- **路徑總和**：`S = Σ board[p[i]].value`。
- 敵人弱點目標：`W`，例 8、12、17、22；不同怪物可不同。Boss 同時有階段門檻與至少幾消的要求。
- 是否精確命中：`exact = (S === W)`；是否接近目標：`near = (abs(S-W) === 1)`。
- 路徑必須先經過合法性驗證，無效路徑不能結算，不能扣行動，不能變動 RNG。

## 2. 倍率函式（可調預設）

弱點：

```
B(S,W) = 3      if S == W
       = 1.25   if |S-W| == 1
       = 1      otherwise
```

連線：

```
L(n) = 1 + 0.25 * min(max(n - 3, 0), 3)
// n=3→1, n=4→1.25, n=5→1.5, n=6→1.75
```

本次原始效果：

```
P = floor(basePowerPerTile[color] * n * B(S,W) * L(n))
```

初版 `basePowerPerTile = {red:2,blue:2,green:2}`。每條路徑結算一次，各色使用相同的總和和倍率判定，但只作用於該色的攻擊／防禦／治療。

### 實例

- 紅 `3+5+9=17`、敵人弱點 `W=17`、三消：`floor(2×3×3×1)=18` 傷害。
- 紅 `2+3+5+7=17`、四消：`floor(2×4×3×1.25)=30` 傷害。
- 藍 `4+6+8=18`、弱點 `W=17`、三消：`floor(2×3×1.25×1)=7` 護盾。
- 綠 `1+1+1=3`、弱點 `W=17`、三消：`floor(2×3×1×1)=6` 治療（最多到生命上限）。

**注意：** 遊戲內如有 `1.25`、`1.5`，可用整數分子（例如 `125/100`）運算，最後統一向下取整，避免多步浮點取整不一致。UI 顯示與結算必須共用同一個 `calculateEffect()` 結果。

## 3. Boss 破防與倍率是兩件事

「達到 Boss 階段條件」≠「一般傷害倍率」。例如第二階段需要 `n>=4 && S==18`，只要符合才 `breakSeal`，然後進入下一血條階段；不符合可以做出普通傷害，但**不能將該階段 HP 降到破防門檻以下**。詳見 `05_COMBAT_BOSSES.md`。

## 4. 特殊怪規則

- `sum_equals(target)`：精確總和。
- `min_chain_count(k)`：至少 k 顆；與總和需同時成立才能破甲。
- `multi_targets([12,18])`：命中任一目標皆精確命中；`B` 的輸入以最近差距判定，若多個完全相同的弱點只算一次。
- `rotating_targets([12,17,22])`：每個**完整怪物回合**切換，必須在怪物行動前公告下一個目標。
- `resistance('red')` 等可作未來選配，不能隱性降低數學正確的回饋。

## 5. 何時允許數字 10／10 倍？

**原始合十十倍是已被新想法取代的舊核心，不是現在的全域規則。** 可以在後期作為限次技能，例如 `DecimalBurst` 遺物：當 `S===10` 且使用者完成指定條件時，這一次**額外觸發**，但必須定義與弱點倍率是相乘、擇高還是封頂；第一版**不實作此效果**，避免影響基準測試。

## 6. 數學透明性與教學

- 每次有效消除都可回顯 `a+b+c=總和`、`弱點 W`、命中狀態與結算值。
- 不用「你算錯了」批評玩家：若路徑合法但總和不符弱點，顯示「本次合計 15，怪物弱點 17，仍施放一般效果」。
- 不強制數學題彈窗；練功房可暫停戰鬥讓玩家先猜測、驗算，再連線。
- 既要鼓勵精準，也不要鼓勵無成本猜解：有效消除耗行動；錯誤路徑操作則只是無效，不扣行動。

## 7. 程式函式契約（TypeScript 偽介面）

```ts
type Color = 'red' | 'blue' | 'green';
type Point = { row:number; col:number };
type Tile = { id:string; color:Color; value:number };
type Board = (Tile | null)[][];
type Weakness = { targets:number[]; minChain:number; requireExactToBreak:boolean };

type PathEvaluation = {
  legal:boolean; reason?:string; values:number[]; sum:number;
  nearestTarget:number; exact:boolean; near:boolean;
  weakpointMultiplierNumerator:number; // denominator = 100
  lengthMultiplierNumerator:number;    // denominator = 100
  basePower:number; rawPower:number; appliedPower:number;
};

function validatePath(board:Board,path:Point[]): {ok:boolean;reason?:string};
function evaluatePath(board:Board,path:Point[],weakness:Weakness): PathEvaluation;
function calculateEffect(evaluation:PathEvaluation, actor:{power:Record<Color,number>}): number;
```

**必要測試：** 同一座標重用、不同色、斜向、盤外、2 顆不足、7 顆超限、輸入非法數字、精確／差一／一般、連線長度、目標重複、Boss 破防與取整一致性。
