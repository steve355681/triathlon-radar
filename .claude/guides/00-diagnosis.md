# 環境診斷：三大漏洞與修法

> 撰於 2026-07-05，依據實際盤點結果（repo 內容、`~/.claude/`、MCP 工具清單、`list_triggers` 回傳）。
> 其他規則檔引用本檔時，用編號 D1 / D2 / D3。
> 若你發現本檔描述與現實不符（例如檔案結構已改變），依 `04-maintenance.md` 的規則更新本檔。

> **狀態更新 2026-07-05**：D1 的排程缺口與 D2 的檔案損壞已於同日修復——CLAUDE.md 與 LESSONS 已建立、每週 trigger 已建立（`trig_01K6j1UtHCWKPitUDVxABw6U`）、index.html 已重建為單一資料驅動文件（損壞現場備份在 `.claude/backups/20260705-index-corrupted.html`）。以下保留原始診斷供理解背景；「修法」中的規則仍然有效。

## D1｜冷啟動無記憶（最大的 token 漏損與錯誤來源）

**現況證據**：2026-07-05 盤點時，本 repo 沒有 CLAUDE.md、沒有任何教訓紀錄檔；`list_triggers` 回傳空——頁面宣稱的「每週五自動更新」排程實際上不存在，過去的更新都是人工開新 session，每次從零重新理解專案。注意：`index.html` 內部對資料來源的描述自相矛盾——meta 描述（第 5 行）寫「YouTube 頻道」，但頁面實際文案（第 226、304、326 行）寫「WebSearch 全網搜尋、每週五 20:00」，且現有資料多為文章與論文連結，非 YouTube 影片。**以 WebSearch 為實際現況**，meta 描述視為過時，待使用者確認意圖後修正。

**代價**：每個新 session 花數千至上萬 token 重新推斷專案結構、資料格式、慣例；弱模型推斷錯誤時直接產出錯誤結果（例如猜錯影片卡片的 HTML 結構、把繁體中文文案改成簡體或英文）。

**修法**：
1. 每個 session 開工前先讀 repo 根目錄的 `CLAUDE.md`（harness 會自動載入）與 `.claude/memory/LESSONS.md`。
2. 踩坑後把教訓寫進 `LESSONS.md`，格式與上限見 `04-maintenance.md` §3。
3. 若要真正實作每週五自動更新，用 `mcp__Claude_Code_Remote__create_trigger` 建 fresh-session routine；可直接套用的 prompt 在 `05-letter-to-future-sessions.md` §1。

## D2｜巨石 index.html：全檔讀寫

**現況證據**：`index.html` 331 行 / 約 26KB，每週資料、CSS、版面結構全部混在同一個檔案。**此風險已實際發生**（2026-07-05 確認）：檔案目前是損壞狀態——第 228 行的 `<script>` 沒有閉合就直接黏上第二份完整的 `<!DOCTYPE html>` 文件（229–329 行整段被吞進 script 內），資料驅動版本不會執行，最新一批資料（updatedAt 2026-06-20）不會顯示。修復任務見 `05-letter-to-future-sessions.md` §2。

**代價與風險**：
- 每次「讀整檔＋重寫整檔」約等於兩次全檔 token。
- 弱模型用 `Write` 全檔覆寫時，容易遺失它沒注意到的區塊（歷史上單檔 artifact 最常見的損壞方式）。
- 影片卡片彼此結構相似，`Edit` 的 `old_string` 錨點容易不唯一而失敗或改錯張卡片。

**修法（規則，立即生效）**：
1. 禁止對 `index.html` 用 `Write` 全檔覆寫。一律用 `Edit`，錨點必須包含該卡片獨有的文字（例如影片標題）。
2. 讀檔先用 `Grep` 定位行號，再用 `Read` 的 `offset`/`limit` 讀需要的區段，不整檔讀。
3. 結構性根治（建議的第一個工程任務，驗收條件見 `05-letter-to-future-sessions.md` §2）：把每週資料抽成檔內單一 JSON `<script>` 區塊、由頁面 JS render。之後每週更新＝只改一個 JSON 陣列，錨點唯一、token 成本降一個量級。

## D3｜工具海：schema 濫載與 list 呼叫不設限

**現況證據**：本環境約有 100 個 deferred tools（GitHub MCP 60+，另有 Strava、Google Calendar、Spotify、Claude Code Remote）。

**代價**：弱模型兩種常見浪費——(a) 用模糊關鍵字 `ToolSearch` 一次載入多個大 schema；(b) 呼叫 GitHub `list_*` / `search_*` 不帶分頁與 `minimal_output`，一次拉回大量 JSON 塞進主對話。

**修法（規則，立即生效）**：
1. `ToolSearch` 一律優先用 `select:確切工具名`；真的不確定名字才用關鍵字，且 `max_results` ≤ 3。
2. GitHub MCP 呼叫帶 `minimal_output: true`、每頁 5–10 筆。
3. Strava / Calendar / Spotify 的工具，只在任務明確涉及時才載入 schema。
4. 大量外部內容（網頁全文、PR diff、長 log）不直接進主對話：先落檔到 scratchpad，再用 `Grep` / `Read` 區段查需要的部分。

## 次要但必須知道

- **分支紀律**：harness 每個 session 會指定一個 `claude/*` 分支。只在該分支開發與 push，不碰 `main`。
- **驗證劇場**：弱模型常宣稱「已完成」但沒有任何工具結果佐證。完成判準與驗法見 `02-judgment.md` §2、§5。
- **hooks 不可動**：`~/.claude/` 下的 hook script 與 launcher 設定由平台管理，不要修改。
