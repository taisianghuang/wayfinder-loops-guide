# 07 · Label、看板與不變式

> 所有狀態都寫在 label 上。看板只是 label 的投影。

---

## Label 權威清單

### mattpocock-skills 自己會讀寫的(10 個)

| Label | writer |
| --- | --- |
| `needs-triage` / `needs-info` / `ready-for-agent` / `ready-for-human` / `wontfix` | triage、to-spec、to-tickets |
| `wayfinder:map` | wayfinder |
| `wayfinder:research` / `wayfinder:prototype` / `wayfinder:grilling` / `wayfinder:task` | wayfinder |

### 本 plugin 新增的

| Label | writer | 意思 |
| --- | --- | --- |
| `wayfinder:ticket` | Cartographer | 所有決策票的聚合 label,一次撈出 |
| `agent:queued` | to-tickets(經 Cartographer) | 已授權給 agent,等 blocker 關閉 |
| `agent:implement` | `promote-queued` workflow | blocker 全關,Implementer 可以撿 |
| `agent:in-progress` | loop | 已被某條 loop 認領;實作票保留到收割關票 |
| `agent:blocked` | workflow / loop | **錯誤旗標**:升級被拒或實作卡住,要人看 |
| `decision:a` / `b` / `c` / `approve` | **只有人** | 終止型決定;關票後留著當紀錄 |
| `decision:discuss` | **只有人** | 門鈴:有疑慮想聊;Resolver 回覆後拔掉 |

**每個 label 只有一個 writer。** 要暫停一張實作票,取下 `agent:queued` 並留言說明;不要貼 `agent:blocked`(那是錯誤,不是暫停)。

## 實作票的 label 生命週期

<div class="arch-embed" style="--h:505px;--h-wide:591px"><iframe src="diagrams/07-impl-ticket-labels.html" loading="lazy" title="實作票的 label 生命週期"></iframe></div>
<p class="arch-open"><a href="diagrams/07-impl-ticket-labels.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

PR 開完之後 `agent:in-progress` **刻意保留**:如果拔掉,票會同時符合「有 implement、沒被認領、沒 assignee」三個撿票條件,下一輪就被重做一次。

## 看板的 13 個欄位

`sync-project-status` 依下表**由上往下**比對,第一個成立的就是那張卡片的欄位:

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

欄位名稱(含 emoji 與空格)必須**逐字一致**,workflow 用字串比對。看板上拖動卡片沒有用,下一次 label 變動就會被算回去。

## 不變式

健檢時逐條核對:

- 任何 loop、任何 workflow 都**不讀**看板 Status 做判斷;判斷一律讀 label。
- 每個 label 只有一個 writer。
- map body 只有 Cartographer 寫。
- 決策票沒有人類新動作 → 所有 loop 對它不動作;**永不逾時自動核准**。
- 子票(sub-issue)永不自動升級成實作票;只有 parent 在 planning 的 spec 子票例外。
- 實作票的 PR base 永遠是 `spec/*` 整合分支;**人類只合併 spec→main 的 PR**。任何 base 是 main 的 `implement/*` PR 都是契約違規。
- 批次 PR 的驗證在本地(`ci` + 獨立 reviewer),不靠 GitHub Actions。
- loop 不改 `.github/workflows/`,不代貼人工核准類的 label。

> 下一步:[08 · 安裝與設定](08-install.md)
