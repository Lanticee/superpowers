# Fork 維護指南

本文件說明如何讓這個 fork（`Lanticee/superpowers`）與官方上游（`obra/superpowers`）保持同步，同時保留自己的客製修改。

fork 版的安裝方式請見 [fork-install.md](fork-install.md)。

## Fork 架構

本地 repo 設定了兩個 remote：

| Remote | 位置 | 角色 |
|--------|------|------|
| `origin` | `https://github.com/Lanticee/superpowers.git` | 你的 fork，本地修改推送到這裡 |
| `upstream` | `https://github.com/obra/superpowers.git` | 官方 repo，只拉取、不推送 |

可用 `git remote -v` 確認設定。

歷史結構的原則是：**上游的 commit 在下面，自己的客製 commit 墊在最上面**。每次同步就是把自己的 commit 重新墊到最新的上游之上（rebase）。

## 本地客製 commit

目前墊在上游之上的修改，依用途分組（`git log --oneline upstream/main..main` 可看完整 commit 列表）：

### 1. Subagent 模型路由與 agy 第二意見（`skills/subagent-driven-development/SKILL.md`）

- **實作任務一律交給 Sonnet** — 機械性實作與單檔修正走 Sonnet，不用 Haiku（最便宜的層級在多步驟工作上會多耗 2-3 倍回合）
- **agy（Gemini Flash）只當跨模型第二意見** — 不派實作任務給 agy（需要 `--dangerously-skip-permissions`，會被 Claude Code 的權限分類器擋下）
- **優先走 agy-cli plugin** — 有安裝時改派給 `agy-cli:gemini-flash` subagent；沒有才用 raw `agy --print ... --mode plan`
- **raw agy 的操作規則** — stdin 導向 `/dev/null`、`--mode plan` 才能讀檔、Windows argv 32KB 上限、空輸出的兩種成因、額度錯誤後整個 session 停用
- **raw agy 的模型名稱** — 目前指定 `"Gemini 3.8 Flash (High)"`。Gemini 出新版時要跟著改，可用 `agy models` 查帳號目前可用的模型

這組是逐步演進的（最早是把 cheap-tier 任務整個交給 Gemini 3.5 Flash，後來改成只做第二意見），rebase 時若上游改到 Model Selection 段落，以上述最終行為為準解衝突。

### 2. 中文文件

- **`docs/fork-maintenance.md`** — 這份文件本身
- **`docs/fork-install.md`** — fork 版各 harness 安裝方式

### 3. 暫時性除錯內容：`testplugin/`

- 為了排查 copilot CLI 安裝失敗，用 bisect 方式逐步縮減的測試 plugin（`Add minimal test plugin…` 與一串 `test: bisect - …` commit）
- 內容是舊版 repo 的部分複本，不會被主 plugin 使用。**排查結束後可整組移除**，移除前確認沒有其他地方引用

### 已不再需要的客製

- **Replace AGENTS.md symlink with a regular file** — 原本因為上游的 `AGENTS.md` 是指向 `CLAUDE.md` 的 symlink，copilot CLI 在 Windows 安裝時重建 symlink 會失敗而改成實體檔。上游 v6.4.1 起已刪除 `CLAUDE.md`、`AGENTS.md` 本身就是一般檔案，這個修改已無作用（commit 仍在歷史中，只剩本文件的說明變更）

> 之後若新增客製 commit，記得更新這份清單，rebase 時才知道哪些是自己的修改。

## 同步流程

三個步驟：

```bash
# 1. 抓取上游最新內容（不會動到本地分支）
git fetch upstream

# 2. 把自己的 commit 重新墊到最新的上游之上
git rebase upstream/main

# 3. 推回自己的 fork（rebase 改寫了歷史，需要強推）
git push --force-with-lease origin main
```

同步前建議先確認工作目錄是乾淨的（`git status`），有未提交的修改先 commit 或 stash。

想先看差距再動手，可以在 fetch 之後執行：

```bash
# 左邊是自己領先的 commit 數，右邊是上游領先的 commit 數
git rev-list --left-right --count main...upstream/main

# 列出上游有而自己沒有的 commit
git log --oneline main..upstream/main
```

## 遇到衝突怎麼辦

如果上游改到了和客製 commit 相同的地方，rebase 會停下來並標示衝突檔案：

1. 打開衝突檔案，解決 `<<<<<<<` / `=======` / `>>>>>>>` 標記的區塊
2. `git add <解完的檔案>`
3. `git rebase --continue`
4. 若有多個 commit 衝突，重複以上步驟直到 rebase 完成

想放棄這次同步、回到 rebase 前的狀態：

```bash
git rebase --abort
```

如果 rebase 完成後才發現結果不對，可以用 reflog 找回 rebase 前的狀態：

```bash
git reflog                 # 找到 rebase 前的 commit hash
git reset --hard <hash>    # 回到該狀態
```

## 注意事項

**為什麼用 `--force-with-lease` 而不是 `--force`？**
rebase 改寫了 commit 歷史，推送時一定需要強推。`--force-with-lease` 會先檢查遠端分支是否和本地認知一致——如果遠端有你不知道的新 commit（例如在別台機器或 GitHub 網頁上改過），推送會被拒絕而不是直接覆蓋。`--force` 則會無條件覆蓋，風險較高。

**為什麼用 rebase 而不是 merge？**
merge 會產生合併 commit，多次同步後歷史會變得混亂，也難以一眼看出哪些是自己的修改。rebase 讓客製 commit 永遠整齊地墊在上游最新版之上，`git log` 前幾筆就是自己的全部修改，之後若要整理或提交上游也容易得多。

**不要直接在 GitHub 網頁上用 "Sync fork" 按鈕。**
該按鈕在 fork 與上游分歧時可能建立 merge commit 或提示放棄本地修改，和本文的 rebase 流程衝突。一律在本地用上述流程同步。
