# 10 · 設計取捨與踩過的坑

> 這套工作流的大部分規則,都是跑真實專案時踩到坑之後補上的。這一篇記錄「為什麼是這樣」,給想修改它的人。

---

## 身分與鎖

### 認領用 label,不用 assignee

wayfinder 原本把「assign 自己」當成認領。但這套系統裡 assignee 有另一個意思:「輪到人類」。兩個意思擠在同一個欄位,看板就分不出「機器在做」和「等人決定」。所以認領一律貼 `agent:in-progress`,assignee 只留給人。

### 用前綴分辨人與機器

五條 loop 共用同一個 GitHub 帳號,留言作者全部一樣。約定 loop 留言一律以 `🤖 [loop名]` 開頭,沒有前綴就是人。Resolver 判斷「人類已出手」完全靠這個。

### 不要多開同名 loop

想讓實作更平行,直覺是多開幾條 implementer。但 label 鎖沒有身分資訊:兩個 session 同時貼 `agent:in-progress`,各自「貼完再讀一次」都會看到 label 在,都以為是自己貼的。結果同一張票被實作兩次。平行度改由**單一 loop 依序認領、平行派工**取得,競態根本不存在。

## 狀態與事件

### 看板只是投影

任何 loop 都不讀看板欄位做判斷。看板由 `sync-project-status` 單向從 label 算出來,人拖卡片沒有效果。這讓「真相只有一個」成立,也讓看板可以隨時重建。

### `decision:discuss` 是門鈴,不是旗子

最早 discuss 跟 a/b/c 一樣是「留著的 label」,結果卡片顯示成「已決定」,而且 Resolver 每輪都重新命中、重複回覆。改成門鈴語意:label 在 = agent 欠人一則回覆;Resolver 回覆後拔掉,卡片落回「等你決定」。

### 留言裡的決定也要算

Resolver 原本只認 label。人在留言寫「選 C」,被當成非終止型 → 回覆但不關票、反覆要求貼 label,票就卡住了。改成:留言明確指名恰好一個選項,Resolver 轉錄成 label(引用原句)再結案;拿不準就當成想討論。

### 出生就沒有 blocker 的票

`promote-queued` 原本只聽「票關閉」事件。新 spec 切出的第一批票出生就沒有 blocker,沒有任何關閉事件會叫醒它們,每份 spec 都在起點停擺。補上兩件事:聽「貼上 `agent:queued`」事件(等 45 秒讓依賴串好),以及 Cartographer 必須在依賴串好**之後最後**才貼 `agent:queued`。`workflow_dispatch` 手動重掃仍是通用救援。

### parent 在 planning 是正常的

早期規則是「子票一律不自動升級」,因為子票通常是決策票的拆解產物。但 spec-train 的實作票就是掛在 planning 的 spec 票底下,這條規則一度把整批正常的實作票全部退回。現在的規則:parent 在 planning → 放行;parent 在其他地方 → 退回。workflow 與 Implementer 的防呆必須用同一條線,不然票會在兩個狀態之間彈跳。

## spec-train

### 為什麼只有一種模式

早期有兩種模式:每張票各自 PR 進 main(ticket-stream),或收進整合分支(spec-train)。ticket-stream 讓每張票都變成人的停等點:一份 8 張票的 spec 要等四波合併,和「人只認證一次」的目標相反。而且模式二選一時,implementer 讀不到標記就預設走 main,Cartographer 漏寫就默默走錯。

現在只有 spec-train。實作票第一行沒有 `Integration-Branch:` → 退回,**絕不預設走 main**。

### `Closes` 的三個坑

1. **PR 合進非預設分支時,`Closes #N` 不會關票。** 所以批次 PR 只寫 `Part of #N`,關票由收割手動做;手動關票才會觸發 `promote-queued`。
2. **`Closes` 包在反引號或 code block 裡,GitHub 不會關票。** 實測過,train 因此停住。一律寫純文字、獨立一行。
3. **跨 repo 的 spec 有多張 integrate 票,只有最後一張能寫 `Closes <planning>#<spec>`。** 否則先合併的那張就把 spec 票關掉,看板顯示交付完成,其實另一個 repo 的半份還沒進 main。判斷方法:這張 integrate 票還擋著別的票,就不是最後一張。

### PR 開完後認領鎖保留

舊規則在 PR 開完就拔 `agent:in-progress`。結果「PR 開了但還沒收割」的票同時符合撿票三條件,下一輪被重做一次。現在保留到收割關票,看板也正確顯示「⚡ 實作中」。

### 整合 PR 的綠燈會過期

GitHub 不會因為 main 往前走而重跑 PR 上的 check。兩份 spec 並行時,先合併的那份一動,後面那張 PR 的綠燈就是對舊 main 的綠燈。Implementer 每輪檢查落後 main 幾個 commit,> 0 就合 main、重跑閘門。

### 產生檔的衝突不要手解

兩份 spec 都會重新產生 API client 這類產生檔,這通常是它們唯一真正相撞的地方。設定 `generated[]` 之後,衝突一律取 main 的版本再重新產生,不讓它留到人的認證 PR 上。

## context 管理

### loop 不做重活

loop session 的 context 會隨輪數累積,主因是查證網頁、讀程式碼、建圖探索的殘渣。現在所有重活都交給 subagent,loop 只做掃描、認領、驗收、回報。subagent 看不到 loop 的契約,所以契約(前綴、認領規則、禁令)必須寫進每個 subagent 的 prompt。

### 實作者不守標準

實作 subagent 的 context 壓力最大。把 coding standards 塞給它,它守不住,還會擠掉真正需要的東西。標準交給只拿到 diff 的 reviewer。

## reviewer 為什麼是 agent 不是 loop

考慮過把 reviewer 做成第六條獨立 loop,最後沒有:

| 考量 | agent(現行) | 獨立 loop |
| --- | --- | --- |
| 延遲 | PR 一開好就派,同一輪內審完 | 每張 PR 平均多等一輪 |
| 機器負載 | Implementer 一個人管「同時最多 K 個 CI」 | 兩條 loop 各自派工,並行數沒人控管 |
| 鎖 | 同一個 session 依序派工,沒有競態 | 要新增 label 與「收割 vs 審查」的鎖 |
| 交接 | 「ready = 審過」,reviewer 中途掛掉由補驗路徑接手 | 跨 loop 的契約變多 |
| 顯眼程度 | 要在文件上明講它存在 | 天生在看板與回報上可見 |

代價是 reviewer 不夠顯眼,所以文件一律寫成「五條 loop 加一個 reviewer agent」。

## 選項數紀律

`decision:*` label 只開到 c。選項超過三個時:先找一條正交的軸拆成兩層決策(自建 vs 採購、同步 vs 非同步);真的是攤平的同儕選項,照列 A–E 改用留言回覆;超過五個候選,那是 shortlist 問題,先開 research 票收斂到三個以內。

> 下一步:[11 · 路線圖](11-roadmap.md)
