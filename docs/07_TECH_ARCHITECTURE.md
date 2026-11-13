# 07｜工程架構、資料模型、存檔與事件界面

## 1. 建議工程堆疊（非強制，開始實作前檢查版本）

- TypeScript（strict）、Vite、Phaser 3（2D Canvas/WebGL）、Vitest、Playwright。
- 網頁前端優先、可部署 GitHub Pages 或 Cloudflare Pages；音訊由原生 Web Audio API 即時生成。
- 學習分析（若使用）為事件紀錄，可先留 localStorage，不需帳號／伺服器。
- 僅把 Sprite Forge、sprite-gen、PixelFX 當作**美術建置工具**，不依賴它們於瀏覽器中即時工作。
- 本文沒有實測所有工具的特定版本組合；建置時先查官方相容性與授權。

## 2. 建議原始碼結構（尚未建立）

```text
src/
  main.ts
  app/
    GameApp.ts                 # 應用啟動與路由
    GameController.ts          # 業務事件序列／狀態變更
  domain/
    math/                      # validatePath/evaluatePath/calculateEffect
    board/                     # board generation, solver, refill, reshuffle
    battle/                    # intents, damage/shield/heal, boss phase
    map/                       # seeded map, reachability
    economy/                   # relics, temporary upgrades, meta upgrades
    rng/                       # deterministic random & serialization
    models/                    # Tile, Weakness, RunState, GameEvent
  scenes/
    BootScene.ts
    MainMenuScene.ts
    MapScene.ts
    BattleScene.ts
    RewardScene.ts
    MetaShopScene.ts
    RunResultScene.ts
  ui/
    widgets/                   # number tile, weakness label, HUD
    accessibility/             # keyboard & touch alternative
  audio/
    AudioEngine.ts
    Patterns.ts                # original chiptune compositions
    SoundEvents.ts
  assets/
    sprites/
    atlases/
    palettes/
  persistence/
    SaveManager.ts
    migrations.ts
    validators.ts
  analytics/
    LocalEventLogger.ts
    HandwiredAdapter.ts         # optional, disabled by default
  config/
    gameConfig.ts
    bossCatalog.ts
  tests/
    math.test.ts
    board.property.test.ts
    map.property.test.ts
    battle.test.ts
    meta.test.ts
    save.test.ts
    mobile.e2e.spec.ts
```

## 3. 系統界面與責任

- **Math Engine**：取得 board/path/weakness/config，輸出合法性、總和、倍率、效果（無副作用）。
- **Board Engine**：負責產生、填補盤面、尋找解路徑、救援；不得直接扣生命。
- **Battle Engine**：以數學輸出結算攻擊／盾／回復，並處理 Boss 封印、怪物回合。
- **Map Engine**：seed→10 層 nodes/edges，控制可走路線；不直接調用畫面。
- **Economy Engine**：定義局內商店、遺物、局外升級的購買/取得/生效規則。
- **GameController**：統一發出遊戲事件，更新狀態，再通知 Phaser UI／音樂層。
- **Audio Engine**：只訂閱事件，如 `MATCH`, `WEAKPOINT_EXACT`, `BOSS_PHASE`, `PLAYER_HURT`；不得回寫分數。
- **Analytics**：只記錄必要的遊戲事件，不干擾戰鬥邏輯。

## 4. 核心狀態契約

```ts
type GameState={
  saveVersion: number;
  meta: MetaState;
  run: RunState | null;
  settings: Settings;
};

type MetaState={
  currency: number;
  clears: number;
  unlockedCharacters: string[];
  upgrades: {survival:number;wisdom:number;technique:number;exploration:number};
};

type RunState={
  runId:string;
  seed:number;
  rngState:number;
  metaSnapshot:MetaState['upgrades'];
  characterId:string;
  floor:number;
  map:MapLayers;
  availableNodeIds:string[];
  visitedNodeIds:string[];
  hp:number;
  maxHp:number;
  gold:number;
  relicIds:string[];
  status:'active'|'won'|'lost';
  battle:BattleState|null;
  settled:boolean;
};
```

請以此作為**概念契約**，真正 TypeScript 定義應補強 `readonly`、版本判別聯集、schema validation 與錯誤處理，並讓 `RunState` 完整攜帶所有重播所需資訊（例如 RNG 與排程意圖）。

## 5. 無副作用事件與狀態轉移

```text
Input (touch/mouse/keyboard)
   ↓
validatePath(board,path)
   ↓
reduceBattle(state, {type:'SUBMIT_PATH', path})
   ├── math result / action cost / combat result
   ├── board drop & refill (seeded)
   └── immutable next BattleState + GameEvent[]
   ↓
GameController publishes events
   ├── Phaser animation & HUD
   ├── WebAudio sound events
   ├── LocalEventLogger (optional)
   └── SaveManager
```

**一個動作只結算一次**；重放動畫或刷新網頁不能再次套用攻擊。以 `actionId` 去重，勝利、節點獎勵與局外結算需有 `settled` guard。

## 6. SaveManager 規則

- storage keys 例如 `math-match-boss-save-v1`、`math-match-boss-settings-v1`（不得與作文遊戲共用存檔 key）。
- `saveVersion=1`；載入時 schema 驗證，非法或損毀狀態不得靜默覆蓋；介面顯示「舊存檔無法載入，暫時不覆寫」。
- 存檔至少在 `startRun`、成功結算一條路徑（如果恢復戰鬥必需）、回合結束、節點完成、購買升級、領獎、結束一局時更新。
- 持久化 `run.rngState` + 完整盤面 + Boss phase + 已領獎節點；否則刷新後不具可重播性。
- `MetaState` 與 `RunState` 嚴格分開；局內購買不可直接改永久級別。
- 初版 `localStorage`；後續如導入帳號同步，另設隱私與衝突處理規格。

## 7. 設定驅動與測試

- 所有倍率、目標、Boss HP/傷害、地圖比例、升級費用都寫 config，**不要散落 hardcoded magic numbers**。
- `design/game-config.example.yaml` 是人類可讀樣板，實作時可轉成 TS / JSON 並執行 schema validation。
- Unit tests 優先覆蓋純邏輯，Playwright 只處理真正需要 UI/觸控的部分；不要把每個算術測試都放到瀏覽器端跑。
- CI 建議 `npm run typecheck && npm run test && npm run build`，以及可選的 Playwright smoke tests。

## 8. 安全與依賴

- 禁止將第三方 API key、學生姓名或連線憑證放到前端程式或 Git。
- 所有遠端資源可替換為本地檔案；可用工具需查授權，GitHub README 不等於授權條款。
- 不依賴中國模型／服務；本案核心算術與生成均可離線執行。
