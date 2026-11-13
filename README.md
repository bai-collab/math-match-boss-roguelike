# Math Match Boss Roguelike｜數學弱點三消爬塔

> **Status: SPECIFICATION / NOT IMPLEMENTED（2026-11-13）**  
> 本儲存庫目前是**開發計畫與可驗收規格**，**不是已完成的遊戲**。請先閱讀 [`AGENTS.md`](AGENTS.md) 與 [`docs/00_PRODUCT_VISION.md`](docs/00_PRODUCT_VISION.md)。

以「**三色數字連線消除 × 怪物數字弱點 × 十層隨機爬塔 × 局內遺物 × 局外成長**」為核心的、適合國小數學學習情境的瀏覽器 Roguelike Boss 戰遊戲。使用 Pixel Art 與網頁即時合成 8-bit 電子音樂，原則上不依賴付費 AI 或音樂 API 才能遊玩。

## 30 秒理解遊戲

1. 進入一局以 seed 產生的 10 層分岔爬塔，選擇普通怪、菁英、營火、商店、試煉與事件，最頂端挑戰 Boss。
2. 戰鬥時在 6×6 盤面**上下左右拖曳連接 3～6 顆相同顏色**的 1～9 數字珠；紅＝攻擊、藍＝護盾、綠＝治療。
3. 怪物展示**弱點目標數字**（如 17）。連線數字合計剛好 17，可使**該顏色的本次效果**提高；不等於 17 仍可正常施放。四消、五消另有連線獎勵。
4. 面對具有不同限制的怪物（如四消才能破甲、雙核心弱點、逐回合變換弱點），學生需計算並選擇行動，而非重複固定合十。
5. 戰勝後選遺物、獲得局內資源；本局結束取得局外貨幣，於局外升級生存、提示、操作、探索；永久強化不得繞過數學破防關卡。

> **需求與假設分離：** 上述「三色、怪物弱點、隨機路徑、局外成長、網頁像素美術／電子音」為使用者已提出的方向；6×6、數值 1～9、3～6 連線、倍率、地圖節點分布、經濟價格等為**可調整的設計預設值，尚未經平衡測試**。詳細見 [`docs/13_DECISIONS_OPEN_QUESTIONS.md`](docs/13_DECISIONS_OPEN_QUESTIONS.md)。

## 文件導覽（依照開發順序閱讀）

| 目的 | 文件 |
|---|---|
| 所有 AI 最先讀取的執行規約 | [`AGENTS.md`](AGENTS.md) |
| 產品目標、範圍、成功條件 | [`docs/00_PRODUCT_VISION.md`](docs/00_PRODUCT_VISION.md) |
| 核心玩法、玩家流程、畫面需求 | [`docs/01_GAME_DESIGN.md`](docs/01_GAME_DESIGN.md) |
| 數學條件、精確判定與倍率公式 | [`docs/02_MATH_RULES.md`](docs/02_MATH_RULES.md) |
| 盤面生成、尋路、無解處理 | [`docs/03_BOARD_SOLVER.md`](docs/03_BOARD_SOLVER.md) |
| 種子地圖、10 層路線、節點 | [`docs/04_ROGUELIKE_MAP.md`](docs/04_ROGUELIKE_MAP.md) |
| 回合制戰鬥、敵人、Boss 設計 | [`docs/05_COMBAT_BOSSES.md`](docs/05_COMBAT_BOSSES.md) |
| 局內遺物、局外成長、資源平衡 | [`docs/06_META_ECONOMY.md`](docs/06_META_ECONOMY.md) |
| TypeScript／Phaser 2D 架構、狀態、存檔 | [`docs/07_TECH_ARCHITECTURE.md`](docs/07_TECH_ARCHITECTURE.md) |
| Pixel Sprite、特效、Web Audio 原創音效 | [`docs/08_ASSET_AUDIO.md`](docs/08_ASSET_AUDIO.md) |
| 數學教學目標、提示、匿名行為紀錄 | [`docs/09_EDUCATION_ANALYTICS.md`](docs/09_EDUCATION_ANALYTICS.md) |
| P0～P4 任務、交付物與禁止擴張 | [`docs/10_IMPLEMENTATION_ROADMAP.md`](docs/10_IMPLEMENTATION_ROADMAP.md) |
| 單元／整合／隨機種子／無障礙驗收 | [`docs/11_QA_ACCEPTANCE.md`](docs/11_QA_ACCEPTANCE.md) |
| 已讀來源、可參考檔案、授權注意 | [`docs/12_REFERENCES.md`](docs/12_REFERENCES.md) |
| 使用者決策、預設假設、待決議題 | [`docs/13_DECISIONS_OPEN_QUESTIONS.md`](docs/13_DECISIONS_OPEN_QUESTIONS.md) |
| 可交給其他 AI 的任務模板 | [`docs/14_AGENT_HANDOFF.md`](docs/14_AGENT_HANDOFF.md) |
| 初始設定檔（示意） | [`design/game-config.example.yaml`](design/game-config.example.yaml) |
| 怪物資料（示意） | [`design/bosses.example.json`](design/bosses.example.json) |

## 建議技術（尚未安裝）

- **Phaser 3 + TypeScript + Vite**：2D 渲染、輸入、動畫與場景；將數學／戰鬥規則放在不依賴 Phaser 的純邏輯模組。
- **Vitest**：數學規則與亂數盤面的單元／性質測試；**Playwright**：桌機與手機尺寸的端對端測試。
- **原生 Web Audio API**：即時合成原創 chiptune，並以事件界面與玩法分離。
- **localStorage（初版）**：本機遊戲進度；未來教師分析另行評估資料收集與同意程序。
- **Sprite Forge／sprite-gen／PixelFX**：美術資產製作工具，**不是必須的執行時相依**。
- **Weight-Space-handwired**：可解釋學習行為分析的後續可選研究模組，不用來取代精確算術判斷。

## 與既有專案的關係

- [bai-collab/pyes_Composition_game](https://github.com/bai-collab/pyes_Composition_game)：參考 10 層六角格隨機路線、seed、局外升級、Web Audio 架構、像素圖。**不要直接移植作文題庫或卡牌規則**。
- [bai-collab/Weight-Space-handwired](https://github.com/bai-collab/Weight-Space-handwired)：參考可解釋行為特徵計分，不是三消引擎。
- [smallwhite219/line-notes](https://github.com/smallwhite219/line-notes)：使用者**私人筆記**。此儲存庫不能假設其他 AI 有權限存取；所需公開工具連結已整理在 [`docs/12_REFERENCES.md`](docs/12_REFERENCES.md)。**不得把私人筆記全文直接複製到公開專案**。

## 實作狀態與下一步

- [x] 綜合產品需求與參考資源、形成詳細計畫
- [ ] P0：遊戲專案骨架、純函式算術規則、設定表與自動測試
- [ ] P1：可拖曳 6×6 三色數字連線＋保證可解盤面
- [ ] P2：一隻三階段 Boss、可完整一局的戰鬥
- [ ] P3：可重現的 10 層隨機地圖、獎勵與局外成長
- [ ] P4：完整像素動畫、瀏覽器電子音、無障礙、學習分析與平衡

**其他 AI 請勿把上述待辦當作已實作功能。** 每完成一項必須附測試證據、更新對應文件與紀錄設計差異。

## 授權與公開注意

本專案尚未選擇 LICENSE。任何從外部參考專案**複製程式碼、圖像、音樂或素材**前，必須各自查核授權及歸屬。特別是 `pyes_Composition_game` 當前檢查的檔案列表未看到 LICENSE；應先採**借鏡設計、重新實作**，不得逕行假設可複製。公開前亦請再次檢查是否含學生個資、私有連結或私人筆記內容。
