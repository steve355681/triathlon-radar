# LESSONS — 踩坑與環境事實紀錄

> 格式與收錄標準見 `.claude/guides/04-maintenance.md` §3。新條目追加到檔尾。

## 2026-07-05 Agent 工具沒有 per-call effort 參數
情境：建立模型調度守則時實測 Agent 工具 schema。
教訓：委派時只能指定 `model`（haiku/sonnet/opus/fable），不能指定 reasoning effort；effort 只能寫在 `.claude/agents/<名字>.md` 的 frontmatter。不要在派工 prompt 裡浪費字要求「用高 effort」。
證據：Agent 工具 schema 的參數只有 description/prompt/model/subagent_type/isolation/run_in_background。

## 2026-07-05 「每週五自動更新」從未真正排程
情境：盤點環境時查 `mcp__Claude_Code_Remote__list_triggers`。
教訓：index.html meta 描述的自動更新不存在，回傳為空 `{}`。要實作見 `.claude/guides/05-letter-to-future-sessions.md` §1，建好後回來更新本條。
證據：2026-07-05 `list_triggers` 回傳 `{}`。
