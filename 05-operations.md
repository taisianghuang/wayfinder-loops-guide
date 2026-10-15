# 05 · 啟動與疑難排解

> 本章給維運的人。本章說明怎麼啟動五條 loop、怎麼處理常見問題,與怎麼更新和移除 plugin。開始前,先完成 [04 章](04-setup.md)的 9 個步驟。

---

## 啟動五條 loop

啟動有兩種模式。建議用模式 A。

| 模式 | 方式 | 適用 |
| --- | --- | --- |
| A | 互動式 `/loop`。每一輪累積在同一段對話中 | 全部五條 loop |
| B | 每一輪開一個新的 headless `claude -p` session,執行完就結束 | resolver、cartographer、researcher |

!> **警告**:每條 loop 只能有一個執行中的 session。兩個相同的 loop 會實作同一張實作票兩次。平行處理由 loop 內部派出的 subagent 做。

### 模式 A:互動式 `/loop`

注意:resolver 的頻率必須高於 cartographer。原因:決策票關閉後,才出現新的霧。

1. 開五個終端機。
2. 在每個終端機,進入 `workspace_dir`,啟動 Claude Code。
3. 在每個終端機,執行一行指令:

   | 終端機 | 指令 | 用途 |
   | --- | --- | --- |
   | 1 | `/loop 10m /wayfinder-loops:resolver` | 頻率最高。人的決定要立刻處理 |
   | 2 | `/loop 20m /wayfinder-loops:cartographer` | 開票、整理 map |
   | 3 | `/loop 15m /wayfinder-loops:researcher` | AFK 查證 |
   | 4 | `/loop 15m /wayfinder-loops:implementer` | 實作、審查、收割 |
   | 5 | `/loop 30m /wayfinder-loops:prototyper` | 人要求原型後才工作。一輪一張 |

注意:`/loop` 排程在 7 天後到期。session 必須一直開著。到期後,重新執行步驟 3。

### 模式 B:每一輪一個新的 headless session

loop 不保存狀態。狀態全部在 GitHub。所以每一輪可以開一個新的 `claude -p` session。

模式 B 有以下限制:

- 不能在 session 中輸入訊息。要溝通時,在 issue 上留言。
- headless session 不能互動授權。腳本內建每條 loop 的工具白名單。
- implementer 與 prototyper 要寫程式、跑測試、發布頁面。白名單無法列出所有權限。implementer 與 prototyper 用模式 A。

1. 對每條 loop,執行一行指令。參數為 loop 名稱與間隔秒數:

   ```bash
   <plugin 安裝路徑>/bin/wayfinder-run-loop resolver 600
   <plugin 安裝路徑>/bin/wayfinder-run-loop cartographer 1200
   <plugin 安裝路徑>/bin/wayfinder-run-loop researcher 900
   ```

## 疑難排解

先執行 `/wayfinder-loops:doctor`。

| 症狀 | 原因 | 處理 |
| --- | --- | --- |
| 看板卡片不移動 | Status 選項名稱不完全一致;或 repo 沒有設定 `PROJECT_URL` / `PROJECT_TOKEN` | 比對 [03 章](03-board.md#看板的-13-個欄位)的 13 個值。檢查 variables 與 secrets |
| loop 看不到看板上的卡片 | 在看板上直接新增的卡片是 draft | 在卡片上按 **Convert to issue**,選 planning repo |
| 實作票停在「📋 排隊中」 | blocker 關閉時沒有觸發升級(依賴是之後才加的) | 在該 repo 的 Actions 頁,手動執行 `promote-queued`。它重新檢查所有 `agent:queued` 實作票 |
| `promote-queued` 失敗 | 沒有設定 `PLANNING_REPO` variable | 設定 `PLANNING_REPO`。沒有設定時,workflow 刻意失敗。否則所有有 parent 的實作票會被退回,而且沒有錯誤訊息 |
| 實作票有 `agent:blocked` | 檢查不通過、本地 CI 紅、reviewer 判定規格不符、合併衝突 | 讀實作票上 `🤖 [implementer]` 或 `🤖 [reviewer]` 的留言。處理後移除 `agent:blocked` |
| 整合 PR 的綠燈不可信 | main 有新的 commit。GitHub 不重跑 PR 上的 check | implementer 每一輪自動合併 main 並重跑閘門。確認它的回報寫著「落後 0、閘門綠」 |
| 同一張實作票被實作兩次 | 同一條 loop 有兩個 session | 關閉一個 session |
| `gh` 指令失敗 | `gh` 版本低於 2.94,或 token 沒有 `project` scope | 升級 `gh`。執行 `gh auth refresh -s project` |

## 更新

1. 執行:

   ```text
   /plugin marketplace update wayfinder-loops
   /plugin update wayfinder-loops@wayfinder-loops
   /reload-plugins
   ```

## 移除

注意:移除 plugin 不會移除 GitHub 上的 label、看板與 workflow。

1. 執行:

   ```text
   /plugin uninstall wayfinder-loops@wayfinder-loops
   ```

> 回到 [00 · 總覽](00-overview.md)
