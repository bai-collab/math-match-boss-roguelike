# AGENTS.md｜給 Codex、Claude Code 與其他 AI 開發者

本檔為開發工作入口。若你是新接手的 AI：**不要直接跳過需求比對就開始產生遊戲程式**。

## 0. 狀態與優先順序

- Repo 初始狀態：**只有計畫文件，不代表遊戲已建立或測試通過**。
- 使用者明確意圖優先，其次是本檔、`docs/13_DECISIONS_OPEN_QUESTIONS.md` 的已確認決議、其餘設計文件、範例設定檔。
- 發現文件不一致時：先記錄衝突、選擇最不會破壞核心需求的最小方案，並將決策寫入 `docs/13_DECISIONS_OPEN_QUESTIONS.md`，避免默默改規則。
- 不可使用大型語言模型判斷 3+5+9 是否等於 17；所有數學規則**必須是可重現的確定性程式碼**。
- 不要暗示任何已完成或已驗證，除非有實際執行結果。

## 1. 不可改動的核心方向（除非使用者另行決定）

1. 三色數字珠分工：紅色＝攻擊、藍色＝防禦、綠色＝恢復。
2. 數學機制：怪物具有變化的數字弱點；不是只玩固定合十。普通消除仍有效，精準弱點命中有加成。
3. 三消起步，可四消、五消以上；連線長度與精確總和都應影響策略。
4. 隨機種子可重現的**分岔路線 Roguelike**，玩家做路徑選擇。
5. **局內遺物與局外成長**同時存在，但永久升級不能繞過必需的數學破防關卡。
6. Pixel Art 怪物；以瀏覽器 Web Audio 程式合成原創 chiptune/BGM/SFX。
7. 網頁平台優先；不可在未取得指示下寫入其他 repo 或複製私有筆記全文。

## 2. 初始可調預設（不是已確認的使用者裁決）

- 盤面 6×6；1～9 正整數；同色、上下左右、3～6 顆連線；不可重複使用座標。
- 攻擊弱點 `sum === target` 時 ×3；相差 1 時 ×1.25；其他 ×1。連線長度另有每多一顆 +25%，最多額外 +75%。
- 十層地圖，第一版 1 個 Boss 三階段；地圖事件與升級價格將靠試玩調整。
- TypeScript / Phaser / Vite / Vitest / Playwright；Web Audio 原生實作。

## 3. 推薦讀檔順序

1. `README.md` 與 `docs/00_PRODUCT_VISION.md`
2. `docs/13_DECISIONS_OPEN_QUESTIONS.md`（確認哪些是提案）
3. `docs/02_MATH_RULES.md` + `docs/03_BOARD_SOLVER.md`（先解決數學精確性）
4. `docs/05_COMBAT_BOSSES.md`
5. `docs/04_ROGUELIKE_MAP.md` + `docs/06_META_ECONOMY.md`
6. `docs/07_TECH_ARCHITECTURE.md` + `docs/11_QA_ACCEPTANCE.md`
7. 依手上任務讀 `docs/08_ASSET_AUDIO.md`、`docs/09_EDUCATION_ANALYTICS.md`、`docs/12_REFERENCES.md`

## 4. 開發原則

- 嚴格分層：`domain/math`、`domain/board`、`domain/battle`、`domain/map`、`domain/economy` 不依賴 Phaser 或 DOM。
- 無副作用的確定性核心；所有隨機以 seed/RNG state 注入、可重播。
- 一律使用**整數數學**檢查數字總和、路徑合法性、目標；倍率若用小數，最終取整規則明訂且統一。
- 任何時候都不能因盤面無解而卡死；生成器與每次盤面改變後都要跑可解性檢查；無解時 deterministic reshuffle 或玩家可理解的救援機制。
- 不要因視覺動畫延遲使戰鬥重複結算；先固定遊戲事件及結果，再播放畫面和聲音。
- 對兒童友善：色盲辨識不只依顏色（圖案／標籤）、數字字級清晰、支援觸控、減少動態、音效開關、無強迫付費或課堂個資收集。
- 對教育敘述保持克制：**遊戲得分與學習成效不是同一件事**。

## 5. 必須交付的品質保證

每個 PR/提交至少列出：

- 範圍及需求依據（對應文件章節）。
- 實際變更檔案與原因；未做事項。
- 可執行測試命令、測試結果與至少一組 seed 重播案例。
- 視覺變更：桌機與手機截圖或等效檢驗；避免圖片被裁切與 HUD 遮擋。
- 發現設計衝突：記於 `docs/13_DECISIONS_OPEN_QUESTIONS.md`。

未執行測試時，必須寫「未執行」及原因，不能寫「已通過」。

## 6. 第一個 AI 實作任務（P0）

1. 建立 Vite + TypeScript + Phaser 3 專案骨架及 `npm` scripts。
2. 定義 `Tile`, `Point`, `Path`, `EnemyWeakness`, `BattleState`, `MapNode`, `RunState`, `MetaState` 等型別。
3. 實作純函式 `validatePath`, `sumPath`, `evaluateWeakness`, `calculateEffect`，含測試。
4. 實作 seeded RNG 及 `findWeaknessPaths`，以 6×6 board、合法同色路徑、弱點精確命中測試。
5. 寫出 board solvability property tests（至少 1,000 seeds；實際測試前不可聲稱已過）。
6. 留下可啟動網頁示範、測試紀錄；**暫時不接 AI 行為推論、遠端資料庫或付費素材 API**。

## 7. 參考專案使用規則

- `bai-collab/pyes_Composition_game`：可讀它的 `game/index.html`、`spec.md`、`tests/game.test.mjs`，參考 seed map、meta snapshot、Web Audio 事件；但**不直接複製作文規則或未確認授權的程式**。
- `bai-collab/Weight-Space-handwired`：只作研究擴充，精確數學判定不需權重模型。
- `smallwhite219/line-notes`：私人資料；若無 GitHub access，標記 ACCESS_UNAVAILABLE，改看 `docs/12_REFERENCES.md` 提供的公開專案，不得臆測私人筆記。
