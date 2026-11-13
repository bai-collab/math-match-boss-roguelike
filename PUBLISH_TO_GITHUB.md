# 如何把此資料夾上傳到 bai-collab 的新 Repository

> 歷史備份指引：本儲存庫已由使用者建立，2026-11-13 起可直接存取 `bai-collab/math-match-boss-roguelike`。下列新建 Repo 的指令供其他新專案參考；**本 Repo 請使用一般 clone / commit / push 流程，不要再次執行 gh repo create。**

## 方式一：網頁建立 Repo，接著用 Git 推送（最通用）

1. 到 <https://github.com/new>。
2. Owner 選 `bai-collab`，Repository name 設 `math-match-boss-roguelike`。
3. 選 **Private**（若要公開分享，再確認所有內容與參考的授權後改 Public）。建立時**不要勾選** Initialize with README、.gitignore、license，因為本檔案包已有 README。
4. 解壓縮下載的 ZIP，開啟 Windows PowerShell，進入解壓的 `math-match-boss-roguelike` 資料夾。
5. 執行：

```powershell
git init -b main
git add .
git commit -m "docs: initial math match boss roguelike design"
git remote add origin https://github.com/bai-collab/math-match-boss-roguelike.git
git push -u origin main
```

如果 `git commit` 提醒未設定身份，請在本機使用你的 Git 作者名稱及 email 設定；**不可**隨意填寫虛構帳號。若遠端已有提交，先查看 Git 歷史後再處理，不可強制推送覆蓋。

## 方式二：GitHub CLI（已安裝且登入 `gh` 的情況）

在解壓資料夾中執行：

```powershell
# 依需要把 --private 改為 --public，但公開前先檢查私人內容與授權
git init -b main
git add .
git commit -m "docs: initial math match boss roguelike design"
gh repo create bai-collab/math-match-boss-roguelike --private --source . --remote origin --push
```

## 建立後必查

- `https://github.com/bai-collab/math-match-boss-roguelike` 網頁能打開。
- `README.md`、`AGENTS.md`、`docs/`、`design/` 全部上傳成功。
- README 明確寫「SPECIFICATION / NOT IMPLEMENTED」。
- 將此 repo 連結交給下一位 AI，並下指令：「請先讀 AGENTS.md，再依 docs/10_IMPLEMENTATION_ROADMAP.md 完成 P0」。
