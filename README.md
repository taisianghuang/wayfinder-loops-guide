# wayfinder-loops 報告書

wayfinder-loops 工作流的說明網站。wayfinder-loops 是一個 Claude Code plugin。它用 GitHub Issues、GitHub Project 看板與五條 `/loop`,把一個想法推進到合併進 main 的程式碼。人只做兩件事:在決策票上選一個選項,審查並合併整合 PR。

📖 線上閱讀:https://taisianghuang.github.io/wayfinder-loops-guide/

## 內容

| # | 章節 | 內容 |
| --- | --- | --- |
| 00 | 總覽 | wayfinder-loops 是什麼、流程、名詞 |
| 01 | 六個角色 | 五條 loop、reviewer agent、三個 workflow 的工作 |
| 02 | 一張票從想法到 main | 一個想法經過的每個階段 |
| 03 | 用看板管理 | 看板欄位、人的三個動作、label |
| 04 | 建立工作流 | 從零安裝與設定的步驟 |
| 05 | 啟動與疑難排解 | 啟動 loop、常見問題、更新與移除 |

## 本機預覽

這個網站用 [docsify](https://docsify.js.org) 製作,沒有建置步驟。

注意:不要用 `file://` 開啟 `index.html`。docsify 要透過 HTTP 載入 markdown。

1. 在本資料夾啟動一個靜態伺服器:`python3 -m http.server 3000`
2. 開啟 `http://localhost:3000`。

## 部署

GitHub Pages 從 `main` 分支的根目錄部署。`.nojekyll` 讓 Pages 不忽略 `_sidebar.md`、`_coverpage.md` 這類底線開頭的檔案。
