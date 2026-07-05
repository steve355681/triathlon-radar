# LESSONS — 踩坑與環境事實紀錄

> 格式與收錄標準見 `.claude/guides/04-maintenance.md` §3。新條目追加到檔尾。

## 2026-07-05 Agent 工具沒有 per-call effort 參數
情境：建立模型調度守則時實測 Agent 工具 schema。
教訓：委派時只能指定 `model`（haiku/sonnet/opus/fable），不能指定 reasoning effort；effort 只能寫在 `.claude/agents/<名字>.md` 的 frontmatter。不要在派工 prompt 裡浪費字要求「用高 effort」。
證據：Agent 工具 schema 的參數只有 description/prompt/model/subagent_type/isolation/run_in_background。

## 2026-07-05 index.html 已損壞：兩份文件黏在一起
情境：fresh-context 審查 agent 驗證品質底線判準時實測檔案。
教訓：第 228 行 `<script>` 未閉合就接上第二份 `<!DOCTYPE html>`（229–329 行整段被吞進 script），資料驅動版與最新資料（updatedAt 2026-06-20）不會渲染。「數標籤總數」抓不到這種損壞——要檢查開閉標籤嚴格交替＋DOCTYPE 唯一（已寫進 02-judgment.md §5）。修復任務見 05-letter §2。
證據：`grep -n '<script\|</script>' index.html` 顯示 228 開、下一筆閉合在 329；229 行結尾 `};<!DOCTYPE html>`。

## 2026-07-05 資料來源是 WebSearch，不是 meta 寫的 YouTube
情境：對照 index.html meta 描述與頁面實際文案。
教訓：meta（第 5 行）寫 YouTube 頻道，但頁面文案（226、304、326 行）與實際資料（文章/論文為主）都是「WebSearch 全網搜尋、每週五 20:00」。以 WebSearch 為準；改 meta 前先跟使用者確認意圖。
證據：index.html:226,304,326 vs index.html:5。

## 2026-07-05 subagent 的工具清單與主對話不同
情境：審查 agent 回報「AskUserQuestion 工具不存在」，但主對話明明有。
教訓：subagent 看不到主對話的部分工具（如 AskUserQuestion）。派工 prompt 不要假設 subagent 有你的工具；subagent 要問使用者的事寫進回報，由主對話轉問。
證據：2026-07-05 sonnet 審查 agent 以 ToolSearch 全查無果，主對話同時持有該工具。

## 2026-07-05 「每週五自動更新」從未真正排程
情境：盤點環境時查 `mcp__Claude_Code_Remote__list_triggers`。
教訓：index.html meta 描述的自動更新不存在，回傳為空 `{}`。要實作見 `.claude/guides/05-letter-to-future-sessions.md` §1，建好後回來更新本條。
證據：2026-07-05 `list_triggers` 回傳 `{}`。
