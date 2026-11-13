# 12｜參考資源與來源查核紀錄

> 查核日期：2026-11-13（以本次 GitHub 連線實際讀取之版本為基準）。**檢索到某專案 ≠ 已驗證全部功能或授權**。以下區分「已讀程式／README」與「只出現在私人筆記」。

## A. 直接檢視的既有專案（優先）

| 資源 | 已讀範圍 | 本專案可參考內容 | 注意 |
|---|---|---|---|
| [bai-collab/pyes_Composition_game](https://github.com/bai-collab/pyes_Composition_game) | `README.md`, `spec.md`, `game/index.html` 的 `generateMap`, `META_UPGRADE_LINES`, `startRun`, `tests/game.test.mjs` 的部分測試 | 10 層地圖、Seed RNG、局外進度快照、Web Audio 音樂架構、像素畫 | **未發現根目錄 LICENSE；不可未查授權直接複製程式。** 作文題庫、句子卡及成語升級不直接沿用 |
| [bai-collab/Weight-Space-handwired](https://github.com/bai-collab/Weight-Space-handwired) | `README.md`, `examples/handwired-basics.mjs`（預設分支 `claude/peaceful-newton-vgciag`，檢視當時） | 可解釋特徵×權重＋偏差＋ReLU、後續學生策略樣態分析 | 不適合取代整數算術的精確驗證；需校準和教師審查 |
| [0x0funky/agent-sprite-forge](https://github.com/0x0funky/agent-sprite-forge) | README | 單張主視覺製作多動作 sprite sheet、透明影格、像素風 | 不保證生成流程無 API 成本，需查授權／工具需求 |
| [aldegad/sprite-gen](https://github.com/aldegad/sprite-gen) | README | Sprite 清理／對齊、frame extraction、Atlas/manifest | 需檢查工具安裝與各素材授權 |
| [patch-gg/pixelfx](https://github.com/patch-gg/pixelfx) | README | 程式化像素特效、瀏覽器使用、匯出 sprite sheet | 特效只能參考／依授權使用 |
| [0x0funky/agent-game-forge](https://github.com/0x0funky/agent-game-forge) | README | AI 協作 2D 遊戲 IDE、網頁 Canvas 輸出、視覺編輯 | README 當時標示 pre-launch；不可假設已成熟、也不必作為執行時相依 |
| [amirmushichge/pixel-plays](https://github.com/amirmushichge/pixel-plays) | README | 不依賴框架的小型 pixel-perfect 網頁遊戲概念 | README 指明音樂不隨程式 MIT 授權；不要直接搬其音樂 |
| [Donchitos/Claude-Code-Game-Studios](https://github.com/Donchitos/Claude-Code-Game-Studios) | README | 一頁遊戲 brief、先做原型、測試與迭代的 Agent 工作流 | 非數學三消引擎，僅參考流程 |

### 來源重點對照

- **路線**：`pyes_Composition_game/spec.md` 的「爬塔、成長與經濟」及 `game/index.html` 的 `generateMap(seed)`。
- **局外經濟**：同一 `spec.md`「墨水商店」與 `game/index.html` 的 `META_UPGRADE_LINES`, `startRun`（只參考層級／存檔快照的設計）。
- **Web Audio**：`pyes_Composition_game/spec.md` 的「音樂與音效」記載原生 Web Audio 合成、不同場景節奏、audio bus 與事件發音；本遊戲重寫原創旋律。
- **Pixel 資產**：`agent-sprite-forge`, `sprite-gen`, `pixelfx` README。

## B. 私人 GitHub 筆記（不能假設每位 AI 都有權限）

來源 Repo：[`smallwhite219/line-notes`](https://github.com/smallwhite219/line-notes)；本次檢查其 visibility 為 private。這是使用者資料，不能複製私人筆記全文到公開 repo。

本次已閱讀、與新專案相關的例子：

- `notes/2026/11/2026-11-03-170848-像素遊戲開發工具清單.md`：提到 Phaser、PixelLab、claude-pxl 等。
- `notes/2026/11/2026-11-13-112619-GitHub-0x0funkyagent-sprite-forge-Agent.md`：Sprite Forge 概念與 URL。
- `notes/2026/11/2026-11-06-051229-GitHub-patch-ggpixelfx-244-pixel-art-eff.md`：PixelFX 連結與用途。
- `notes/2026/11/2026-11-12-084250-AI-生成精靈圖與裁切流程.md`：生成素材、裁切與檢核構想。
- `notes/2026/05/2026-05-15-083750-GitHub-0x0funkyagent-game-forge-The-loca.md`：AI 2D IDE 介紹。
- `notes/2026/09/2026-09-10-205635-GitHub-aldegadsprite-gen-Generate-clean.md`：2D Atlas 流程。
- `notes/2026/11/2026-11-12-170230-GitHub-amirmushichgepixel-plays-Small-br.md`：瀏覽器像素小遊戲。
- `notes/2026/10/2026-10-30-181544-Claude-Code-遊戲工作室.md`：精簡規格、先試做原型。

**存取規則**：AI 若無私人 GitHub 授權，標記 `ACCESS_UNAVAILABLE`，直接使用 A 欄提供的公開專案；不得憑此文件推斷私人筆記所有內容均已確認。

## C. 官方技術入口（供後續版本驗證）

- Phaser 官方文件：<https://docs.phaser.io/>
- Vite 官方文件：<https://vite.dev/guide/>
- TypeScript 官方文件：<https://www.typescriptlang.org/docs/>
- Vitest 官方文件：<https://vitest.dev/guide/>
- Playwright 官方文件：<https://playwright.dev/docs/intro>
- MDN Web Audio API：<https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API>
- MDN Web Audio 自動播放相關限制：<https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay>
- GitHub Pages：<https://docs.github.com/en/pages>

這些是官方入口參考，**不代表本次逐一瀏覽並查核了最新版本**。開始實作時，負責的 AI 必須自行查詢最新 API、依賴相容性與授權。

## D. 關於未直接使用的素材與方法

- `line-notes` 的內容是「使用者曾收藏／筆記」，不等於筆記所談的技術有效性已外部驗證。
- `pyes_Composition_game` 是**另一套作文卡牌遊戲**；借鏡地圖、局外與音效結構，不是照搬作文邏輯。
- 所有新增 Boss、HP、倍率、平衡表都是這次共同設計的**原創提案／預設**，不是第三方專案的實際數據。
- 如要使用第三方 Sprite、音樂或文字，必須建立獨立的 LICENSE/provenance 檢查，而不是只因其程式開源就認為全部資產可用。
