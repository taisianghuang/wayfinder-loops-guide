# 01 · 六個角色

> 本章給想知道 agent 在做什麼的人。本章說明五條 loop、一個 reviewer agent 與三個 GitHub Actions workflow 的工作。

---

## 角色一覽

| 角色 | 種類 | 頻率 | 工作 |
| --- | --- | --- | --- |
| cartographer | loop | 20 分鐘 | 把想法寫成 map 與 decision ticket。決策完成後,寫 spec 與 implementation ticket。 |
| resolver | loop | 10 分鐘 | 讀人在 decision ticket 上的選擇。記錄結論並關閉 decision ticket。 |
| researcher | loop | 15 分鐘 | 查證外部事實。把結果寫在 decision ticket 上並關閉。 |
| implementer | loop | 15 分鐘 | harvest、claim implementation ticket、平行派工、派出 reviewer、維護 integration PR。 |
| prototyper | loop | 30 分鐘 | 人要求原型時,做原型頁與截圖。 |
| reviewer | agent | implementer 派出 | 審查每個 batch PR。修正問題,判斷 one-way door,把 PR 設為 ready。 |

## 每一輪的步驟

每條 loop 依頻率執行一輪。每一輪依序做以下步驟:

1. 讀 `wayfinder.yml`。讀不到時,停止。
2. 掃描:用 `gh` 查 label,找出要處理的 ticket。沒有 ticket 時,跳到步驟 6。
3. claim:貼 `agent:in-progress`。
4. 派工:把工作交給 subagent。
5. 驗收:到 GitHub 核對結果。
6. 回報:列出還在等人的 ticket。

各步驟的規則:

| 項目 | 規則 |
| --- | --- |
| 讀設定 | 每一輪開始時,loop 從 planning repo 讀 `wayfinder.yml`。讀不到時,loop 停止。loop 不使用預設值。 |
| 派工 | 讀檔、查證、寫程式由 subagent 做。loop 只做掃描、claim、驗收、回報。loop session 的 context 不隨輪數增加。 |
| 驗收 | loop 不相信 subagent 的回報。loop 到 GitHub 確認 PR、label 與留言存在。 |
| subagent 失敗 | loop 移除 claim,並留言記錄進度。 |
| claim | loop 貼 `agent:in-progress`。loop 不把 issue 指派給自己。assignee 欄位只用來表示「輪到人」。 |
| 留言前綴 | loop 的每則留言以 `🤖 [loop 名稱]` 開頭。沒有這個前綴的留言是人的留言。 |

五條 loop 用同一個 GitHub 帳號。所以 loop 用 label 與留言前綴分辨身分,不用留言作者。

## cartographer(20 分鐘)

cartographer 是唯一寫 map 內文的角色。其他角色只在 map 上留言。cartographer 把留言整理進 map 內文。

一輪依序做以下五件事:

1. **檢查 WIP 上限。** 等人決定的 decision ticket 有 5 張以上時,這一輪不開新的 HITL decision ticket。開 ticket 速度不超過人的決定速度。
2. **整理 map 留言。** 把 resolver、researcher、implementer 的回報與人的方向留言寫進 Decisions so far,並更新 fog。
3. **把 fog 變成 decision ticket。** 能精確描述的問題,開成新的 decision ticket。HITL decision ticket 附 decision packet,並指派給人。AFK decision ticket 不指派,由 researcher 處理。
4. **處理新想法。** planning repo 中沒有管線 label 的 issue 是新想法。每個想法派一個 subagent,開成 map 與第一批 decision ticket。
5. **把完成的 map 寫成 spec 與 implementation ticket。** 條件:decision ticket 全部關閉,fog 清空。subagent 寫 spec、在正確的 code repo 開 implementation ticket、建 integration branch、開 integrate-and-verify ticket。依賴設定完成後,最後才貼 `agent:queued`。

## resolver(10 分鐘)

resolver 的頻率要高於 cartographer。原因:decision ticket 關閉後,才出現新的 fog。人的每個決定要在幾分鐘內處理。

resolver 找出每張 decision ticket 的最後一則人的留言,並判斷種類:

| 種類 | 例子 | resolver 的動作 |
| --- | --- | --- |
| 終止型 | `decision:*` label;留言指出一個選項 | 記錄結論、關閉 decision ticket(觸發 `promote-queued`)、通知 map |
| 非終止型 | `decision:discuss`、問題、猶豫 | 回覆一則留言,移除 `decision:discuss`。一則人的留言只回覆一則。 |

resolver 不替人決定。例外:人在留言中已指出一個選項時,resolver 把它轉成 label,並引用原句。

## researcher(15 分鐘)

researcher 只做 AFK 工作。一輪最多處理 2 張 decision ticket。

- researcher 處理 `wayfinder:research` decision ticket,與內容可以獨立完成的 `wayfinder:task` decision ticket。有任何一步需要人(註冊帳號、付費、操作 UI)時,researcher 不處理。
- 每個重要事實附來源連結與查證日期。優先使用官方文件、原始碼、changelog。
- 找不到可靠答案時,researcher 寫下查了什麼、為什麼不可靠、還缺什麼。researcher 不關閉 decision ticket,並建議改成 HITL。

## implementer(15 分鐘)

implementer 實作 implementation ticket,並把 batch PR 合併到 integration branch。integration PR 由人合併。一輪最多處理 K 張 implementation ticket。K 的預設值是 3。補驗的 PR 也計入 K。

<div class="arch-embed" style="--h:683px;--h-wide:821px"><iframe scrolling="no" src="diagrams/04-implementer-round.html" loading="lazy" title="Implementer 一輪的順序"></iframe></div>
<p class="arch-open"><a href="diagrams/04-implementer-round.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

implementer 依 implementation ticket 的複雜度選模型:

| implementation ticket | 模型 |
| --- | --- |
| 機械改動 | haiku |
| 一般實作 | 預設模型 |
| 跨模組推理、重試 | 上限模型 |

上限模型也失敗時,implementation ticket 留給人。implementer 不無限重試。

警告:不要為了平行處理而多開 implementer loop。claim lock 只是一個 label,沒有身分資訊。兩條 implementer loop 會實作同一張 implementation ticket 兩次。implementer 依序 claim implementation ticket,再平行派工。

## prototyper(30 分鐘)

prototyper 一輪最多處理 1 張 prototype ticket。原因:原型的價值在人看完的反應。一次給兩個原型,人只看第一個。

- prototyper 只處理最後一則人的留言是開工指示的 prototype ticket。人沒有要求時,prototyper 不做原型。prototyper 不替人選要畫的選項。
- prototyper 在 code repo 的 worktree 做原型。prototyper 只 push 分支,不開 PR,不合併進 main。
- prototyper 交付兩個連結(原型頁、截圖圖庫),加上觀察與建議。prototyper 不關閉 prototype ticket。人回覆後,resolver 關閉 prototype ticket。
- 每個 code repo 可以有自己的原型指南。原型指南放在 planning repo,由 `wayfinder.yml` 指定。原型指南寫明原型的打包方式與截圖尺寸。

## reviewer(agent)

reviewer 不是 loop。implementer 對每個 batch PR 派出一個 reviewer(`wayfinder-loops:reviewer`)。reviewer 的 context 只有 diff。所以由 reviewer 檢查程式碼標準。

| 項目 | 內容 |
| --- | --- |
| 審查依據 | 該 repo 的 `CODING_STANDARDS.md`(選用)、implementation ticket 的規格、測試是否驗證實際行為、Fowler smell baseline |
| 確定的問題 | 直接修正。一個問題一個 commit,前綴 `review:` |
| 有疑義的問題 | 不修正。在 PR 留言,最多 5 條,附檔案與行號 |
| 規格嚴重不符 | 不修正。退回 implementation ticket,貼 `agent:blocked` |
| 結束 | 重跑本地 CI,在 PR 內文末尾加 Merge Danger,把 PR 設為 ready |

## 三個 GitHub Actions workflow

| Workflow | 觸發 | 工作 |
| --- | --- | --- |
| `promote-queued` | ticket 關閉、貼上 `agent:queued`、手動執行 | 把 blocker 全部關閉的 `agent:queued` ticket 升級成 `agent:implement`。退回 parent 不在 planning repo 的 sub-issue。 |
| `sync-project-status` | label、assignee、PR 變動 | 從 label 計算看板 Status。它是看板欄位唯一的 writer。 |
| `dispatch-on-decision` | 貼上 `decision:*` | 發出 `repository_dispatch`。用於即時觸發 resolver。 |

> 下一步:[02 · 一張 ticket 從想法到 main](02-journey.md)
