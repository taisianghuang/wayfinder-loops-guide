# 04 · 建立工作流

> 本章給要導入 wayfinder-loops 的人。本章說明從零建立工作流的 4 個步驟。步驟 2 由 `/wayfinder-loops:setup` 自動完成大部分設定;不使用 setup 時,改做本章最後的「手動設定」。完成後,看板上出現第一張「⏳ 等你決定」的 decision ticket。

---

## 前置需求

### 適用條件

| 條件 | 說明 |
| --- | --- |
| 團隊 | 小團隊。PM / SA 不寫程式,但要自己提出需求、做決定。 |
| issue tracker | 已經用 GitHub Issues 管理需求,可以使用 GitHub Project 看板。 |
| 機器 | 一台可以長時間執行 Claude Code 的機器。 |
| CI | 每個 code repo 有一個能在本機執行完成的 CI 指令。 |

以下情況不適用:

- code repo 沒有本機 CI 指令。batch PR 的驗證依賴這個指令。
- 使用 GitHub 以外的 issue tracker。
- 不能讓 agent 用你的 GitHub 帳號開 ticket 與開 PR。
- 一台機器同時服務多個獨立專案,而且每個專案有自己的 planning repo 或 org。plugin 的設定只有一組,五條 loop 只服務一個 planning repo。多個專案可以共用同一個 planning repo:在 `wayfinder.yml` 的 `code_repos` 列出所有專案的 code repo。

### 工具與帳號

| 項目 | 說明 |
| --- | --- |
| Claude Code | 2.1.275 或更新版本 |
| mattpocock-skills | 安裝本 plugin 時一起安裝 |
| `gh` CLI | 2.94 或更新版本(需要 `--blocked-by`、sub-issue 等功能) |
| `gh` token scope | 含 `repo`、`project`、`workflow`:`gh auth refresh -h github.com -s repo,project,workflow` |
| GitHub organization | 一個 planning repo、一個或多個 code repo、一個 **org** Project 看板(Free 方案可用;個人帳號的 Project 不支援) |
| classic PAT | scope 為 `repo` 與 `project`。workflow 用它寫看板與跨 repo 升級 implementation ticket |

## 1. 安裝 plugin

1. 在 Claude Code 中,加入 marketplace:

   ```text
   /plugin marketplace add taisianghuang/wayfinder-loops
   ```

2. 安裝 plugin:

   ```text
   /plugin install wayfinder-loops@wayfinder-loops
   ```

3. 輸入兩個設定值:

   | 設定 | 意思 | 例 |
   | --- | --- | --- |
   | `planning_repo` | 放想法、map、decision ticket 與 `wayfinder.yml` 的 repo | `my-org/planning` |
   | `workspace_dir` | 放 code repo 本機 clone 的資料夾。loop 只在這個資料夾中工作 | `~/wayfinder-workspace` |

注意:要修改這兩個值,執行 `/plugin`,選 wayfinder-loops,選 Configure。

## 2. 執行 setup

`/wayfinder-loops:setup` 是互動安裝精靈。它先盤點現況,只讀不寫;再列出所有寫入動作,等你同意後才執行。每一步已經完成時就跳過,所以可以重複執行。

| setup 做的事 | 說明 |
| --- | --- |
| 前置檢查 | `gh` 版本與 scope、mattpocock-skills、planning repo 的 owner 是 organization |
| planning repo | 不存在時建立 |
| `wayfinder.yml` | 問 code repo、本機資料夾名稱與 CI 指令,產生設定,用 PR 送進 planning repo |
| clone | code repo 不在 `workspace_dir` 時,clone 進去 |
| label | 在 planning repo 與每個 code repo 建立 label |
| 看板 | 建立 org Project 看板,把 Status 改成 [13 個欄位](03-board.md#看板的-13-個欄位),把檢視改成以 Status 分欄的 Board,連結每個 repo |
| variable | 在每個 repo 設定 `PROJECT_URL` 與 `PLANNING_REPO` |
| workflow | 用 PR 把三個 workflow 送進每個 repo |

setup 不做以下兩件事。它列出步驟,由你完成:

- 建立 classic PAT,設定 `PROJECT_TOKEN` 與 `ORG_AUTOMATION_TOKEN` 兩個 secret。
- 合併 setup 開的 PR。

1. 執行:

   ```text
   /wayfinder-loops:setup
   ```

2. 回答 setup 的問題。
3. 讀 setup 列出的計畫。同意後,setup 開始執行。

   警告:不要把 token 貼進 Claude Code 的對話。不要在對話中用 `!` 執行設定 secret 的指令。`!` 不能互動輸入,secret 會被設成空值。

4. 依 setup 的說明建立 PAT。在 Claude Code 以外的終端機,執行 setup 給的指令設定 secret。
5. 回到 Claude Code,回覆 setup。setup 確認 secret 存在。
6. 合併 setup 開的每個 PR。workflow 與 `wayfinder.yml` 合併後才生效。

## 3. 健檢

doctor 只讀取,不寫入。它逐項回報 ✅ 或 ❌:設定是否生效、`gh` 版本與 scope、`wayfinder.yml` 能否讀取、每個 clone 的 origin 是否正確、mattpocock-skills 是否安裝、每個 repo 的 variables 與 secrets。

1. 執行:

   ```text
   /wayfinder-loops:doctor
   ```

2. 確認所有項目是 ✅。有 ❌ 時,依 doctor 的說明修正,再執行步驟 1。

## 4. 冒煙測試

1. 在 planning repo 開一個 issue。寫一個小而模糊的想法。
2. 手動執行一次 `/wayfinder-loops:cartographer`。
3. 確認看板上出現 map(🗺️ 規劃中)與 decision ticket(⏳ 等你決定)。decision ticket 指派給你。
4. 在一張 decision ticket 上貼 `decision:approve`。
5. 手動執行一次 `/wayfinder-loops:resolver`。
6. 確認 decision ticket 已關閉,map 上有一則 `🤖 [resolver]` 留言。

所有結果正確時,依 [05 章](05-operations.md)啟動五條 loop。

## 手動設定

不使用 setup 時,在步驟 1 之後依序完成 M1 到 M6,再執行步驟 3。

### M1. clone code repo

注意:資料夾名稱必須與 M2 中 `wayfinder.yml` 的 `dir` 相同。

1. 把每個 code repo clone 到 `workspace_dir` 中。

### M2. 寫 `wayfinder.yml`

`wayfinder.yml` 是整個團隊共用的設定。它要納入版本控制。

注意:`ci` 指令必須能在本機一次執行完成。`ci` 指令必須與 GitHub 上的 CI 檢查相同。batch PR 的驗證依賴這個指令。

1. 在 planning repo 的根目錄建立 `wayfinder.yml`。
2. 依你的 repo 填寫內容。範例:

   ```yaml
   version: 1
   project_url: https://github.com/orgs/my-org/projects/1

   code_repos:
     - repo: my-org/api
       dir: api                         # 在 workspace_dir 底下
       ci: make ci                      # 本地 CI 對應指令,全綠才算過
       ci_optional:
         - when: docker                 # docker info 成功才跑
           run: make integration

     - repo: my-org/web
       dir: web
       ci: pnpm check
       generated:                       # 合併衝突時不手解:取 main 的版本再重新產生
         - paths: [packages/api-client/openapi.json, packages/api-client/src/generated/]
           regenerate: pnpm codegen
       prototype_guide: prototype-guides/web.md   # planning repo 內的路徑,選用

   one_way_doors:                       # 追加在預設清單之後,選用
     - 對使用者發送的推播訊息

   limits:
     per_round: 3

   models:
     implementer_default: sonnet
     implementer_max: opus
     reviewer: opus
   ```

### M3. 建立 label

planning repo 與每個 code repo 都要建立 label。腳本可以重複執行。

注意:執行 `claude plugin list --json`。`installPath` 欄位是 plugin 的安裝路徑。

1. 執行 `setup-labels.sh`。參數為 planning repo 與每個 code repo:

   ```bash
   bash "<plugin 安裝路徑>/scripts/setup-labels.sh" my-org/planning my-org/api my-org/web
   ```

### M4. 建立 org Project 看板

警告:Status 值必須與 [03 章的 13 個欄位](03-board.md#看板的-13-個欄位)**逐字一致**,包含 emoji 與空格。不同時,看板不會更新。

1. 在 org 的 Projects 頁面,選 New project → Board。檢視要是 Board,Column by 選 Status。
2. 把 Status 欄位的選項改成 13 個欄位值。
3. 加入 Sub-issues progress、Assignees、Repository 欄位。
4. 在 Workflows 中,設定 Auto-add to project,指向 planning repo。

### M5. 設定 secrets 與 variables

1. 在 planning repo 與每個 code repo,設定以下項目:

   | 名稱 | 種類 | 說明 |
   | --- | --- | --- |
   | `PROJECT_URL` | variable | 看板網址 |
   | `PLANNING_REPO` | variable | planning repo 的 `owner/name`。沒有設定時,`promote-queued` 會失敗 |
   | `PROJECT_TOKEN` | secret | classic PAT,用來寫看板 |
   | `ORG_AUTOMATION_TOKEN` | secret | classic PAT,用來跨 repo 升級 implementation ticket(可以與 `PROJECT_TOKEN` 相同) |

### M6. 部署 workflow

1. 把 plugin 的 `workflows/` 中的三個檔案,放進每個 repo 的 `.github/workflows/`。

注意:在 code repo,建議用 PR 加入這三個檔案。

> 下一步:[05 · 啟動與疑難排解](05-operations.md)
