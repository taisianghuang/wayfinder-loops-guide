# 04 · 建立工作流

> 本章給要導入 wayfinder-loops 的人。本章說明從零建立工作流的 9 個步驟。完成後,看板上出現第一張「⏳ 等你決定」的決策票。

---

?> **狀態**:plugin 的 repo 目前是 private。公開前,只有受邀的人可以安裝。第二階段會提供 `/wayfinder-loops:setup` 精靈,自動執行步驟 3 到 7。

## 前置需求

### 適用條件

| 條件 | 說明 |
| --- | --- |
| 團隊 | 小團隊。PM / SA 不寫程式,但要自己提出需求、做決定。 |
| issue tracker | 已經用 GitHub Issues 管理需求,可以使用 GitHub Project 看板。 |
| 機器 | 一台可以長時間執行 Claude Code 的機器。 |
| CI | 每個 code repo 有一個能在本機執行完成的 CI 指令。 |

以下情況不適用:

- code repo 沒有本機 CI 指令。批次 PR 的驗證依賴這個指令。
- 使用 GitHub 以外的 issue tracker。
- 不能讓 agent 用你的 GitHub 帳號開票與開 PR。

### 工具與帳號

| 項目 | 說明 |
| --- | --- |
| Claude Code | 2.1.275 或更新版本 |
| mattpocock-skills | 安裝本 plugin 時一起安裝 |
| `gh` CLI | 2.94 或更新版本(需要 `--blocked-by`、sub-issue 等功能) |
| `gh` token scope | 含 `project`:`gh auth refresh -h github.com -s project` |
| GitHub organization | 一個 planning repo、一個或多個 code repo、一個 **org** Project 看板(Free 方案可用;個人帳號的 Project 不支援) |
| classic PAT | scope 為 `repo` 與 `project`。workflow 用它寫看板與跨 repo 升級實作票 |

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
   | `planning_repo` | 放想法、map、決策票與 `wayfinder.yml` 的 repo | `my-org/planning` |
   | `workspace_dir` | 放 code repo 本機 clone 的資料夾。loop 只在這個資料夾中工作 | `~/wayfinder-workspace` |

注意:要修改這兩個值,執行 `/plugin`,選 wayfinder-loops,選 Configure。

## 2. clone code repo

注意:資料夾名稱必須與步驟 3 中 `wayfinder.yml` 的 `dir` 相同。

1. 把每個 code repo clone 到 `workspace_dir` 中。

## 3. 寫 `wayfinder.yml`

`wayfinder.yml` 是整個團隊共用的設定。它要納入版本控制。

注意:`ci` 指令必須能在本機一次執行完成。`ci` 指令必須與 GitHub 上的 CI 檢查相同。批次 PR 的驗證依賴這個指令。

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

## 4. 建立 label

planning repo 與每個 code repo 都要建立 label。腳本可以重複執行。

注意:執行 `claude plugin list --json`。`installPath` 欄位是 plugin 的安裝路徑。

1. 執行 `setup-labels.sh`。參數為 planning repo 與每個 code repo:

   ```bash
   bash "<plugin 安裝路徑>/scripts/setup-labels.sh" my-org/planning my-org/api my-org/web
   ```

## 5. 建立 org Project 看板

警告:Status 值必須與 [03 章的 13 個欄位](03-board.md#看板的-13-個欄位)**逐字一致**,包含 emoji 與空格。不同時,看板不會更新。

1. 在 org 的 Projects 頁面,選 New project → Board。
2. 把 Status 欄位的選項改成 13 個欄位值。
3. 加入 Sub-issues progress、Assignees、Repository 欄位。
4. 在 Workflows 中,設定 Auto-add to project,指向 planning repo。

## 6. 設定 secrets 與 variables

1. 在 planning repo 與每個 code repo,設定以下項目:

   | 名稱 | 種類 | 說明 |
   | --- | --- | --- |
   | `PROJECT_URL` | variable | 看板網址 |
   | `PLANNING_REPO` | variable | planning repo 的 `owner/name`。沒有設定時,`promote-queued` 會失敗 |
   | `PROJECT_TOKEN` | secret | classic PAT,用來寫看板 |
   | `ORG_AUTOMATION_TOKEN` | secret | classic PAT,用來跨 repo 升級實作票(可以與 `PROJECT_TOKEN` 相同) |

## 7. 部署 workflow

1. 把 plugin 的 `workflows/` 中的三個檔案,放進每個 repo 的 `.github/workflows/`。

注意:在 code repo,建議用 PR 加入這三個檔案。

## 8. 健檢

doctor 只讀取,不寫入。它逐項回報 ✅ 或 ❌:設定是否生效、`gh` 版本與 scope、`wayfinder.yml` 能否讀取、每個 clone 的 origin 是否正確、mattpocock-skills 是否安裝、每個 repo 的 variables 與 secrets。

1. 執行:

   ```text
   /wayfinder-loops:doctor
   ```

2. 確認所有項目是 ✅。有 ❌ 時,依 doctor 的說明修正,再執行步驟 1。

## 9. 冒煙測試

1. 在 planning repo 開一個 issue。寫一個小而模糊的想法。
2. 手動執行一次 `/wayfinder-loops:cartographer`。
3. 確認看板上出現 map(🗺️ 規劃中)與決策票(⏳ 等你決定)。決策票指派給你。
4. 在一張決策票上貼 `decision:approve`。
5. 手動執行一次 `/wayfinder-loops:resolver`。
6. 確認決策票已關閉,map 上有一則 `🤖 [resolver]` 留言。

所有結果正確時,依 [05 章](05-operations.md)啟動五條 loop。

> 下一步:[05 · 啟動與疑難排解](05-operations.md)
