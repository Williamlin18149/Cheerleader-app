# 專案文件索引（給人與 AI 用）

## 任何人／AI 請先讀這段（整專案在幹嘛）

本專案是 **一個給所有人共用的網站網址**（前端靜態頁），搭配 **Firebase Realtime Database** 做 **雲端即時資料**。因此：

- **GitHub**：放原始碼（例如 `index.html`、本 `docs/`），版本控管與協作。
- **Netlify**（或同類靜態託管）：從 GitHub **建置／發布**同一套前端，讓大家 **永遠打開同一個正式網址** 使用（網站統一一個）。
- **Firebase**：承載 **房間內選擇、成員狀態、可選的全域啦啦隊名單**；多人同時開同一網址、輸入同一房間碼時，資料透過 RTDB **即時同步**，不需自架後端 API。

一句話：**GitHub 管程式碼 → Netlify 管「唯一公開網站」→ Firebase 管「大家同步的資料」**。

更細的部署與資料路徑見：[01-專案總覽.md](./01-專案總覽.md)（含本流程圖）、[03-Firebase與資料.md](./03-Firebase與資料.md)。

---

本目錄整理 **台灣啦啦隊支持表** 單頁應用的架構、資料流與修改方式。日後改版或交給 AI 協助時，建議依序閱讀（或一併貼上）下列檔案：

| 檔案 | 內容摘要 |
|------|----------|
| [01-專案總覽.md](./01-專案總覽.md) | 專案目的、**GitHub／Netlify／Firebase 與單一網址同步**、檔案地圖、依賴與機密注意 |
| [02-技術架構.md](./02-技術架構.md) | 單一 `index.html`、React 寫法、畫面階段與模組分界 |
| [03-Firebase與資料.md](./03-Firebase與資料.md) | Realtime Database 路徑、成員資料結構、合併邏輯 |
| [04-功能模組.md](./04-功能模組.md) | 分頁功能、共同支持兩種模式、管理後台 |
| [05-後續修改與驗證.md](./05-後續修改與驗證.md) | 常改位置、密鑰注意、`scripts/check-inline.mjs` |

**單一真相來源（程式）**：行為與常數以 `index.html` 內嵌腳本為準；本文件為輔助說明，若與程式不一致以程式為準。

## 建議給 AI 的最小提示包（複製貼上）

以下整段可與 `index.html` 一併貼給 AI：

```text
台灣啦啦隊支持表：單一 index.html（React createElement + Firebase RTDB），無 npm 建置。
程式碼在 GitHub；網站由 Netlify 發布成「唯一公開網址」；多人即時同步靠 Firebase Realtime Database（rooms/...、config/members）。
請依序閱讀專案內 docs/README.md、docs/01-專案總覽.md、docs/03-Firebase與資料.md 的說明後再改程式。
```
