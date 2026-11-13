# 08｜Pixel Art、動畫資產與瀏覽器原創電子音

## 1. 視覺規格（初始提案）

- 像素風格 16-bit 感、深藍夜色背景，怪物外型易讀、數字清晰。
- 怪物原稿 64×64 px（可依美術品質擴充），角色與小道具 32×32 px，並用整數倍 nearest-neighbor 放大；勿使用平滑縮放導致邊緣模糊。
- 每隻 Boss 至少 `idle / attack / hurt / death`；普通怪可先做 `idle / hurt / death`。
- 同一怪物必須維持固定調色盤、固定朝向、腳底對齊、透明背景、統一 frame 尺寸。
- 先用簡單程式化方塊占位，功能完成後再換成正式 sprite；**不要讓資產生成卡住數學規則開發**。
- 像素特效依事件觸發：弱點命中、破盾、護盾、治療、Boss 階段變化。正確／錯誤回饋除顏色外應有文字圖案。

## 2. 參考資產流程（已讀 README/私人筆記摘要）

1. **[0x0funky/agent-sprite-forge](https://github.com/0x0funky/agent-sprite-forge)**：一張主視覺 → 多個動作影片／影格 → 對齊 → 透明 sprites/atlas。可搭 Codex／Claude Code。
2. **[aldegad/sprite-gen](https://github.com/aldegad/sprite-gen)**：清理透明 alpha、state rows、frame extraction、runtime atlas + manifest；適合補強動畫一致性。
3. **[patch-gg/pixelfx](https://github.com/patch-gg/pixelfx)**：程式化像素特效與匯出精靈圖；用於傷害／治療／暴擊等。
4. **[Scenario Pixel VFX Forge](https://www.scenario.com/explorations/pixel-vfx-forge/)**：瀏覽器特效概念及素材產生參考。
5. **[amirmushichge/pixel-plays](https://github.com/amirmushichge/pixel-plays)**：小型網頁像素遊戲的低解析渲染、pixel perfect 風格參考，**不表示可直接照搬其音樂素材**。
6. **[bai-collab/pyes_Composition_game](https://github.com/bai-collab/pyes_Composition_game)**：`art/pixel-preview.html` 與 `game/index.html` 有程式化像素角色、SVG/矩陣與動作事件可參考。

以上工具未保證全部無成本：如果使用 AI 影像／影片生成，可能牽涉帳號、API、授權與費用，須自行查驗。遊戲本身應能使用自產像素圖離線運作。

## 3. 資產格式規範

```text
assets/sprites/<enemy-id>/master.png
assets/sprites/<enemy-id>/idle.png
assets/sprites/<enemy-id>/attack.png
assets/sprites/<enemy-id>/hurt.png
assets/sprites/<enemy-id>/death.png
assets/atlases/<enemy-id>.png
assets/atlases/<enemy-id>.json
assets/palettes/default.json
assets/provenance.csv   # file,source,creator,license,generated_on
```

每個影格 `frameWidth`、`frameHeight`、`origin` 與 `fps` 都要有機器可讀 manifest；遊戲不可把圖集格數硬寫在單一動畫函式中。動作結算與播放分離；減少動態開啟時用靜止代表格。

## 4. 原創 chiptune 設計

- 使用**原生 Web Audio API**（`AudioContext`, `OscillatorNode`, `GainNode`, `BiquadFilterNode`, `AudioBufferSourceNode`）。
- 原創波形：方波 lead（旋律）、三角波 bass、鋸齒波和聲、噪音鼓組（kick/snare/hat 可用振幅與濾波包絡製作）。
- `AudioEngine` 在使用者第一次有效點擊／按鍵後才建立／`resume()` AudioContext，避免瀏覽器自動播放限制。
- 音樂和音效各有 bus，接到 master gain/compressor；支援全靜音、分別音量、頁面背景暫停與按需恢復。
- **不引用現有非公版音樂的旋律、採樣或 MIDI**。所有旋律、節奏與音效為新作的程序合成，或使用使用者確認擁有權利的素材。
- BGM 不需要音檔、不需要網路 API 才能播放；離線正常。

## 5. 建議曲目與事件

| 場景 | 節奏（初始建議） | 描述 |
|---|---|---|
| 主選單／地圖 | 約 88～96 BPM | 輕快且可無縫循環 |
| 一般戰鬥 | 約 118～124 BPM | 鼓與低音增加緊張感 |
| Boss 階段 1 | 約 138 BPM | 小調，清晰短旋律 |
| Boss 階段 2 | 約 145 BPM | 增加額外聲部 |
| Boss 階段 3 | 約 152 BPM | 更密集節奏但避免過度刺激 |
| 勝利／戰敗 | 短提示音 | 不強制延續長 BGM |

| Sound Event | 音效建議 |
|---|---|
| `MATCH` | 逐顆較短上行音 |
| `WEAKNESS_EXACT` | 和弦或八度上行，明確且不刺耳 |
| `WEAKNESS_NEAR` | 中性雙音，不誤導為精確成功 |
| `DEFEND` | 柔和盾牌聲 |
| `HEAL` | 三角波上行琶音 |
| `BOSS_PHASE_BREAK` | 低音衝擊加和聲 |
| `PLAYER_HURT` | 短降音、可調低音量 |
| `MENU_SELECT` | 柔和短提示 |

**聽覺可辨識**不代表必須開聲；所有重要資訊都要同時有文字/圖像。可加入音效節流（例同事件間隔 50ms）防止多次重複出聲。

## 6. 畫面與聲音同一事件來源

```ts
const {nextState,events} = reduceBattle(state, action);
render(nextState);           // 視覺
playSoundsFrom(events);      // 聲音，可能因靜音而無效果
recordEvents(events);        // 記錄，須可停用
```

**不可**讓 `AudioEngine` 決定數學正確與否，也不可讓動畫結束 callback 再次計算傷害。禁用音訊或減少動態時，遊戲玩法結果必須完全相同。
