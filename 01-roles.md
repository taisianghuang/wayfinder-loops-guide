# 01 · 六個角色

> 本章給想知道 agent 在做什麼的人。本章說明五條 loop、一個 reviewer agent 與三個 GitHub Actions workflow 的工作。

---

## 角色一覽

| 角色 | 種類 | 頻率 | 工作 |
| --- | --- | --- | --- |
| cartographer | loop | 20 分鐘 | 把想法寫成 map 與決策票。決策完成後,寫 spec 與實作票。 |
| resolver | loop | 10 分鐘 | 讀人在決策票上的選擇。記錄結論並關閉決策票。 |
| researcher | loop | 15 分鐘 | 查證外部事實。把結果寫在決策票上並關閉。 |
| implementer | loop | 15 分鐘 | 收割、認領實作票、平行派工、派出 reviewer、維護整合 PR。 |
| prototyper | loop | 30 分鐘 | 人要求原型時,做原型頁與截圖。 |
| reviewer | agent | implementer 派出 | 審查每個批次 PR。修正問題,判斷單向門,把 PR 設為 ready。 |

## 每一輪的步驟

每條 loop 依頻率執行一輪。每一輪有五個步驟:掃描、認領、派工、驗收、回報。

<div class="arch-embed" style="--h:524px;--h-wide:614px"><iframe src="diagrams/04-loop-round.html" loading="lazy" title="每條 loop 每一輪的共同骨架"></iframe></div>
<p class="arch-open"><a href="diagrams/04-loop-round.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

| 項目 | 規則 |
| --- | --- |
| 讀設定 | 每一輪開始時,loop 從 planning repo 讀 `wayfinder.yml`。讀不到時,loop 停止。loop 不使用預設值。 |
| 派工 | 讀檔、查證、寫程式由 subagent 做。loop 只做掃描、認領、驗收、回報。loop session 的 context 不隨輪數增加。 |
| 驗收 | loop 不相信 subagent 的回報。loop 到 GitHub 確認 PR、label 與留言存在。 |
| subagent 失敗 | loop 移除認領,並留言記錄進度。 |
| 認領 | loop 貼 `agent:in-progress`。loop 不把 issue 指派給自己。assignee 欄位只用來表示「輪到人」。 |
| 留言前綴 | loop 的每則留言以 `🤖 [loop 名稱]` 開頭。沒有這個前綴的留言是人的留言。 |

五條 loop 用同一個 GitHub 帳號。所以 loop 用 label 與留言前綴分辨身分,不用留言作者。

## cartographer(20 分鐘)

cartographer 是唯一寫 map 內文的角色。其他角色只在 map 上留言。cartographer 把留言整理進 map 內文。

一輪依序做以下五件事:

1. **檢查 WIP 上限。** 等人決定的決策票有 5 張以上時,這一輪不開新的 HITL 決策票。開票速度不超過人的決定速度。
2. **整理 map 留言。** 把 resolver、researcher、implementer 的回報與人的方向留言寫進 Decisions so far,並更新霧。
3. **把霧變成決策票。** 能精確描述的問題,開成新的決策票。HITL 決策票附決策封包,並指派給人。AFK 決策票不指派,由 researcher 處理。
4. **處理新想法。** planning repo 中沒有管線 label 的 issue 是新想法。每個想法派一個 subagent,開成 map 與第一批決策票。
5. **把完成的 map 寫成 spec 與實作票。** 條件:決策票全部關閉,霧清空。subagent 寫 spec、在正確的 code repo 開實作票、建整合分支、開 integrate-and-verify 票。依賴設定完成後,最後才貼 `agent:queued`。

## resolver(10 分鐘)

resolver 的頻率要高於 cartographer。原因:決策票關閉後,才出現新的霧。人的每個決定要在幾分鐘內處理。

resolver 找出每張決策票的最後一則人的留言,並判斷種類:

| 種類 | 例子 | resolver 的動作 |
| --- | --- | --- |
| 終止型 | `decision:*` label;留言指出一個選項 | 記錄結論、關閉決策票(觸發 `promote-queued`)、通知 map |
| 非終止型 | `decision:discuss`、問題、猶豫 | 回覆一則留言,移除 `decision:discuss`。一則人的留言只回覆一則。 |

resolver 不替人決定。例外:人在留言中已指出一個選項時,resolver 把它轉成 label,並引用原句。

## researcher(15 分鐘)

researcher 只做 AFK 工作。一輪最多處理 2 張決策票。

- researcher 處理 `wayfinder:research` 決策票,與內容可以獨立完成的 `wayfinder:task` 決策票。有任何一步需要人(註冊帳號、付費、操作 UI)時,researcher 不處理。
- 每個重要事實附來源連結與查證日期。優先使用官方文件、原始碼、changelog。
- 找不到可靠答案時,researcher 寫下查了什麼、為什麼不可靠、還缺什麼。researcher 不關閉決策票,並建議改成 HITL。

## implementer(15 分鐘)

implementer 是唯一把程式寫進 main 的 loop。一輪最多處理 K 張實作票。K 的預設值是 3。補驗的 PR 也計入 K。

<div class="arch-embed" style="--h:683px;--h-wide:821px"><iframe src="diagrams/04-implementer-round.html" loading="lazy" title="Implementer 一輪的順序"></iframe></div>
<p class="arch-open"><a href="diagrams/04-implementer-round.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

implementer 依實作票的複雜度選模型:

| 實作票 | 模型 |
| --- | --- |
| 機械改動 | haiku |
| 一般實作 | 預設模型 |
| 跨模組推理、重試 | 上限模型 |

上限模型也失敗時,實作票留給人。implementer 不無限重試。

警告:不要為了平行處理而多開 implementer loop。label 鎖沒有身分資訊。兩條 implementer loop 會實作同一張實作票兩次。implementer 依序認領實作票,再平行派工。

## prototyper(30 分鐘)

prototyper 一輪最多處理 1 張原型票。原因:原型的價值在人看完的反應。一次給兩個原型,人只看第一個。

- prototyper 只處理最後一則人的留言是開工指示的原型票。人沒有要求時,prototyper 不做原型。prototyper 不替人選要畫的選項。
- prototyper 在 code repo 的 worktree 做原型。prototyper 只 push 分支,不開 PR,不合併進 main。
- prototyper 交付兩個連結(原型頁、截圖圖庫),加上觀察與建議。prototyper 不關閉原型票。人回覆後,resolver 關閉原型票。
- 每個 code repo 可以有自己的原型指南。原型指南放在 planning repo,由 `wayfinder.yml` 指定。原型指南寫明原型的打包方式與截圖尺寸。

## reviewer(agent)

reviewer 不是 loop。implementer 對每個批次 PR 派出一個 reviewer(`wayfinder-loops:reviewer`)。reviewer 的 context 只有 diff。所以由 reviewer 檢查程式碼標準。

| 項目 | 內容 |
| --- | --- |
| 審查依據 | 該 repo 的 `CODING_STANDARDS.md`(選用)、實作票的規格、三種會說謊的測試、Fowler smell baseline |
| 確定的問題 | 直接修正。一個問題一個 commit,前綴 `review:` |
| 有疑義的問題 | 不修正。在 PR 留言,最多 5 條,附檔案與行號 |
| 規格嚴重不符 | 不修正。退回實作票,貼 `agent:blocked` |
| 結束 | 重跑本地 CI,在 PR 內文末尾加 Merge Danger,把 PR 設為 ready |

## 三個 GitHub Actions workflow

| Workflow | 觸發 | 工作 |
| --- | --- | --- |
| `promote-queued` | 票關閉、貼上 `agent:queued`、手動執行 | 把 blocker 全部關閉的 `agent:queued` 票升級成 `agent:implement`。退回 parent 不在 planning repo 的子票。 |
| `sync-project-status` | label、assignee、PR 變動 | 從 label 計算看板 Status。它是看板欄位唯一的 writer。 |
| `dispatch-on-decision` | 貼上 `decision:*` | 發出 `repository_dispatch`。用於即時觸發 resolver。 |

> 下一步:[02 · 一張票從想法到 main](02-journey.md)
