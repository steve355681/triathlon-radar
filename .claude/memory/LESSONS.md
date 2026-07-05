# LESSONS — 踩坑與環境事實紀錄

> 格式與收錄標準見 `.claude/guides/04-maintenance.md` §3。新條目追加到檔尾。

## 2026-07-05 Agent 工具沒有 per-call effort 參數
情境：建立模型調度守則時實測 Agent 工具 schema。
教訓：委派時只能指定 `model`（haiku/sonnet/opus/fable），不能指定 reasoning effort；effort 只能寫在 `.claude/agents/<名字>.md` 的 frontmatter。不要在派工 prompt 裡浪費字要求「用高 effort」。
證據：Agent 工具 schema 的參數只有 description/prompt/model/subagent_type/isolation/run_in_background。

## 2026-07-05 index.html 曾損壞：兩份文件黏在一起（已修復）
情境：fresh-context 審查 agent 驗證品質底線判準時實測檔案。
教訓：`<script>` 未閉合就接上第二份 `<!DOCTYPE html>`，資料驅動版不會渲染。「數標籤總數」抓不到這種損壞——要檢查開閉標籤嚴格交替＋DOCTYPE 唯一（已寫進 02-judgment.md §5）。推測損壞原因：某次更新把「資料 script＋新版模板」整段 append 到舊版文件之後，而不是取代。同日已修復為單一資料驅動文件；損壞現場在 `.claude/backups/20260705-index-corrupted.html`。
證據：損壞版 `grep -n '<script\|</script>'` 顯示 228 開、下一筆閉合在 329；修復驗證＝headless Chromium dump-dom render 出 5 張卡片。

## 2026-07-05 驗 render 用 headless Chromium；grep -c 數行不數次
情境：驗證修復後的 index.html。
教訓：`/opt/pw-browsers/chromium --headless --no-sandbox --dump-dom file://<路徑>` 可直接拿到 JS 執行後的 DOM，用來確認卡片真的渲染出來。注意 `grep -c` 數的是「含匹配的行數」，dump-dom 輸出常擠在同一行——數出現次數要用 `grep -o … | wc -l`。
證據：同一檔案 `grep -c` 回 1、`grep -o | wc -l` 回 5。

## 2026-07-05 資料來源是 WebSearch，不是 meta 寫的 YouTube
情境：對照 index.html meta 描述與頁面實際文案。
教訓：meta 原寫 YouTube 頻道，但頁面文案與實際資料（文章/論文為主）都是「WebSearch 全網搜尋、每週五 20:00」。以 WebSearch 為準；meta 描述已於 2026-07-05 修復時同步改正。
證據：損壞版 index.html:226,304,326 vs 第 5 行（見 `.claude/backups/20260705-index-corrupted.html`）。

## 2026-07-05 subagent 的工具清單與主對話不同
情境：審查 agent 回報「AskUserQuestion 工具不存在」，但主對話明明有。
教訓：subagent 看不到主對話的部分工具（如 AskUserQuestion）。派工 prompt 不要假設 subagent 有你的工具；subagent 要問使用者的事寫進回報，由主對話轉問。
證據：2026-07-05 sonnet 審查 agent 以 ToolSearch 全查無果，主對話同時持有該工具。

## 2026-07-05 每週自動更新已建立（原本從未排程）
情境：盤點時發現 meta 宣稱的自動更新不存在（`list_triggers` 回傳 `{}`），同日建立。
教訓：trigger `trig_01K6j1UtHCWKPitUDVxABw6U`，cron `0 12 * * 5`（伺服器 UTC，＝台灣週五 20:00），fresh-session 模式，push 通知開啟。調整或重建照 05-letter §1。伺服器時區實測為 UTC（`date +%z` 回 `+0000`），排 cron 前一律先實測。
證據：create_trigger 回傳，next_run_at 2026-07-10T12:04Z。
