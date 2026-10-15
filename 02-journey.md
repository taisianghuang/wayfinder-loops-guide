# 02 · 一張票從想法到 main

> 本章給所有讀者。本章用一個虛構的例子,說明一個想法經過的每個階段。例子:PM 要讓客戶匯出月報 PDF。每個階段的標題後面是看板欄位。

---

## 全程時序

<div class="arch-embed" style="--h:690px;--h-wide:829px"><iframe src="diagrams/05-ticket-journey.html" loading="lazy" title="一張票從想法到 main"></iframe></div>
<p class="arch-open"><a href="diagrams/05-ticket-journey.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

## 各階段

### 1. 想法進來(💡 收件匣)

PM 在 planning repo 開一個 issue。內容可以只有一句話。這個 issue 沒有管線 label。`sync-project-status` 把它放進「💡 收件匣」。

### 2. 建立 map(🗺️ 規劃中)

cartographer 下一輪派一個 subagent。subagent 照 wayfinder 的流程,先確定目標,再列出所有未定問題:

| 問題 | 處理 | 例子 |
| --- | --- | --- |
| 能精確描述 | 開成決策票 | 「PDF 在請求時同步產生,或在背景產生後寄信通知?」(grilling);「三個 PDF 函式庫的授權與中文字型支援」(research) |
| 還不能精確描述 | 寫進 map 的霧 | 「之後可能要支援 Excel」 |

原始 issue 留言連到 map,然後關閉。

### 3. 查證與決策(🤖 查證中 → ⏳ 等你決定 → ✅ 已決定,待收斂)

1. researcher 處理 research 決策票。它貼回結論、來源與日期,然後關閉決策票。
2. PM 讀 grilling 決策票的決策封包:
   - 選項 A:同步產生。做法簡單。大報表會逾時。
   - 選項 B:背景產生。要多一個 job 與一個通知。
   - 建議:B。
   - 決策樹:選 B 時,下一個問題是「通知用 email 或站內訊息」。
3. PM 貼 `decision:b`。
4. resolver 在幾分鐘內記錄結論、關閉決策票,並在 map 留言「建議畢業的霧:通知管道」。

### 4. 霧清空,map 完成

cartographer 整理留言,把「通知管道」開成新的決策票。這個循環重複,直到決策票全部關閉、霧清空。

### 5. spec 與實作票(📋 排隊中 → 🚀 可實作)

cartographer 派一個 subagent。subagent 依序做:

1. 寫 spec,放在 planning repo。
2. 從 main 建整合分支 `spec/57-monthly-pdf`。
3. 在正確的 code repo 開實作票(後端的 job 與 API、前端的按鈕與狀態)。每張實作票第一行寫 `Integration-Branch: spec/57-monthly-pdf`。用 GitHub 原生 dependency 設定順序。
4. 開一張 integrate-and-verify 票。它被所有批次的實作票擋住(blocked-by)。
5. 最後貼 `agent:queued`。

`promote-queued` 把沒有 blocker 的實作票升級成 `agent:implement`。

### 6. 實作與審查(⚡ 實作中)

implementer 處理可以開始的實作票。每張實作票一個 subagent,各自一個 worktree。

| 誰 | 做什麼 |
| --- | --- |
| 實作 subagent | 讀 mattpocock 的 implement skill 並照做。可以時用 TDD。本地 CI 全綠後才 push。開 draft 批次 PR。 |
| reviewer | 審查 diff。直接 commit 修正。判斷單向門(例:這個批次 PR 改了資料庫 migration,是單向門)。重跑本地 CI。把批次 PR 設為 ready。 |

### 7. 收割(⚡ 實作中 → ✔️ 已關閉)

下一輪開始時,implementer 對每個 ready 的批次 PR 做以下步驟:

1. 在本地把批次 PR 合併到整合分支的最新 HEAD。
2. 重跑 CI。
3. CI 綠時,執行 `gh pr merge`。
4. 手動關閉實作票。

關閉實作票會觸發 `promote-queued`。被這張實作票擋住的實作票升級,下一批開始。

### 8. 整合與人審查(👀 待你 review)

所有批次的實作票關閉後,integrate-and-verify 票開始:

1. 把最新的 main 合併到整合分支。
2. 執行全部閘門。
3. 閘門綠時,開整合 PR,保持 draft,等人審查。

之後每一輪,main 有新的 commit 時,implementer 再合併 main 一次,並重跑閘門。整合 PR 的綠燈因此對應目前的 main。

工程師開啟整合 PR。PR 內文最上方是:

> **審這裡**:#214 `alembic/versions/…_add_report_jobs.py` — 新增資料表,最壞情況是 migration 在大表上鎖太久。其餘 5 張 PR 皆為雙向門。

工程師細看這個檔案,快速看過其他變更,然後合併。spec 票與 integrate-and-verify 票自動關閉。

## 一個 spec 人做幾次

| 誰 | 次數 | 做什麼 |
| --- | --- | --- |
| PM | 每張決策票 1 次 | 貼一個 label |
| 工程師 | 1 次 | 審查並合併整合 PR |
| agent | 其他全部 | 開票、查證、寫 spec、開實作票、實作、審查、收割、維護整合 PR |

> 下一步:[03 · 用看板管理](03-board.md)
