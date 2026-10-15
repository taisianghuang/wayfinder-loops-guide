# 08 · 安裝與設定

> 目標:從零到看板上出現第一張「⏳ 等你決定」的票。

---

?> **狀態**:plugin 的 repo 目前仍是 private,公開前只有受邀的人能安裝。第二階段會提供 `/wayfinder-loops:setup` 精靈自動完成步驟 3 到 7。

## 前置需求

| 項目 | 說明 |
| --- | --- |
| Claude Code | 2.1.275 以上 |
| mattpocock-skills | 安裝本 plugin 時會一起裝 |
| `gh` CLI | 2.94 以上(要用 `--blocked-by`、sub-issue 等功能) |
| `gh` token scope | 含 `project`:`gh auth refresh -h github.com -s project` |
| GitHub organization | 一個 planning repo + 一或多個 code repo + 一個 **org** Project 看板(Free 方案可用;個人帳號的 Project 不支援) |
| classic PAT | `repo` + `project` scope,給 workflow 寫看板與跨 repo 升級票 |

## 1. 安裝 plugin

在 Claude Code 裡執行:

```text
/plugin marketplace add taisianghuang/wayfinder-loops
/plugin install wayfinder-loops@wayfinder-loops
```

安裝時會問兩個值:

| 設定 | 意思 | 例 |
| --- | --- | --- |
| `planning_repo` | 放想法、map、決策票與 `wayfinder.yml` 的 repo | `my-org/planning` |
| `workspace_dir` | code repo 的本機 clone 都放在這底下;loop 只在這裡面工作 | `~/wayfinder-workspace` |

之後可以用 `/plugin` → wayfinder-loops → Configure 修改。

## 2. clone code repo

把每個 code repo clone 到 `workspace_dir` 底下。資料夾名稱要跟下一步 `wayfinder.yml` 的 `dir` 一致。

## 3. 寫 `wayfinder.yml`

放在 planning repo 根目錄。這是整個團隊共用、要版控的設定:

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

`ci` 是整套品質把關的地基:它必須能在本機一次跑完,而且跟 GitHub 上的 CI 檢查同一組東西。

## 4. 建 label

三類 repo 都要。腳本是冪等的,可以重跑:

```bash
bash "<plugin 安裝路徑>/scripts/setup-labels.sh" my-org/planning my-org/api my-org/web
```

plugin 安裝路徑可以用 `claude plugin list --json` 查到。

## 5. 建 org Project 看板

1. 在 org 的 Projects 頁面 New project → Board。
2. 把 Status 欄位的選項改成 [07 篇](07-labels-board.md#看板的-13-個欄位)列出的 13 個值,**逐字一致**。
3. 加上 Sub-issues progress、Assignees、Repository 欄位。
4. Workflows → Auto-add to project,指向 planning repo。

## 6. 設 secrets 與 variables

planning repo 與每個 code repo 都要設:

| 名稱 | 種類 | 說明 |
| --- | --- | --- |
| `PROJECT_URL` | variable | 看板網址 |
| `PLANNING_REPO` | variable | planning repo 的 `owner/name`;沒設時 `promote-queued` 會直接報錯 |
| `PROJECT_TOKEN` | secret | classic PAT,寫看板用 |
| `ORG_AUTOMATION_TOKEN` | secret | classic PAT,跨 repo 升級票用(可與上面同一顆) |

## 7. 部署 workflows

把 plugin 的 `workflows/` 三個檔案放進每個 repo 的 `.github/workflows/`。code repo 建議走 PR 合入。

## 8. 健檢

```text
/wayfinder-loops:doctor
```

它只讀不寫,逐項回報 ✅ / ❌:設定是否生效、`gh` 版本與 scope、`wayfinder.yml` 讀不讀得到、每個 clone 的 origin 是否正確、mattpocock-skills 是否安裝、各 repo 的 variables 與 secrets。全部 ✅ 再往下。

## 9. 冒煙測試

1. 在 planning repo 開一張 issue,寫一個小而模糊的想法。
2. 手動跑一次 `/wayfinder-loops:cartographer`。
3. 看板上應該出現 map(🗺️ 規劃中)與決策票(⏳ 等你決定),票 assign 給你。
4. 對一張決策票貼 `decision:approve`,手動跑一次 `/wayfinder-loops:resolver`。
5. 票應該被關閉,map 收到一則 `🤖 [resolver]` 留言。

都通了,就可以照 [09 篇](09-operations.md)常駐五條 loop。

## 選用:讓 reviewer 有標準可守

在 code repo 根目錄放一份 `CODING_STANDARDS.md`。原則:

- 每條規則寫成 reviewer 看 diff 就能回答「是/否」的問題,編號 R-01、R-02…
- 機械性的規則(固定語法、禁用 API、import 形狀)不要寫進去,改成 lint 或 CI 規則。
- 不要放進 `AGENTS.md` / `CLAUDE.md`:那兩個檔每次都會塞進實作者的 context,而實作者刻意不負責守標準。

沒有這份檔案時,reviewer 照樣會審規格、會說謊的測試與 code smell。

> 下一步:[09 · 運轉、成本與疑難排解](09-operations.md)
