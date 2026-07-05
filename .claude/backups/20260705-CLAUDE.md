# CLAUDE.md — triathlon-radar

## 這個專案是什麼
單一產品檔案 `index.html`（繁體中文單頁儀表板）：TrainingPeaks YouTube 頻道鐵人三項訓練影片的每週雷達，呈現影片摘要、訓練分類標籤、本週推薦。使用者是 Steve（繁體中文使用者、鐵人三項訓練者）。repo 裡沒有 build 系統、沒有測試框架——驗證方式是直接開啟或 render 這個 HTML。

## 硬規則（違反即算任務失敗）
1. 回覆與 UI 文案一律繁體中文（程式碼、commit message 用英文可以）。
2. `index.html` 禁止用 `Write` 全檔覆寫。只用 `Edit`，`old_string` 錨點必須含該處獨有文字（如影片標題）。讀檔先 `Grep` 定位，再用 `Read` 的 offset/limit 讀區段。
3. 只在 harness 指定的 `claude/*` 分支開發與 push，不直接碰 `main`。
4. 不修改 `~/.claude/` 下的 hooks 與 launcher 設定（平台管理）。
5. 修改 `.claude/guides/` 或本檔前，先複製一份到 `.claude/backups/`（命名規則見 guides/04-maintenance.md）。

## 開工前 30 秒
1. 讀 `.claude/memory/LESSONS.md`（若存在）——前人踩過的坑，直接影響你這次會不會重踩。
2. 按需要查路由表：

| 情境 | 讀這個檔 |
|---|---|
| 想知道這環境哪裡最容易漏 token、出錯 | `.claude/guides/00-diagnosis.md` |
| 要派 subagent、選模型、升降級 | `.claude/guides/01-delegation.md` |
| 不確定「做完了沒」「該不該問使用者」「方向對不對」 | `.claude/guides/02-judgment.md` |
| 要寫委派 prompt（搜尋/實作/重構/研究/審查） | `.claude/guides/03-prompt-templates.md` |
| 要更新以上任何規則檔 | 先讀 `.claude/guides/04-maintenance.md` |
| 想了解本環境的待辦大事與制度退化風險 | `.claude/guides/05-letter-to-future-sessions.md` |

## 環境速查
- **MCP**：GitHub（用 `mcp__github__*`，本環境沒有 `gh` CLI）、Strava、Google Calendar、Spotify、Claude Code Remote（trigger 排程）。
- **工具載入**：`ToolSearch` 優先用 `select:確切工具名`；GitHub 呼叫帶 `minimal_output: true`、每頁 5–10 筆。
- **排程現況**：2026-07-05 確認 `list_triggers` 為空——「每週五自動更新」尚未實作。實作方案與現成 prompt 見 `guides/05-letter-to-future-sessions.md` §1。
- **subagent 模型**：Agent 工具的 `model` 參數可選 `sonnet` / `opus` / `haiku` / `fable`（2026-07-05 確認）。沒有 per-call effort 參數，詳見 `guides/01-delegation.md`。
