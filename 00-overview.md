# 00 · 總覽

> wayfinder-loops 是一個 Claude Code plugin。它在你的 GitHub Issues 與 Project 看板上跑五條 `/loop`,把模糊的想法一路推進到 Pull Request。人類只出現在兩個地方:**在看板上點一個決策**,以及**審整份 spec 的最後那一張 PR**。

---

## 一句話

**PM 丟想法、點決策;機器開票、查證、寫 code、審 code、收割;工程師只審一次。**

## 全流程一張圖

<div class="arch-embed" style="--h:690px;--h-wide:829px"><iframe src="diagrams/00-pipeline.html" loading="lazy" title="wayfinder-loops 操作管線"></iframe></div>
<p class="arch-open"><a href="diagrams/00-pipeline.html" target="_blank" rel="noopener">全頁開啟這張圖 ↗</a></p>

## 角色一覽

| 角色 | 形式 | 一句話 |
| --- | --- | --- |
| Cartographer | loop(20 分鐘) | 把想法畫成地圖、開決策票、決策定了就寫 spec 與切實作票 |
| Resolver | loop(10 分鐘) | 讀人類的決定,記錄結論、關票;人有疑慮就回覆 |
| Researcher | loop(15 分鐘) | 純 AFK 查證外部事實,貼回票上結案 |
| Implementer | loop(15 分鐘) | 收割、認領實作票、平行派工、派 reviewer、維護 spec→main PR |
| Prototyper | loop(30 分鐘) | 人說「畫」之後才動,做手機點得開的原型頁 |
| reviewer | agent(由 Implementer 派) | 獨立審每張批次 PR、直接 commit 修正、判斷單向門 |
| 人類 | — | 丟想法、點決策、審最後一張 PR |

為什麼 reviewer 是 agent 而不是第六條 loop,見 [10 · 設計取捨](10-lessons.md#reviewer-為什麼是-agent-不是-loop)。

## 適合誰

- 小團隊,PM / SA 不碰程式碼,但要能自己丟需求、做決策。
- 已經用 GitHub Issues 管需求,願意用 GitHub Project 看板。
- 有一台能長時間開著 Claude Code 的機器,code repo 有能在本機跑完的 CI 指令。

**不適合**:沒有本機 CI 指令的 repo(批次 PR 的驗證全靠它)、GitHub 以外的 issue tracker、不能讓 agent 以你的 GitHub 帳號開票與開 PR 的環境。

## 建立在什麼之上

| 元件 | 角色 |
| --- | --- |
| [mattpocock/skills](https://github.com/mattpocock/skills) | wayfinder(畫地圖)、to-spec、to-tickets、implement、tdd、code-review、pr 這些「怎麼做」的方法 |
| Claude Code `/loop` | 讓每條 loop 依固定頻率跑一輪 |
| GitHub Issues + sub-issues + dependencies | 所有狀態的唯一真相 |
| GitHub Project 看板 | 給人看的介面,欄位由 workflow 從 label 算出來 |
| 三個 GitHub Actions workflow | 升級實作票、鏡射看板狀態、決策事件 hook |

## 閱讀路徑

| # | 篇章 | 給誰 |
| --- | --- | --- |
| 01 | [為什麼需要這套工作流](01-why.md) | 所有人 |
| 02 | [核心概念](02-concepts.md) | 所有人 |
| 03 | [人類的一天](03-human-day.md) | PM、SA、工程師 |
| 04 | [六個角色](04-roles.md) | 想知道機器在做什麼的人 |
| 05 | [一張票從想法到 main](05-journey.md) | 所有人 |
| 06 | [品質的三層把關](06-quality-gates.md) | 工程師、技術主管 |
| 07 | [Label、看板與不變式](07-labels-board.md) | 維運者 |
| 08 | [安裝與設定](08-install.md) | 要導入的人 |
| 09 | [運轉、成本與疑難排解](09-operations.md) | 維運者 |
| 10 | [設計取捨與踩過的坑](10-lessons.md) | 想改這套系統的人 |
| 11 | [路線圖](11-roadmap.md) | 所有人 |

> 下一步:[01 · 為什麼需要這套工作流](01-why.md)
