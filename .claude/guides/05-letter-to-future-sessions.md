# 給未來 session 的信

> 撰於 2026-07-05，由建立這套制度的 session 留下。
> 內容：三件使用者沒問、但對這個環境最重要的事；這套制度最可能的退化方式與預防法；以及 harness 的誠實極限。

## §1 第一件事：把「每週五自動更新」變成真的

meta 描述宣稱的自動更新不存在（`list_triggers` 為空）。這是本環境價值最高的未完成工程。做法：

用 `mcp__Claude_Code_Remote__create_trigger`，參數：
- `name`: `weekly-triathlon-radar-update`
- `cron_expression`: `0 9 * * 5`（**先與使用者確認**：這是伺服器時區的週五 09:00，未必等於台灣時間；建 trigger 前問使用者要台灣時間幾點，並實測伺服器時區換算）
- `create_new_session_on_fire`: `true`（每次全新 session，prompt 必須自足）
- `prompt`（可直接用，內含自足上下文）：

```
你在 triathlon-radar repo。先讀 CLAUDE.md 與 .claude/memory/LESSONS.md。
任務：更新 index.html 的每週鐵人三項訓練影片雷達。
步驟：
1. 取得 TrainingPeaks YouTube 頻道最近 7 天的影片清單（首選方法看 LESSONS.md 裡「YouTube 資料取得」條目；若條目不存在，嘗試 YouTube 頻道 RSS：https://www.youtube.com/feeds/videos.xml?channel_id=<ID>，成功後把可行方法與 channel_id 寫進 LESSONS.md）。
2. 為每支影片產生繁體中文摘要與訓練分類標籤（游泳/自行車/跑步/肌力/營養/恢復/綜合）。
3. 依 CLAUDE.md 硬規則更新 index.html（只用 Edit、不整檔覆寫），同步更新統計數字與「本週推薦」。
4. 依 .claude/guides/02-judgment.md §5 跑品質底線檢查。
5. commit 並 push 到 harness 指定的分支。
若 7 天內沒有新影片：只更新日期標記，如實註明本週無新片，不要編造內容。
若資料來源完全失敗：不要用假資料填充，在 LESSONS.md 記錄失敗方式後結束。
```

已知風險：直接 WebFetch youtube.com 頁面可能被擋或拿到殼頁。RSS feed（XML）通常可直接抓，且 entry 內含 `media:description` 可當摘要素材——**此法尚未實測**，第一次跑通的人請把結果（含 channel_id）寫進 LESSONS.md。

## §2 第二件事：重構 index.html 為資料驅動（根治診斷 D2）

目前每週更新要在 331 行的 HTML 裡改散落多處，是弱模型最容易改壞的操作。建議的重構（用 `03-prompt-templates.md` T3 模板派工）：

**目標結構**：所有每週變動的內容（影片清單、統計數字、本週推薦、更新日期）集中到一個 `<script type="application/json" id="radar-data">` 區塊，頁面 JS 讀取它 render 出卡片與統計。CSS 與版面骨架不再需要被碰。

**驗收條件**（直接填進 T3）：
1. 重構前 `Grep` 出所有影片標題、連結、統計數字存成基準清單；重構後在瀏覽器 render 的結果含有完全相同的清單。
2. 每週變動資料只存在於 `#radar-data` 一個區塊；`Grep` 任一影片標題在檔內只出現一次。
3. 之後的每週更新只需要一次 `Edit`（換掉 JSON 陣列內容）。
4. 頁面在無網路環境下仍能開啟（不引入外部資源——這是 artifact 的 CSP 限制）。

**完成後**：更新 `00-diagnosis.md` 把 D2 標為已解決（此動作依 `04-maintenance.md` §2 可自行做）。

## §3 第三件事：這個環境不只是這個 repo

使用者接了 Strava、Google Calendar、Spotify 的 MCP。合理推測（未與使用者確認）：他在乎自己的鐵人三項訓練。這代表兩個機會，**等使用者提起時**你已經知道怎麼做：
- 儀表板可以加「我的本週訓練」區塊：`mcp__Strava__list_activities` 拉近 7 天活動，與影片主題對照（例如本週騎車量低→推薦騎車影片）。
- 訓練排程可與 Calendar 對照。
不要主動做這些——那是範圍變更（`02-judgment.md` §3），但被問到「還能做什麼」時，這是答案。

## §4 這套制度最可能的退化方式與預防

1. **規則膨脹**：每次踩坑就加一條，guides 長到沒人讀完 → 判準被淹沒。
   預防：`04-maintenance.md` §4 的行數上限是硬的，到了就精簡，精簡優先於新增。
2. **規則過時但沒人敢改**：工具改名、模型清單變動後，照舊規則執行必然失敗，弱模型卻因「不准改規則」而卡死。
   預防：事實性修正已明確授權自行改（§2 權限分級），前提是實測。記住：**規則描述現實，現實變了規則就該改**；只有「判斷與門檻」才需要問使用者。
3. **教條化**：弱模型拿規則對抗使用者的明確指示（「規則說不能 Write 全檔所以我拒絕」）。
   預防：**使用者當下的明確指示永遠優先於本制度**。制度是使用者不在場時的代理判斷，不是凌駕使用者的憲法。使用者指示與硬規則衝突時，提醒一句後照使用者說的做，並記進 LESSONS。
4. **驗證形式化**：T5 審查淪為橡皮圖章（審查 agent 每次都放行）。
   預防：審查 prompt 不透露期望結論（T5 已內建）；若連續 5 次審查全數放行零退回，該懷疑的是審查品質，換 `opus` 審一次對照。

## §5 誠實條款：harness 的極限（拆解與驗證補不了的事）

- **品味與模糊判斷**：多候選＋評審能提升下限，到不了 Fable 等級的上限。視覺設計、文案語感、「推薦」的選片眼光，處理方式見 `02-judgment.md` §6：多候選 → `fable`/`opus` 評審 → 仍不確定就給使用者選並明說這是品味題。
- **`fable` 模型的可用性未經實測**：它出現在 Agent 工具的 model enum（2026-07-05），但本 session 未實際以它派工驗證方案支援。第一個用它的人：失敗就 fallback `opus`，並把結果寫進 LESSONS。
- **token 計量是推估**：診斷 D1–D3 的「代價」是依據結構推理，不是精確計量——harness 不提供 per-session token 帳單。方向可信，數字別引用。
- **YouTube 資料源未實測**（見 §1）。
- **cron 時區未實測**（見 §1）。
- 這封信的作者也會錯。發現本檔與現實矛盾時，相信實測，按 `04-maintenance.md` 修正。
