# 03 — Firebase 與資料

## 與「唯一網站、多人同步」的關係

**Firebase** 只負責 **雲端資料與即時推送**；所有人看到的 **同一套網頁** 來自 **Netlify（或類似）託管的 `index.html`**，原始碼則在 **GitHub**。瀏覽器載入頁面後，用內嵌的 `FIREBASE_CONFIG` 連到本專案的 Realtime Database，因此多人開 **同一網址**、輸入 **同一房間碼** 時會即時同步。部署總覽見 [01-專案總覽.md](./01-專案總覽.md)。

## Firebase 產品

- **Realtime Database（RTDB）**：房間即時狀態、**啦啦隊主名單**（`config/members`）。
- **本專案未使用** Firebase Authentication（登入房間仍為自訂暱稱 + 房間碼；管理員為 HTML 內密碼）。

## 內嵌設定

`index.html` 頂部內嵌腳本含：

- `FIREBASE_CONFIG` — `firebase.initializeApp` 用
- `firebase.database()` → `db`

實際 `databaseURL`、專案 id 以檔案為準。

## RTDB 路徑約定

### 房間與成員選擇

- `rooms/{roomCode}/members/{nickname}`
  - 常見欄位：`name`（暱稱）、`faves`（陣列，啦啦隊員 **id**）、`ts`（時間戳 server value）

房內即時監聽（進入 `app` 後）大致為：

- `db.ref(\`rooms/${roomCode}/members\`)` — 整包 `roomData`（object keyed by 暱稱）

寫入個人選擇時會 `update` 該暱稱節點下的 `faves` 與 `ts`。

### 全域啦啦隊名單（唯一真相來源）

- `config/members` — **陣列**：每位啦啦隊員一筆 object

啟動時 `once("value")` 讀取；結果經 **`membersFromFirebase`** 正規化（過濾無效項、`id` 轉數字、`sortMembersByTeam` 排序）後寫入 React `members`。

- **無**程式內建名單與雲端「合併補齊」：雲端沒有或不是陣列時，`members` 為 **`[]`**。
- **大調整名單**：請在 Firebase Console **匯出 JSON**、編輯後再 **匯入／覆寫** `config/members`，或於管理後台逐筆編輯。

## 啦啦隊員資料結構（概念 schema）

陣列中每一筆大致為：

| 欄位 | 型別 | 說明 |
|------|------|------|
| `id` | number | 穩定主鍵；`faves` 存的是此 id |
| `n` | string | 顯示名稱 |
| `nat` | string | 國籍代碼：`K` 韓、`J` 日、`H` 港澳、`T` 台（見 `NF` 對照） |
| `t` | string[] | 所屬球隊 id（對應 `TEAMS` 的 key，如 `dragon`、`dreamers`） |
| `r` | string | 身份／角色（隊長、外援、正式…，對應 `RS` 樣式） |
| `note` | string? | 選填備註 |

球隊與聯盟對照在 `TEAMS`、`TEAM_BY_LEAGUE` 常數表（仍內建在程式，僅作為 **UI 與篩選對照表**，不是啦啦隊員名單本體）。

## `membersFromFirebase(val)` 行為

- `val` 非陣列或空陣列 → 回傳 `[]`。
- 否則過濾非物件、無效 `id`，其餘 `{...c, id}` 再經 `sortMembersByTeam` 排序。

## `cheerleader-app-default-rtdb-export.json`

- 多為 Firebase 匯出的 JSON 樹，結構上常含 `config`、`rooms` 等。
- 可用於本機對照、還原測試；與執行中 App 是否一致取決於你是否將其匯入同一專案 RTDB。

## Security Rules（本文件不寫死規則內文）

實際讀寫權限以 Firebase Console 的 **Realtime Database Rules** 為準。若後台無法讀 `rooms` 根、只能讀子路徑，程式已有對應降級行為。
