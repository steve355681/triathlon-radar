# CLAUDE.md — system design session

## 這個 repo 是什麼
本 repo 有兩個身分（GitHub 名稱：`system-design-session`；2026-07-06 前舊名 `triathlon-radar`，若你看到的還是舊名，代表使用者尚未在 GitHub 完成改名，內容不變照用）：
1. **Steve 的 Claude session 治理制度中央 repo**：`.claude/guides/` 全套制度可移植到其他 repo。使用者在任何新 repo 的 session 說「照 system design session 的制度移植」，該 session 就照 `guides/05-letter-to-future-sessions.md` §5 執行移植。
2. **鐵人三項雷達儀表板**：單一產品檔案 `index.html`（繁體中文單頁儀表板）：鐵人三項訓練內容的每週雷達，呈現 STAR 結構摘要、訓練分類標籤、本週推薦。資料來源是 **WebSearch 全網搜尋**（文章/論文/影片）。使用者是 Steve（繁體中文使用者、鐵人三項訓練者）。repo 裡沒有 build 系統、沒有測試框架——驗證方式是 headless Chromium render 這個 HTML（`/opt/pw-browsers/chromium --headless --no-sandbox --dump-dom file://…`）。
檔案是資料驅動結構：**每週更新＝一次 `Edit` 把 `const DATA_PLACEHOLDER = {…};` 整段換成新資料**，其餘部分（CSS、render 邏輯）不需要碰。

## 硬規則（違反即算任務失敗）
1. 回覆與 UI 文案一律繁體中文（程式碼、commit message 用英文可以）。
2. `index.html` 禁止用 `Write` 全檔覆寫。只用 `Edit`，`old_string` 錨點必須含該處獨有文字。讀檔先 `Grep` 定位，再用 `Read` 的 offset/limit 讀區段。（唯一例外：修復已損壞的檔案，規範見 `guides/05-letter-to-future-sessions.md` §2。）
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
- **排程現況**：每週自動更新已於 2026-07-05 建立（trigger `trig_01ErnJ4TnKhxvXyLqT3pQf2W`，cron `0 12 * * 5` UTC＝台灣週五 20:00，fresh-session 模式）。內容見 `guides/05-letter-to-future-sessions.md` §1。
- **subagent 模型**：Agent 工具的 `model` 參數可選 `sonnet` / `opus` / `haiku` / `fable`（2026-07-05 確認）。沒有 per-call effort 參數，詳見 `guides/01-delegation.md`。
