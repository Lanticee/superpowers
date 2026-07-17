# Fork 版安裝指南（Lanticee/superpowers）

本文件仿照官方 `README.md` 的 Installation 章節，說明如何安裝**這個 fork 版本**
（[`Lanticee/superpowers`](https://github.com/Lanticee/superpowers)），而不是上游官方
（`obra/superpowers`）發佈的版本。

fork 與上游的同步流程請見 [fork-maintenance.md](fork-maintenance.md)。

> **Marketplace 名稱說明**：repo 內建的 `.claude-plugin/marketplace.json` 定義的
> marketplace 名稱是 `superpowers-dev`，plugin 來源指向 repo 根目錄，因此註冊這個
> repo（GitHub 或本地路徑）後，安裝指令一律是 `superpowers@superpowers-dev`。

## 安裝方式（依 harness 區分）

安裝方式因 harness 而異。如果你同時使用多個 harness，每一個都要各自安裝一次。

### Claude Code

#### 從 GitHub fork 安裝

- 註冊 fork 的 marketplace：

  ```bash
  /plugin marketplace add Lanticee/superpowers
  ```

- 從這個 marketplace 安裝 plugin：

  ```bash
  /plugin install superpowers@superpowers-dev
  ```

#### 從本地 clone 安裝（開發用）

適合一邊改一邊測試：本地修改後不需重新發佈，重啟 session 即生效。

- 註冊本地目錄為 marketplace：

  ```bash
  /plugin marketplace add D:\git\superpowers
  ```

- 安裝 plugin：

  ```bash
  /plugin install superpowers@superpowers-dev
  ```

#### 更新

```bash
/plugin marketplace update superpowers-dev
/plugin update superpowers
```

（本地 clone 安裝的話，`git pull` 或本地 commit 後重啟 session 即可。）

### Antigravity

直接從 fork repository 安裝：

```bash
agy plugin install https://github.com/Lanticee/superpowers
```

Antigravity 會執行 plugin 的 session-start hook，所以第一則訊息開始 Superpowers
就會生效。更新時重新執行同一條指令即可。

### Factory Droid

- 註冊 marketplace：

  ```bash
  droid plugin marketplace add https://github.com/Lanticee/superpowers
  ```

- 安裝 plugin：

  ```bash
  droid plugin install superpowers@superpowers-dev
  ```

### GitHub Copilot CLI

- 註冊 marketplace：

  ```bash
  copilot plugin marketplace add Lanticee/superpowers
  ```

- 安裝 plugin：

  ```bash
  copilot plugin install superpowers@superpowers-dev
  ```

### Kimi Code

- 直接從 fork repository 安裝：

  ```text
  /plugins install https://github.com/Lanticee/superpowers
  ```

### OpenCode

OpenCode 使用自己的 plugin 機制；即使其他 harness 已安裝，這裡也要另外裝一次。

- 告訴 OpenCode：

  ```
  Fetch and follow instructions from https://raw.githubusercontent.com/Lanticee/superpowers/refs/heads/main/.opencode/INSTALL.md
  ```

### 無法安裝 fork 版的 harness

以下 harness 只能從各自的**官方 marketplace** 安裝，無法指定 GitHub fork 來源，
因此只能裝到上游官方版本：

- **Codex App / Codex CLI** — 只透過 OpenAI 官方 Codex plugin marketplace 發佈。
- **Cursor** — `/add-plugin superpowers` 走 Cursor 官方 marketplace。

如需在這些 harness 使用 fork 版，得自行維護對應的 marketplace 或手動部署，
不在本文件範圍內。
