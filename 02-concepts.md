# 02 · 核心概念

> 讀懂這一篇的十個名詞,後面每一篇都會順很多。

---

## 三種 repo

```
<org>/planning   💡 想法 → 🗺️ map → 決策票 → spec        ← PM / SA 唯一需要的入口
                           │ 切實作票時才決定歸哪個 code repo
                           ▼
<org>/api        實作票 → 批次 PR → 整合分支 → spec→main PR   ← 機器與工程師
<org>/web        (code repo 可以有很多個)
```

- **planning repo**:想法、地圖、決策票、spec 全部住這裡。PM 不需要知道功能歸前端還是後端,那是管線的產出,不是入場費。
- **code repo**:只放實作票與 PR。既有的 backlog 筆記 issue 不受影響,Cartographer 只掃 planning。

## map(地圖)與 fog(霧)

一個模糊的想法會被 Cartographer 用 mattpocock 的 wayfinder skill 開成一張 **map issue**。map 的 body 有五個區塊:

| 區塊 | 意思 |
| --- | --- |
| Destination | 這個想法要抵達哪裡:一份 spec、一個要先鎖定的決定,或一次就地的改動 |
| Notes | 解題時要參考的 skill、文件與背景 |
| Decisions so far | 已經走過的路:定案的決定(由 Cartographer 收斂寫入) |
| Not yet specified | **fog**:在範圍內、但現在還講不清楚的問題 |
| Out of scope | 刻意排除在這次之外的事;永遠不會畢業成票 |

每個決定都可能讓一部分霧散開。當某個問題「能精確講出來」了,它就**畢業**成一張新的決策票。霧散光、決策票全關,map 就熟了,可以寫 spec。

## 決策票的四種類型

| Label | 誰處理 | 意思 |
| --- | --- | --- |
| `wayfinder:grilling` | 人(HITL) | 需要人拍板的設計問題 |
| `wayfinder:prototype` | 人說「畫」→ Prototyper | 文字講不清楚,先做原型看看 |
| `wayfinder:research` | Researcher(AFK) | 查證外部事實:API 限制、價格、文件 |
| `wayfinder:task` | 依內容 | 明確可 AFK 的雜工歸 Researcher;需要人動手的歸人 |

所有決策票另外帶 `wayfinder:ticket`,方便一次撈出。

## 決策封包

每張要人決定的票都附一份決策封包,讓人**讀完就能點**:

1. **選項表**:做法、代價、之後變簡單的事、之後變難的事
2. **明確建議**:引用 repo 裡既有的 ADR、詞彙表或程式碼
3. **決策樹**:「如果你選 A,下一個問題會是…」
4. **怎麼回**:貼 `decision:a` / `b` / `c` / `approve` / `discuss`,或直接留言

選項超過三個時,先找一條正交的軸拆成兩層決策;超過五個候選,那是 shortlist 問題,先開 research 票收斂。

## HITL 與 AFK

- **HITL**(human in the loop):沒有人的新動作,所有 loop 對它完全不動作。**永不逾時自動核准。**
- **AFK**(away from keyboard):機器可以獨立完成,不需要人。

## spec 與 spec-train

map 熟了之後,Cartographer 用 mattpocock 的 to-spec 寫成 spec,再用 to-tickets 切成實作票。系統只有一種實作模式,叫 **spec-train**:

1. 從最新的 main 建一條**整合分支** `spec/<票號>-<slug>`。
2. 每張實作票 body 第一行寫 `Integration-Branch: spec/<...>`,它的 PR base 就是這條分支,不是 main。
3. 多切一張 **integrate-and-verify 票**,blocked-by 全部的批次票。它負責把 main 合進整合分支、跑全套測試、開 **spec→main** 的 PR 給人審。

<div class="arch-embed" style="--h:683px;--h-wide:821px"><iframe src="diagrams/02-spec-train-branches.html" loading="lazy" title="spec-train 分支流向"></iframe></div>
<p class="arch-open"><a href="diagrams/02-spec-train-branches.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

好處是:一份 8 張票的 spec,人只需要審一次、合併一次,而不是八次。

## 批次 PR 與收割

- **批次 PR**:實作票的 PR,base 是整合分支。人**不審**這些 PR。
- **收割**:Implementer 每一輪開頭做的事。把通過驗證的批次 PR 一張一張合進整合分支、手動關票。關票會觸發 `promote-queued`,被它擋住的下一批票才會醒。

## 單向門與雙向門

借用 AWS 的說法:

- **雙向門(two-way door)**:做錯了可以退回,影響範圍小。審查可以輕。
- **單向門(one-way door)**:做了就回不去。預設清單是資料庫 migration、資料刪除或不可逆改寫、權限與認證、對外發送(email、webhook、推播)、金流、公開 API 契約。審查要嚴。

reviewer 對每張批次 PR 判斷 door,整合 PR 最上面彙整成「**審這裡**」清單。

## 認領鎖

五條 loop 共用同一個 GitHub 帳號,所以不能用「誰留的言」分辨身分。系統用兩個約定:

- 認領 = 貼 `agent:in-progress` label,**不是** assign 自己。assignee 欄位保留給「輪到人類」。
- loop 發的每則留言都以 `🤖 [loop名]` 開頭;**沒有這個前綴的留言一律視為人類發言**。

> 下一步:[03 · 人類的一天](03-human-day.md)
