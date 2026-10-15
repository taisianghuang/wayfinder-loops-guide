# 11 · 路線圖

---

## 現況

| 階段 | 內容 | 狀態 |
| --- | --- | --- |
| 內部使用 | 五條 loop 以個人 slash command 跑在單一組織 | ✅ 已運轉 |
| 獨立 reviewer | 實作與審查拆成兩個 context、Merge Danger、審這裡清單 | ✅ 已上線 |
| 第一階段 | 包成 Claude Code plugin、寫死的值改讀 `wayfinder.yml`、doctor 健檢 | 🚧 PR 審查中 |
| 第二階段 | `/wayfinder-loops:setup` 安裝精靈、在乾淨的 org 從零驗證 | ⏳ 規劃中 |
| 第三階段 | 發佈前檢查、版本 0.1.0、plugin repo 公開 | ⏳ 規劃中 |

## 第二階段:setup 精靈

把 [08 篇](08-install.md)的步驟 3 到 7 變成一個互動精靈,每一步先檢查、做過就跳過(可以重跑):

1. 前置檢查:`gh` 版本、token scope、mattpocock-skills。
2. 問 planning repo、code repo 與 CI 指令 → 產生 `wayfinder.yml`,開 PR 進 planning。
3. 建齊 label。
4. 驗證或建立看板與 13 個 Status 選項。
5. 把 workflows 以 PR 送進每個 repo。
6. 只有人能做的步驟(建 PAT、設 secret)列成清單,等人回報後驗證。
7. 冒煙測試。

## 之後

### retro loop

同一則審查意見,不應該寫第二次。每週一次,從這些資料找重複出現的問題:

- 整合分支上 `review:` 開頭的 commit(依規則編號計數:實作者一直犯什麼錯)
- spec→main PR 上人類的留言與修正 commit
- `agent:blocked` 的原因、收割第二層的紅燈

對每類重複問題,依序考慮:能不能變成**確定性檢查**(lint、CI 規則)→ 能不能寫成 `CODING_STANDARDS.md` 的一條 → 是不是 `AGENTS.md` 缺一個導航指標。產出是 planning 上的決策票,走既有的決策流程,不直接改 repo。

### CODING_STANDARDS onboarding

選用的一次性流程:盤點 code repo 既有的 `AGENTS.md`、`CLAUDE.md`、架構文件,把每條規則分成「導航」「機械可檢查」「需要判斷」「實作者的工作方式」「過時」五類,產出 `CODING_STANDARDS.md` 草稿與瘦身後的 `AGENTS.md`,由人逐類核准。

### 手動審查入口

`/wayfinder-loops:review <PR>`:對任何 PR(包括人自己開的)跑同一個 reviewer agent,規則只有一份。

## 致謝

- [Matt Pocock](https://github.com/mattpocock/skills) 的 wayfinder 與工程類 skills 是這套流程的地基。
- 三層把關、獨立 reviewer、Merge Danger 與 retro 的想法來自 Matt Pocock 的演講〈Fixing the PR Bottleneck〉(AI Engineer Paris 2026)。

---

> 回到 [00 · 總覽](00-overview.md)
