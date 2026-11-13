# 05｜怪物、弱點、回合制戰鬥、Boss 階段

## 1. 戰鬥狀態

- `player`: `hp`, `maxHp`, `shield`, `maxShield`, `actionsRemaining`, `relics`, `temporaryEffects`。
- `enemy`: `id`, `name`, `hp`, `maxHp`, `attackIntent`, `weakness`, `phase`, `sealBroken`。
- `board`: 6×6 Tile 與隨機種子狀態。
- `turn`: 玩家第幾回合，怪物有無行動，當回合成功消除次數。
- `log`: 事件紀錄（行為、數字總和、倍率、護盾、治療、弱點命中、提示使用）。

## 2. 回合流程（初始提案）

```text
PLAYER_TURN_START: actionsRemaining = 2; showEnemyIntent()
PLAYER_ACTION:
  selectPath -> validate -> preview -> commit
  compute exact math -> effect -> apply -> drop -> ensureSolvable
  if boss seal condition met: breakSeal/advancePhase
  if win: REWARD or RUN_RESULT
  else actionsRemaining--
END_PLAYER_TURN:
  if still alive: enemyActs(telegraphedIntent)
  absorb by shield -> apply remaining to HP -> death check
  announce next weakness (if rotates) + next intent
  ensureSolvable(targetNow)
  go to PLAYER_TURN_START
```

- 每次 action 必須**原子結算**。动画和音訊事件不能反過來改戰鬥結果。
- 無效拖曳不耗行動、不改盤面、不改 RNG。
- 補血不能超過 max HP；護盾不能超過 cap；數值顯示與遊戲計算共用同一筆事件。

## 3. 敵人家族（初始示意）

| 敵人 | 弱點範例 | 特殊要求 | 教學意義 |
|---|---|---|---|
| Slime | 8 | `sum_equals(8)` | 加法、拆數 |
| Skeleton | 12 | `sum_equals(12)` | 多種組合 |
| Goblin | 17 | `sum_equals(17)` | 三消與四消權衡 |
| Iron Guard | 18 | `sum_equals(18)` 且 `n>=4` 才破甲 | 組合數字／規劃路徑 |
| Twin Core | 12 / 18 | 命中任一弱點可破甲 | 比較可行解 |
| Clock Mage | 12→17→22 | 每個敵人回合切換，事前公告 | 動態策略 |
| Gear Golem | 22 | 累積弱點命中兩次破甲 | 多回合資源規劃 |

**怪物名稱、弱點與數值皆為設計提案；不是從參考 repo 複製的既存敵人。**

## 4. Boss 初版：數字魔龍（三階段）

- **第 1 階段**：`W=12`, `minChain=3`。
- **第 2 階段**：`W=18`, `minChain=4`。
- **第 3 階段**：`W=25`, `minChain=3`，可加入額外攻擊意圖；仍必須先顯示限制。
- 每階段弱點**精確命中且符合最少連線數**時，才破除該階封印。
- 每階段採**獨立 HP 條**較直觀；封印未破時，任何普通傷害不可使該階段 HP 低於 `1`，避免靠純攻擊跳關；命中後使封印破裂，然後可進下一階（具體先結算再切階的順序需於測試固定）。
- 每階段可先採 `hp=[30,45,60]`、`attackIntent=[5,7,9]`、最大兩次玩家行動；**這些全是未平衡的測試值**，留在 config 而非硬編碼。
- 升級攻擊可以加快普通怪進度，但**不准**在沒有完成封印條件時直接清掉 Boss 階段。

## 5. 封印與一般傷害分開

建議 `PhaseState = {hp,maxHp,sealBroken,weakness}`：

1. 路徑有效先計算 `sum`, `n`, `power` 與 `isWeaknessSolved`。
2. 如果 `isWeaknessSolved` 且顏色為紅色（預設），設 `sealBroken=true`，套用攻擊，若本階段血條歸零才進下一階；**防禦／治療即使對準弱點，也不能直接打破封印**（但可獲得共鳴倍率）。
3. 如果未破封印，紅色傷害最多把當前血條降至 1；不能觸發下一階段。
4. Boss 最後階段同時 `sealBroken` 且 HP=0 才能勝利。

> 裁決補充：先前構想只說「破解弱點才能進階」，此處將「紅珠破封印」設定為可調預設，以免藍／綠消除無意間通關。

## 6. 怪物行動系統

- 每隻怪物的下一回合行為應可預告：攻擊 X、護甲 X、變更弱點、增傷等。
- 怪物意圖以資料表定義，不能讓 AI 即時任意改玩家挑戰條件。
- 初版採 deterministic intent sequence，預期結果可測；之後可加以 seed 取樣的招式池。
- 戰鬥難度的來源應是可見的條件與取捨，而非突然隱藏的數值跳動。

## 7. 難度與提示

- 普通怪 W 通常取易於三消的可表示範圍；菁英可要求四消或雙弱點。
- 暫時不要根據學生身份自動貼標籤；可以依當局進度調整提示與怪物攻擊，但要可關閉與可解釋。
- 任何怪物只要標上弱點，在目前盤面必須有條件可達的紅珠精確解，或提供無成本救援；不能靠運氣決定學生是否能答對。
