# 14｜可直接交給其他 AI 的實作任務模板

## A. 第一個任務（請直接複製給 Codex / Claude Code）

```text
你已接手一個尚未開始實作的「數學弱點三消 × Roguelike Boss 戰」Web 專案。
先完整閱讀 AGENTS.md、README.md、docs/00_PRODUCT_VISION.md、
docs/02_MATH_RULES.md、docs/03_BOARD_SOLVER.md、
docs/07_TECH_ARCHITECTURE.md、docs/11_QA_ACCEPTANCE.md、
docs/13_DECISIONS_OPEN_QUESTIONS.md。

本次只完成 P0，不要擅自延伸到 P3、後端、AI 評分或商業素材。

要求：
1. 建立 Vite + TypeScript + Phaser 3 骨架、TypeScript strict、Vitest。
2. 完成純函式 validatePath/sumPath/evaluatePath/calculateEffect；精確整數算術與倍率取整規則要一致。
3. seeded RNG、DFS 合法紅珠弱點尋路、盤面生成／無解救援；每次 6×6 board 都能符合弱點條件。
4. 設置 1,000 seeds property test 與 docs/11_QA_ACCEPTANCE.md 的 M/B 測試。
5. 不要複製 pyes_Composition_game 原始程式（授權未確定），僅參考其設計；記錄每個外部素材來源。
6. 實際跑 npm run typecheck、npm run test、npm run build，提交實際結果；若無法跑，明說未測。
7. 更新 README 實作狀態、決策記錄，以及你修改的檔案說明。

請先用 1 段摘要本次要做的事情，再直接產出可運行程式與測試；遇到不影響 P0 的開放問題，可依 HYP 預設做最小決策並記錄。
```

## B. 接手盤點模板

```text
請依序提供：
1. 已讀文件：請列出路徑，不可只說「看過規格」。
2. 現有程式與計畫差異：已完成 / 缺失 / 偏離。
3. 本次範圍：對照 10_IMPLEMENTATION_ROADMAP 中 P0~P4。
4. 會修改的檔案、測試方式、不可更動的需求。
5. 執行之後：git diff、測試結果、截圖／重播 seed、未完成事項。
```

## C. 不應接受的 AI 交付

- 報稱「完成遊戲」，實際只有靜態畫面或一個 `README.md`。
- 在畫面上顯示「弱點 17」但程式沒有真正計算數字總和。
- 只在進戰鬥時檢查一次可解，消除後盤面可能永遠無法命中弱點。
- 玩家未完成 Boss 數學弱點也能靠永久升級直接擊殺。
- 用 AI 生成聲音，但沒有本機 Web Audio 備援，或將既有受版權保護的音樂照搬。
- 未實際執行測試卻聲稱「所有測試通過」。
- 未取得授權就直接複製別的專案大段程式、音樂、素材。

## D. 二階段以後的任務建議

- P1：只做拖曳連線盤面與可讀 UI；不要先做大量怪物動畫。
- P2：只做數字魔龍三階段與完整勝敗流程；使用占位 Sprite 亦可。
- P3：可參考 `pyes_Composition_game` 地圖與 Meta 形狀，但重新命名資料並增加數學事件節點。
- P4：將程序化 Pixel Art、音效、行為分析和教育提示逐步插入；每個擴充都要確保原本固定種子測試依然通過。
