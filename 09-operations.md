# 09 · 運轉、成本與疑難排解

---

## 啟動五條 loop

### 模式 A:互動式 `/loop`(建議)

每條 loop 開一個終端機,在 `workspace_dir` 啟動 Claude Code,各執行一行:

| 終端機 | 指令 | 為什麼是這個頻率 |
| --- | --- | --- |
| 1 | `/loop 10m /wayfinder-loops:resolver` | 最勤:人的決定要立刻被消化 |
| 2 | `/loop 20m /wayfinder-loops:cartographer` | 開票、收斂地圖 |
| 3 | `/loop 15m /wayfinder-loops:researcher` | AFK 查證 |
| 4 | `/loop 15m /wayfinder-loops:implementer` | 實作、審查、收割 |
| 5 | `/loop 30m /wayfinder-loops:prototyper` | 只在人說「畫」之後動,一輪一張 |

Resolver 必須跑得比 Cartographer 勤:新的霧是決策票關掉之後才冒出來的。

### 模式 B:每輪全新的 headless session

`/loop` 的每一輪會累積在同一段對話裡。loop 是無狀態設計(狀態全在 GitHub),所以也可以每一輪都開全新的 `claude -p`,跑完即退:

```bash
<plugin 安裝路徑>/bin/wayfinder-run-loop resolver 600
<plugin 安裝路徑>/bin/wayfinder-run-loop cartographer 1200
<plugin 安裝路徑>/bin/wayfinder-run-loop researcher 900
```

代價:不能在 session 裡插話(要溝通就去 issue 上留言,本來就該這樣);headless 無法互動授權,腳本內建各 loop 的工具白名單。Implementer 與 Prototyper 要寫程式、跑測試、發頁面,權限列不完,建議用模式 A。

!> **同一條 loop 只能開一個。** 平行度由 loop 內部派 subagent 取得;開兩個同名 loop 會搶同一張票,實作兩次。

## 成本與資源

| 項目 | 說明 |
| --- | --- |
| token | 五條 loop 長時間開著會持續消耗。沒事做的一輪很便宜(只查 label);成本集中在 subagent 實作、審查、查證的時候 |
| 機器負載 | Implementer 一輪最多 `limits.per_round` 個 subagent 同時跑完整 CI,另外每張 PR 還有 reviewer 的一次 CI 與收割的一次 CI |
| 平行測試 | 同一台機器上的平行測試會搶資料庫與 port;code repo 的 CI 指令最好能隔離(不同 compose 專案名、不同 port) |
| `/loop` 期限 | 排程任務 7 天後到期,session 必須一直開著 |
| 常駐 context | plugin 裝上後每個 session 約多 800 tokens(skill 的描述);loop 被呼叫時才載入全文 |

## 安全

- loop 以**你的 GitHub 身分**開票、留言、開 PR、把批次 PR 合進整合分支。**合併進 main 只由人做。**
- loop 不改 `.github/workflows/`、不代貼人工核准類的 label、不動 `workspace_dir` 以外的目錄。
- PAT 只需要 `repo` + `project`,放在 repo secret,不寫進設定檔。
- 建議在 code repo 對「裁判檔」(CI 設定、lint 設定、測試 baseline、`CODING_STANDARDS.md`)設一個需要人工 label 才能通過的 workflow,防止 agent 為了過關而改規則。

## 疑難排解

先跑 `/wayfinder-loops:doctor`。常見狀況:

| 症狀 | 原因 | 處理 |
| --- | --- | --- |
| 看板卡片不動 | Status 選項名稱不完全一致;或 repo 沒設 `PROJECT_URL` / `PROJECT_TOKEN` | 對照 07 篇的 13 個值;檢查 variables 與 secrets |
| 看板上的卡片 loop 看不到 | 看板上直接新增的是 draft | 按 **Convert to issue** 轉到 planning |
| 票一直停在「📋 排隊中」 | blocker 關掉時沒觸發升級(依賴是事後才補的) | 到該 repo 的 Actions 手動執行 `promote-queued`,它會重掃所有 `agent:queued` |
| `promote-queued` 直接失敗 | 沒設 `PLANNING_REPO` variable | 補上;刻意設計成大聲失敗,不然所有有 parent 的票都會被默默退回 |
| 實作票被貼 `agent:blocked` | 防呆不過、本地 CI 紅、reviewer 判定規格不符、合併衝突 | 看票上 `🤖 [implementer]` 或 `🤖 [reviewer]` 的留言;處理完拔掉 `agent:blocked` |
| spec→main PR 的綠燈不可信 | main 往前走了,GitHub 不會重跑 PR 上的 check | Implementer 每輪會自動合 main 並重跑閘門;看它的回報「落後 0、閘門綠」 |
| 同一張票被做了兩次 | 開了兩條同名 loop | 只留一條 |
| `gh` 指令失敗 | 版本低於 2.94,或 token 缺 `project` scope | 升級 `gh`;`gh auth refresh -s project` |

## 更新與移除

```text
/plugin marketplace update wayfinder-loops
/plugin update wayfinder-loops@wayfinder-loops
/reload-plugins
```

移除:`/plugin uninstall wayfinder-loops@wayfinder-loops`。GitHub 上的 label、看板、workflow 不會被移除。

> 下一步:[10 · 設計取捨與踩過的坑](10-lessons.md)
