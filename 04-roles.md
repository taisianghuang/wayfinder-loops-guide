# 04 · 六個角色

> 五條 loop 各自依頻率跑一輪,加上一個由 Implementer 派出的 reviewer agent。每條 loop 的一輪都是同一個形狀:**掃描 → 認領 → 派工 → 驗收 → 回報**。

---

## 共同的一輪

<div class="arch-embed" style="--h:524px;--h-wide:614px"><iframe src="diagrams/04-loop-round.html" loading="lazy" title="每條 loop 每一輪的共同骨架"></iframe></div>
<p class="arch-open"><a href="diagrams/04-loop-round.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

- **第 0 步**:每一輪先從 planning repo 讀 `wayfinder.yml`。讀不到就停,不猜預設值。
- **派工**:讀檔、查證、寫 code 這些會留下大量殘渣的工作,一律交給 subagent。loop 本身只做掃描、認領、驗收、回報,所以 loop session 的 context 不會隨輪數膨脹。
- **驗收不信自述**:subagent 說「做完了」不算,loop 會回 GitHub 核對 PR、label、留言是否真的存在。subagent 死掉,由 loop 善後:拔認領、留言註明進度。

## Cartographer · 製圖者(20 分鐘)

**map 的唯一 writer。** 其他角色只能在 map 上留言,由它收斂寫進 body。

一輪依序做:

1. **檢查 WIP 上限**:等人決定的票 ≥ 5 張,這一輪不開新的 HITL 票。開票速度不能超過人的決策速度。
2. **收斂 map 留言**:把 Resolver、Researcher、Implementer 的回報與人類的方向性留言,寫進 Decisions so far,更新霧。
3. **讓霧畢業**:能精確講出來的問題開成新決策票;HITL 類附決策封包並 assign 給人,AFK 類不 assign、等 Researcher。
4. **接新想法**:planning 裡沒有任何管線 label 的 issue,一個想法派一個 subagent 開成 map 與第一批決策票。
5. **熟了的 map → spec → 實作票**:決策票全關、霧清空,派 subagent 寫 spec、切實作票到對的 code repo、建整合分支、切 integrate-and-verify 票。`agent:queued` 一定是依賴串好之後**最後**貼的 label。

## Resolver · 決策記錄者(10 分鐘)

**要跑得比 Cartographer 勤**:新的霧是票關掉之後才冒出來的,人的每個決定都應該在幾分鐘內被消化。

- 找出每張決策票「最後一則人類留言」,判斷是**終止型**(label 或明確指名一個選項)還是**非終止型**(discuss、疑問、猶豫)。
- 終止型 → 記錄結論、關票(觸發 promote-queued)、通知 map。
- 非終止型 → 回一則聚焦的回覆就停,拔掉 `decision:discuss`。**一則人類訊息只回一則。**
- 絕不替人決定。唯一的例外是把人已經在留言說出的選擇轉錄成 label,並引用原句。

## Researcher · 查證者(15 分鐘)

純 AFK。一輪最多處理 2 張,查得深比查得多重要。

- 撿 `wayfinder:research`,以及內容明確可獨立完成的 `wayfinder:task`。只要有一步需要人(註冊帳號、付費、點 UI),就不是它的。
- 每個關鍵事實附來源連結與查證日期;官方文件、原始碼、changelog 優先。
- 查不到可靠答案:貼回「查了什麼、為什麼不可靠、還缺什麼」,不關票,建議轉 HITL。

## Implementer · 實作者(15 分鐘)

**唯一把程式寫進 main 的 loop。** 一輪最多 K 張(預設 3,補驗的 PR 也算名額)。

<div class="arch-embed" style="--h:683px;--h-wide:821px"><iframe src="diagrams/04-implementer-round.html" loading="lazy" title="Implementer 一輪的順序"></iframe></div>
<p class="arch-open"><a href="diagrams/04-implementer-round.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

- **依票的複雜度選模型**:機械改動用 haiku,一般實作用預設模型,跨模組推理或重試用上限模型。上限也失敗的票留給人,不無限重試。
- **絕對不要多開 implementer loop 來取得平行度**:label 鎖沒有身分資訊,兩條 loop 會實作同一張票兩次。平行度靠「同一條 loop 依序認領、平行派工」取得。

## Prototyper · 原型師(30 分鐘)

一輪最多 1 張:原型的價值全在人看完的反應,一次給兩張,人只會看第一張。

- 只撿「最後一則人類留言是開工訊號」的原型票。**人沒開口就不畫**,不替人選要畫哪個選項。
- 原型碼在 code repo 的 worktree 做,只 push 分支,**永不開 PR、永不進 main**。
- 交付兩個連結(可點的原型頁、截圖圖庫)加上觀察與建議;不關票,等人回應後由 Resolver 結案。
- 每個 code repo 可以有自己的**原型指南**(放在 planning repo,由設定檔指過去),寫明這個 repo 的原型怎麼打包、截什麼尺寸。

## reviewer · 獨立審查者(agent)

不是 loop,而是 Implementer 對每張批次 PR 派出的 agent(`wayfinder-loops:reviewer`)。它的 context 只有 diff,所以最適合守標準。

| 項目 | 內容 |
| --- | --- |
| 審查依據 | 該 repo 的 `CODING_STANDARDS.md`(選用)、票的規格、三種會說謊的測試、Fowler smell baseline |
| 確定的問題 | 直接修,一個問題一個 commit,前綴 `review:` |
| 有疑義的 | 不修,在 PR 留言,最多 5 條,附檔案與行號 |
| 規格嚴重不符 | 不修,退回票、貼 `agent:blocked` |
| 收尾 | 重跑本地 CI、在 PR body 末尾附 Merge Danger、轉 ready |

細節見 [06 · 品質的三層把關](06-quality-gates.md)。

## 三個 GitHub Actions workflow

| Workflow | 觸發 | 做什麼 |
| --- | --- | --- |
| `promote-queued` | 票關閉、貼上 `agent:queued`、手動 | blocker 全關的 `agent:queued` 票升級成 `agent:implement`;parent 不在 planning 的子票退回 |
| `sync-project-status` | label、assignee、PR 變動 | 從 label 算出看板 Status,**看板欄位唯一的 writer** |
| `dispatch-on-decision` | 貼上 `decision:*` | 發 `repository_dispatch`,給想即時觸發 Resolver 的人掛勾 |

> 下一步:[05 · 一張票從想法到 main](05-journey.md)
