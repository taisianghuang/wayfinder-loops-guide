# 05 · 一張票從想法到 main

> 用一個虛構的例子走一遍:PM 想要「讓客戶能匯出月報 PDF」。

---

## 全程時序

<div class="arch-embed" style="--h:690px;--h-wide:829px"><iframe src="diagrams/05-ticket-journey.html" loading="lazy" title="一張票從想法到 main"></iframe></div>
<p class="arch-open"><a href="diagrams/05-ticket-journey.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

## 逐站說明

### 1. 想法進來(💡 收件匣)

PM 在 planning 開 issue,只寫一句話也可以。`sync-project-status` 看到 planning 上一張沒有管線 label 的 issue,把它放進「💡 收件匣」。

### 2. 畫地圖(🗺️ 規劃中)

Cartographer 下一輪派 subagent,照 wayfinder 的流程先釐清目的地,再廣度優先地列出所有待決問題:

- **能精確講出來的** → 開成決策票。例如「PDF 要在請求當下同步產生,還是背景產生後寄信通知?」(grilling)、「三個 PDF 函式庫的授權與中文字型支援」(research)。
- **還講不清楚的** → 寫進 map 的霧。例如「之後可能要支援 Excel」。

原始 issue 留言連到 map 後關閉。

### 3. 查證與決策(🤖 查證中 → ⏳ 等你決定 → ✅)

- Researcher 撿 research 票,附來源與日期貼回結論,關票。
- PM 在 grilling 票上讀封包:同步產生簡單但大報表會逾時;背景產生要多一個 job 與通知。封包建議 B(背景),並寫好「選 B 的話,下一題是通知用 email 還是站內訊息」。PM 點 `decision:b`。
- Resolver 幾分鐘內記錄結論、關票、到 map 留言「建議畢業的霧:通知管道」。

### 4. 霧散開、地圖熟成

Cartographer 收斂留言、把「通知管道」畢業成新的決策票……直到決策票全關、霧清空。

### 5. spec 與實作票(📋 排隊中 → 🚀 可實作)

Cartographer 派 subagent:

1. 寫 spec,發佈在 planning。
2. 從 main 建整合分支 `spec/57-monthly-pdf`。
3. 切實作票到對的 code repo(後端的 job 與 API、前端的按鈕與狀態),每張第一行寫 `Integration-Branch: spec/57-monthly-pdf`,用原生 dependency 串好順序。
4. 切一張 integrate-and-verify 票,blocked-by 全部批次票。
5. **最後**才貼 `agent:queued`。

`promote-queued` 把沒有 blocker 的票升成 `agent:implement`。

### 6. 實作與審查(⚡ 實作中)

Implementer 撿 frontier 票,一張一個 subagent、各自的 worktree:

- 實作 subagent:讀 mattpocock 的 implement 照做,能 TDD 就 TDD,本地 CI 全綠才 push,開 **draft** PR。
- reviewer:審 diff、直接 commit 修正、判斷「這張動到資料庫 migration → 單向門」、重跑本地 CI、轉 **ready**。

### 7. 收割(⚡ → ✔️)

下一輪開頭,Implementer 對每張 ready 的批次 PR 再驗一次:在本地把它合進整合分支的最新 HEAD、重跑 CI。綠了才 `gh pr merge`、手動關票。關票觸發 `promote-queued`,被它擋住的票升級,train 繼續往前。

### 8. 整合與人審(👀 待你 review)

所有批次票關閉後,integrate 票醒來:先把最新的 main 合進整合分支、跑全套閘門,綠了才開 spec→main PR,保持 draft 等人。之後每一輪,只要 main 又往前走,Implementer 就再合一次 main、重跑閘門,讓那張 PR 的綠燈永遠對應現在的 main。

工程師打開 PR,最上面寫著:

> **審這裡**:#214 `alembic/versions/…_add_report_jobs.py` — 新增資料表,最壞情況是 migration 在大表上鎖太久。其餘 5 張 PR 皆為雙向門。

細看那一個檔案,其餘快速掃過,合併。spec 票與 integrate 票自動關閉。

## 一份 spec 人動手幾次?

| 誰 | 次數 | 做什麼 |
| --- | --- | --- |
| PM | 每張決策票 1 次 | 點一個 label |
| 工程師 | 1 次 | 審並合併 spec→main PR |
| 機器 | 其餘全部 | 開票、查證、寫 spec、切票、實作、審查、收割、維護整合 PR |

> 下一步:[06 · 品質的三層把關](06-quality-gates.md)
