# 03 · 用看板管理

> 本章給 PM、SA 與工程師。本章說明看板的 13 個欄位、人的三個動作、怎麼加入想法、怎麼處理原型票,以及每個 label 的意思。開票、查證、實作、開 PR、更新看板欄位都由 agent 與 workflow 做。

---

PM 與 SA 只需要兩個權限:planning repo 的 write 權限,與 Project 看板的權限。PM 與 SA 不需要 code repo。

## 看板的 13 個欄位

所有狀態寫在 label 上。看板欄位由 label 計算。`sync-project-status` 依下表**由上往下**比對。第一個成立的條件決定卡片的欄位。

| 順序 | 條件 | 欄位 |
| --- | --- | --- |
| 1 | 已關閉 | ✔️ 已關閉 |
| 2 | `agent:blocked` | 🚫 卡住 |
| 3 | 有開著的 PR 會關掉它 | 👀 待你 review |
| 4 | `agent:in-progress` | ⚡ 實作中 |
| 5 | `agent:implement` | 🚀 可實作 |
| 6 | `agent:queued` | 📋 排隊中 |
| 7 | `decision:discuss` | 💬 討論中 |
| 8 | 其他 `decision:*` | ✅ 已決定,待收斂 |
| 9 | `wayfinder:research` | 🤖 查證中 |
| 10 | `wayfinder:grilling` / `prototype` | 有 assignee → ⏳ 等你決定;沒有 → 🔍 準備選項中 |
| 11 | `wayfinder:task` | 有 assignee → ⏳ 等你決定;沒有 → 🤖 查證中 |
| 12 | `wayfinder:map` | 🗺️ 規劃中 |
| 13 | planning repo 上沒有管線 label | 💡 收件匣 |

警告:欄位名稱(含 emoji 與空格)必須**逐字一致**。workflow 用字串比對。

注意:在看板上拖動卡片沒有作用。下一次 label 變動時,卡片回到計算出的欄位。要改變狀態,在 issue 上改 label、assignee,或關閉 issue。

注意:「🚫 卡住」是錯誤旗標。開啟卡片,讀 `🤖 [...]` 留言中寫的問題。

## 你的三個動作

agent 開始執行後,人只做三個動作。

### 動作一:查看「⏳ 等你決定」欄

每條 loop 在每一輪結束時,回報還在等人的決策票。

1. 開啟 org 的 Project 看板。
2. 查看最左邊的「⏳ 等你決定」欄。
3. 欄中沒有卡片時,關閉看板。欄中有卡片時,做動作二。

### 動作二:讀決策封包,選一個選項

每張 HITL 決策票附一份決策封包:

| 部分 | 內容 |
| --- | --- |
| 選項表 | 每個選項的做法、代價、之後變簡單的事、之後變難的事 |
| 建議 | 一個明確的建議。引用 repo 中的 ADR、詞彙表或程式碼 |
| 決策樹 | 選了某個選項後,下一個問題是什麼 |
| 回覆方法 | 要貼的 label,或用留言回覆 |

步驟:

1. 點卡片。看板右側面板顯示整張 issue。
2. 讀決策封包。
3. 在 Labels 區加一個 label:

   | Label | 意思 | 之後的結果 |
   | --- | --- | --- |
   | `decision:a` / `b` / `c` | 選 A / B / C | resolver 在幾分鐘內記錄結論、關閉決策票、通知 map |
   | `decision:approve` | 選建議的選項 | 同上 |
   | `decision:discuss` | 有疑問,要討論 | 卡片移到「💬 討論中」。resolver 回覆後移除 label,卡片回到「⏳ 等你決定」 |

4. 關閉面板。

要用 `decision:discuss` 時,先在面板中留言寫下疑問,再貼 label。agent 在同一張卡片上回覆。

#### 用留言回覆

注意:沒有以 `🤖 [` 開頭的留言是人的留言。

1. 在決策票上留言。
2. 在留言中指出一個選項。例:「選 B」「照建議」。

resolver 把留言轉成 label,引用原句,再關閉決策票。留言沒有明確指出一個選項時,resolver 當作要討論。resolver 不替人決定。

決策封包有 A–E 選項時,用留言回覆選項字母。`decision:*` label 只有 a、b、c。

#### 決策票的問題錯誤時

1. 把決策票關閉為 **Not planned**。

cartographer 把這個問題退回 map 的霧,重新規劃。

<div class="arch-embed" style="--h:512px;--h-wide:599px"><iframe scrolling="no" src="diagrams/03-decision-ticket.html" loading="lazy" title="決策票在你手上的狀態"></iframe></div>
<p class="arch-open"><a href="diagrams/03-decision-ticket.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

### 動作三:審查整合 PR

批次 PR 不合併進 main。批次 PR 合併進該 spec 的整合分支。implementer 驗證批次 PR 後,合併它、關閉實作票,讓下一批實作票開始。批次 PR 不出現在「👀 待你 review」。看板顯示「⚡ 實作中」。

一個 spec 的實作票全部收割後,implementer 開一個整合 PR。只有整合 PR 進「👀 待你 review」。整合 PR 的內文依序有:

| 部分 | 內容 |
| --- | --- |
| Merge Danger 彙整 | 每個批次 PR 的單向門 / 雙向門判斷。單向門逐條列成「**審這裡**」清單:PR、檔案、最壞情況 |
| Summary | 一張 Mermaid 圖。說明這個 spec 改變了什麼行為 |
| 驗證 | 合併進來的 main SHA、閘門指令與結果 |
| 未解疑點 | reviewer 留給人判斷的問題 |

警告:只有人可以合併整合 PR。不要讓 agent 合併整合 PR。

1. 開啟「👀 待你 review」欄中的整合 PR。
2. 讀內文最上方的「審這裡」清單。
3. 細看清單中的每個單向門。
4. 快速看過其他變更。
5. 確認 implementer 的回報寫著「落後 main 0 個 commit、閘門綠」。
6. 合併整合 PR。

## 丟想法進系統

想法屬於前端或後端,不需要先決定。管線會決定實作票放在哪個 code repo。

### 從 planning repo 加入

1. 在 planning repo 開一個 issue。可以用手機 GitHub app。
2. 寫下想法。
3. 不要加 label。

### 從看板加入

警告:沒有轉換的 draft 只存在看板上。loop 看不到 draft。

1. 在看板上按 **+ Add item**。
2. 輸入想法。
3. 在這張卡片上按 **Convert to issue**。
4. 選 planning repo。

想法出現在「💡 收件匣」欄。cartographer 下一輪把它開成 map(移到「🗺️ 規劃中」)與第一批決策票。

## 原型票

文字無法說清楚的畫面問題,cartographer 開 `wayfinder:prototype` 決策票(原型票)。原型票附選項,並指派給人。人沒有要求時,prototyper 不做原型。

1. 在原型票上留言,指出要畫的選項。例:「先畫 V1」「A 和 B 都畫出來」。
2. 等 prototyper 下一輪交付:
   - 一個手機可以開啟的原型頁。頁面底部有切換器,可以切換情境。
   - 一個截圖圖庫頁。
   - 做原型時發現的事,與對原型票上每個問題的初步建議。
3. 開啟原型頁與截圖圖庫頁。
4. 在原型票上回覆哪些正確、哪些不正確。

resolver 記錄結論並關閉原型票。

## label 清單

每個 label 只有一個 writer。

### mattpocock-skills 的 label(10 個)

| Label | writer |
| --- | --- |
| `needs-triage` / `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix` | triage、to-spec、to-tickets |
| `wayfinder:map` | wayfinder |
| `wayfinder:research` / `wayfinder:prototype` / `wayfinder:grilling` / `wayfinder:task` | wayfinder |

### 決策票的種類

| Label | 誰處理 | 意思 |
| --- | --- | --- |
| `wayfinder:grilling` | 人(HITL) | 要人決定的設計問題 |
| `wayfinder:prototype` | 人要求後,由 prototyper 處理 | 文字無法說清楚,先做原型 |
| `wayfinder:research` | researcher(AFK) | 查證外部事實:API 限制、價格、文件 |
| `wayfinder:task` | 依內容 | 可以 AFK 完成的工作由 researcher 處理;需要人做的由人處理 |

### wayfinder-loops 的 label

| Label | writer | 意思 |
| --- | --- | --- |
| `wayfinder:ticket` | cartographer | 所有決策票都有這個 label。用來一次找出所有決策票 |
| `agent:queued` | to-tickets(由 cartographer 執行) | 已授權給 agent,等 blocker 關閉 |
| `agent:implement` | `promote-queued` workflow | blocker 全部關閉,implementer 可以處理 |
| `agent:in-progress` | loop | 已被一條 loop 認領。實作票保留到收割時關閉 |
| `agent:blocked` | workflow / loop | **錯誤旗標**:升級被拒絕,或實作失敗。要人處理 |
| `decision:a` / `b` / `c` / `approve` | **只有人** | 終止型決定。關閉決策票後保留,作為紀錄 |
| `decision:discuss` | **只有人** | 要求討論。resolver 回覆後移除 |

### 暫停一張實作票

警告:不要用 `agent:blocked` 暫停實作票。`agent:blocked` 是錯誤旗標,不是暫停。

1. 取下 `agent:queued`。
2. 留言說明暫停的原因。

## 實作票的 label 生命週期

<div class="arch-embed" style="--h:505px;--h-wide:591px"><iframe scrolling="no" src="diagrams/07-impl-ticket-labels.html" loading="lazy" title="實作票的 label 生命週期"></iframe></div>
<p class="arch-open"><a href="diagrams/07-impl-ticket-labels.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

批次 PR 開啟後,`agent:in-progress` 保留到收割。原因:移除後,實作票同時符合三個處理條件(有 `agent:implement`、沒有被認領、沒有 assignee)。下一輪會再實作一次。

> 下一步:[04 · 建立工作流](04-setup.md)
