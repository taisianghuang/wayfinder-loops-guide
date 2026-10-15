# wayfinder-loops 報告書

wayfinder-loops 工作流的說明網站。wayfinder-loops 是一個 Claude Code plugin:它用 GitHub Issues、Project 看板與五條 `/loop`,把模糊的想法推進到 Pull Request。人只做兩件事:做決策、審最後一張 PR。

📖 線上閱讀:https://taisianghuang.github.io/wayfinder-loops-guide/

## 內容

| # | 章節 |
| --- | --- |
| 00 | 總覽 |
| 01 | 為什麼需要這套工作流 |
| 02 | 核心概念 |
| 03 | 人類的一天 |
| 04 | 六個角色 |
| 05 | 一張票從想法到 main |
| 06 | 品質的三層把關 |
| 07 | Label、看板與不變式 |
| 08 | 安裝與設定 |
| 09 | 運轉、成本與疑難排解 |
| 10 | 設計取捨與踩過的坑 |
| 11 | 路線圖 |

## 本機預覽

這個網站用 [docsify](https://docsify.js.org) 製作,沒有建置步驟。

1. 在本資料夾啟動一個靜態伺服器:`python3 -m http.server 3000`
2. 打開 `http://localhost:3000`。

不要直接用 `file://` 開 `index.html`。docsify 要透過 HTTP 載入 markdown。

## 部署

GitHub Pages 從 `main` 分支的根目錄部署。`.nojekyll` 讓 Pages 不忽略 `_sidebar.md`、`_coverpage.md` 這類底線開頭的檔案。
