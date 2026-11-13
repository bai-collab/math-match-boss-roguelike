# 03｜數字盤生成、合法路徑搜尋與無解救援

> 這是本遊戲重要的**公平性與可測試性**需求。單純隨機填滿 6×6 盤面不能保證有怪物弱點可打。

## 1. 預設盤面

- 6 列 × 6 行；每格 `{id,color,value}`，顏色三種，數字 1..9。
- 玩家僅能選取相同顏色、上下左右連續、長度 3..6 的路徑。
- 每次消除後，空位向下掉落，上方補新珠；補珠的 RNG state 記在 `RunState` 中，以便回放。
- **預設公平性保證**：戰鬥開始時，以及每次有效消除與補盤後，保證至少存在一條可令當前 `W` 精確命中的**紅色攻擊路徑**。其他顏色的弱點命中路徑可以但不保證存在；至少保證普通合法行動可走。
- 若弱點即將切換，應先生成／檢查下一個弱點的解；不能在 UI 承諾弱點後突然更改規則。

## 2. DFS / 回溯尋路

```text
findWeaknessPaths(board, target, color='red', minLength=3, maxLength=6, limit=20):
  solutions = []
  for each tile in row-major order:
    if tile.color != color: continue
    DFS(start=tile, path=[start], sum=tile.value, visited={start})

  DFS(last,path,sum,visited):
    if path.length>=minLength and sum==target:
      solutions.push(copy(path))
      if solutions.length>=limit: return STOP
    if path.length==maxLength or sum>=target: return CONTINUE
      # 此剪枝僅適用珠子 value 均為正整數；若未來加入 0 或負數須修改
    for each neighbor in fixed order [UP, RIGHT, DOWN, LEFT]:
      if inside && neighbor not visited && neighbor.color == color:
        DFS(neighbor,path+[neighbor],sum+neighbor.value,visited+neighbor)
  return solutions
```

### 可達與可用性的區別

- **數字可分解**（`2+3+5=10`）不等於**盤面可連線**。必須檢查同色、座標、相鄰、使用次數與長度。
- 若 Boss 階段要求至少 4 顆，solver 同時檢查 `minLength=4`，不能只看總和。
- 多個弱點 `targets=[12,18]` 可搜尋任一；對旋轉弱點仍需檢查當前真正生效的 target。

## 3. Seeded RNG、盤面產生與救援

- 使用明確的 32-bit deterministic RNG（例 Mulberry32，與 `pyes_Composition_game` 類似）；**不能依賴 `Math.random`** 產生可重播的關鍵遊戲狀態。
- `generateSolvableBoard(seed, weakness)` 最多嘗試 `N=100` 次隨機盤面（初始參數），每次用完整 path solver 檢查。
- 若 100 次仍沒有可用路徑，**確定性地植入一條合法路徑**：先從 `target` 分解為 3～6 個 1～9 的數字（符合弱點的最少連線數），再選擇預先驗證的、完全在盤內且不重複的連通座標並配置同色數字。其餘珠子由同一 RNG 填補。
- 若目標完全不能以允許的 1～9 × 3～6 珠子表示（例如 target 超出 3～54），應在**怪物配置驗證**階段拒絕該敵人；不可進戰鬥後才偷偷修改弱點。
- 若是 Boss 階段的 `minChain=4`，其有效目標範圍為 4～54；但仍必須跑實際 decomposition 搜尋，不能只檢查範圍。
- 如果遊戲中盤面被技能改動到無解，優先 deterministic reshuffle，顯示「盤面重整」；重整不消耗玩家行動、不產生戰鬥收益、不計入數學作答。

## 4. 不讓玩家「自動知道答案」

- solver 是**內部可解性保障**，預設不顯示實際座標。
- 只有玩家啟用提示時，才按照提示階層逐步揭露：弱點差距 → 可行數字組數 → 路徑第一顆 → 完整路徑（如果老師允許）。
- 系統記錄提示使用量、類型；不得把使用提示當成天生能力低落。

## 5. 重要屬性測試

1. `validatePath` 接受 solver 輸出的所有路徑。
2. `evaluatePath` 的 `sum` 等於路徑所有 tile 的值，精確符合 target。
3. 針對 1,000 個 seeds × 各階段 weakness，`generateSolvableBoard()` 有解且時間在可接受門檻（建議在 CI 紀錄 P95，正式閾值需實測後決定）。
4. 同一 seed、同一設定產生完全相同盤面、弱點、掉落結果。
5. 任意合法路徑消除後再次檢查有解；無解時重整結果仍可解且不扣行動。
6. 不會出現 `NaN`、0、10、負數或非整數（若未來加道具，另行制定範圍）。
7. 不會出現無限迴圈：重試與植入路徑都有明確上限，不能用 `while(!solvable)` 無界重試。
8. 不洩漏未經玩家操作就已算出的位置；無解救援不寫成學生成功作答。

## 6. 風險與取捨

**保證每回合有紅色精確解**會增加連續命中率，可能使遊戲過易；但先確保不會卡死更重要。等 MVP 可測後，可 A/B 比較「每回合保證精準紅珠路徑」「僅保證任意同色合法路徑並提供重整」兩種盤面政策。不得未測即聲稱哪種提高學習。
