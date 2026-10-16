# 00 · 總覽

> 本章給所有讀者。本章說明 wayfinder-loops 是什麼、整體流程、本報告書的名詞,與各章的讀法。

---

## wayfinder-loops 是什麼

wayfinder-loops 是一個 Claude Code plugin。它用 GitHub Issues 與 GitHub Project 看板,把一個想法推進到合併進 main 的程式碼。

wayfinder-loops 有五條 loop 與一個 reviewer agent。agent 做開 ticket、查證、實作、審查與 harvest。人只做兩件事:

1. 在 decision ticket 上選一個選項。
2. 審查並合併 integration PR。

wayfinder-loops 使用 [mattpocock/skills](https://github.com/mattpocock/skills) 的方法、Claude Code `/loop`、GitHub Issues、GitHub Project 看板與三個 GitHub Actions workflow。

## 一句話

PM 提出想法並做決定。agent 開 ticket、查證、實作、審查並 harvest。工程師只審查一次。

## 流程

<div class="arch-embed" style="--h:688px;--h-wide:827px"><iframe scrolling="no" src="diagrams/00-pipeline.html" loading="lazy" title="wayfinder-loops 操作管線"></iframe></div>
<p class="arch-open"><a href="diagrams/00-pipeline.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

## 名詞

本報告書中每個名詞只有一個意思。

| 名詞 | 意思 |
| --- | --- |
| planning repo | 放想法、map、decision ticket、spec 與 `wayfinder.yml` 的 repo。 |
| code repo | 放程式碼、implementation ticket 與 PR 的 repo。可以有多個。 |
| map | 記錄一個想法的目標、已做的決定與未定問題的 issue。只有 cartographer 寫 map 的內文。 |
| fog | map 中在範圍內、但還不能精確描述的問題。能精確描述時,它變成一張 decision ticket。 |
| decision ticket | map 中一個問題的 issue。人做決定,或 researcher 查證事實。種類見 [03 章](03-board.md#decision-ticket-的種類)。 |
| decision packet | decision ticket 上給人的資料:選項表、建議、決策樹、回覆方法。 |
| spec | map 的 decision ticket 全部關閉、fog 清空後,cartographer 寫的規格。放在 planning repo。 |
| implementation ticket | 要求 agent 實作一項變更的 issue。放在 code repo。 |
| integrate-and-verify ticket | 一個 spec 的最後一張 implementation ticket。它把 main 合併到 integration branch、執行全部閘門、開 integration PR。 |
| integration branch | 一個 spec 的所有變更先合併到的分支,名稱為 `spec/<編號>-<名稱>`。 |
| batch PR | implementation ticket 的 PR。目標分支是 integration branch。人不審查 batch PR。 |
| harvest | implementer 再驗證 batch PR、把它合併到 integration branch、關閉 implementation ticket 的動作。 |
| integration PR | 從 integration branch 到 main 的 PR(spec→main 的 PR)。每個 spec 只有一個。人審查這個 PR。 |
| one-way door | 合併後不能撤回的變更。預設清單:資料庫 migration、資料刪除或不可逆改寫、權限與認證、對外發送(email、webhook、推播)、金流、公開 API 契約。 |
| two-way door | 合併後可以撤回、影響範圍小的變更。 |
| claim | loop 在 issue 上貼 `agent:in-progress`。表示這條 loop 在處理這張 issue。 |
| HITL | human in the loop。需要人決定的工作。沒有人的新動作時,loop 不處理它,也不逾時自動核准。 |
| AFK | away from keyboard。agent 可以獨立完成、不需要人的工作。 |

## 讀法

| 段 | 章 | 讀者 | 內容 |
| --- | --- | --- | --- |
| 認識 | [01 · 六個角色](01-roles.md)、[02 · 一張 ticket 從想法到 main](02-journey.md) | 所有人 | 每個角色的工作;一個想法經過的每個階段 |
| 管理 | [03 · 用看板管理](03-board.md) | PM、SA、工程師 | 看板欄位、人的三個動作、label |
| 建立 | [04 · 建立工作流](04-setup.md)、[05 · 啟動與疑難排解](05-operations.md) | 導入與維運的人 | 安裝與設定步驟;啟動 loop、處理問題 |

> 下一步:[01 · 六個角色](01-roles.md)
