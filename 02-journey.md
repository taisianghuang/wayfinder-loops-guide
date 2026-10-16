# 02 · 一張 ticket 從想法到 main

> 本章給所有讀者。本章說明一個功能需求從 issue 到合併進 main 經過的每個階段。每個階段的標題後面是看板欄位。

---

## 全程時序

<div class="arch-embed" style="--h:690px;--h-wide:829px"><iframe scrolling="no" src="diagrams/05-ticket-journey.html" loading="lazy" title="一張 ticket 從想法到 main"></iframe></div>
<p class="arch-open"><a href="diagrams/05-ticket-journey.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

## 各階段

### 1. 想法進來(💡 收件匣)

PM 在 planning repo 開一個 issue,提出一個功能需求。內容可以只有一句話。這個 issue 沒有管線 label。`sync-project-status` 把它放進「💡 收件匣」。

### 2. 建立 map(🗺️ 規劃中)

cartographer 下一輪派一個 subagent。subagent 照 wayfinder 的流程,先確定目標,再列出所有未定問題:

| 問題 | 處理 |
| --- | --- |
| 能精確描述,要人選擇 | 開成 grilling decision ticket |
| 能精確描述,要查外部事實 | 開成 research decision ticket |
| 還不能精確描述 | 寫進 map 的 fog |

原始 issue 留言連到 map,然後關閉。

### 3. 查證與決策(🤖 查證中 → ⏳ 等你決定 → ✅ 已決定,待收斂)

1. researcher 處理 research decision ticket。它貼回結論、來源與日期,然後關閉 decision ticket。
2. PM 讀 grilling decision ticket 的 decision packet:
   - 選項表:每個選項的做法與代價。
   - 建議:其中一個選項,與理由。
   - 決策樹:選每個選項之後,下一個問題是什麼。
3. PM 貼 `decision:b`。
4. resolver 在幾分鐘內記錄結論、關閉 decision ticket,並在 map 留言,列出可以畢業的 fog。

### 4. fog 清空,map 完成

cartographer 整理留言,把可以畢業的 fog 開成新的 decision ticket。這個循環重複,直到 decision ticket 全部關閉、fog 清空。

### 5. spec 與 implementation ticket(📋 排隊中 → 🚀 可實作)

cartographer 派一個 subagent。subagent 依序做:

1. 寫 spec,放在 planning repo。
2. 從 main 建 integration branch `spec/<編號>-<名稱>`。
3. 在正確的 code repo 開 implementation ticket(例:後端的 API、前端的畫面)。每張 implementation ticket 第一行寫 `Integration-Branch: spec/<編號>-<名稱>`。用 GitHub 原生 dependency 設定順序。
4. 開一張 integrate-and-verify ticket。它被所有批次的 implementation ticket 擋住(blocked-by)。
5. 最後貼 `agent:queued`。

`promote-queued` 把沒有 blocker 的 implementation ticket 升級成 `agent:implement`。

### 6. 實作與審查(⚡ 實作中)

implementer 處理可以開始的 implementation ticket。每張 implementation ticket 一個 subagent,各自一個 worktree。

| 誰 | 做什麼 |
| --- | --- |
| 實作 subagent | 讀 mattpocock 的 implement skill 並照做。可以時用 TDD。本地 CI 全綠後才 push。開 draft batch PR。 |
| reviewer | 審查 diff。直接 commit 修正。判斷 one-way door(例:這個 batch PR 改了資料庫 migration,是 one-way door)。重跑本地 CI。把 batch PR 設為 ready。 |

### 7. harvest 階段(⚡ 實作中 → ✔️ 已關閉)

下一輪開始時,implementer 對每個 ready 的 batch PR 做以下步驟:

1. 在本地把 batch PR 合併到 integration branch 的最新 HEAD。
2. 重跑 CI。
3. CI 綠時,執行 `gh pr merge`。
4. 手動關閉 implementation ticket。

關閉 implementation ticket 會觸發 `promote-queued`。被這張 implementation ticket 擋住的 implementation ticket 升級,下一批開始。

### 8. 整合與人審查(👀 待你 review)

所有批次的 implementation ticket 關閉後,integrate-and-verify ticket 開始:

1. 把最新的 main 合併到 integration branch。
2. 執行全部閘門。
3. 閘門綠時,開 integration PR,保持 draft,等人審查。

之後每一輪,main 有新的 commit 時,implementer 再合併 main 一次,並重跑閘門。integration PR 的綠燈因此對應目前的 main。

工程師開啟 integration PR。PR 內文最上方是:

> **審這裡**:#<PR 編號> `<檔案>` — 這個 batch PR 新增一個資料表。最壞情況:migration 在大表上鎖太久。其餘 batch PR 都是 two-way door。

工程師細看這個檔案,快速看過其他變更,然後合併。spec ticket 與 integrate-and-verify ticket 自動關閉。

## 一個 spec 人做幾次

| 誰 | 次數 | 做什麼 |
| --- | --- | --- |
| PM | 每張 decision ticket 1 次 | 貼一個 label |
| 工程師 | 1 次 | 審查並合併 integration PR |
| agent | 其他全部 | 開 ticket、查證、寫 spec、開 implementation ticket、實作、審查、harvest、維護 integration PR |

> 下一步:[03 · 用看板管理](03-board.md)
