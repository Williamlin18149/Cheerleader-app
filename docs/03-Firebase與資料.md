# 03 — Firebase 與資料

## 與「唯一網站、多人同步」的關係

**Firebase** 只負責 **雲端資料與即時推送**；所有人看到的 **同一套網頁** 來自 **Netlify（或類似）託管的 `index.html`**，原始碼則在 **GitHub**。瀏覽器載入頁面後，用內嵌的 `FIREBASE_CONFIG` 連到本專案的 Realtime Database，因此多人開 **同一網址**、輸入 **同一房間碼** 時會即時同步。部署總覽見 [01-專案總覽.md](./01-專案總覽.md)。

## Firebase 產品

- **Realtime Database（RTDB）**：房間即時狀態、可選的全域啦啦隊名單。
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

### 全域啦啦隊名單（可選）

- `config/members` — **陣列**：每位啦啦隊員一筆 object（見下節 schema）

啟動時 `once("value")` 讀取；若存在且為非空陣列，則與程式內 `DEFAULT_MEMBERS` 做 **合併**（非整包覆蓋）。

### 管理後台

- 可能讀取 `rooms` 根節點列出多房；若規則不允許，程式會退回只讀目前房間的 `rooms/{roomCode}/members`（見程式內註解與 `adminRoomsNotice`）。

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

球隊與聯盟對照在 `TEAMS`、`TEAM_BY_LEAGUE` 常數表。

## `mergeMembersWithDefaults(custom, defs)` 行為（重要）

目的：**雲端舊資料不會把程式新版本新增的預設成員「弄丟」**。

- 以 `defs`（`DEFAULT_MEMBERS`）為骨架，依 **id** 合併 `custom` 中同 id 的覆寫欄位。
- `custom` 裡有、但 `defs` 沒有的 id，會 **append** 到結果尾端（自訂新增成員）。

因此：**新增預設成員時只要在 `DEFAULT_MEMBERS` 加一筆新 id**；使用者雲端若尚未含該 id，下次載入合併後仍會出現。

## `cheerleader-app-default-rtdb-export.json`

- 多為 Firebase 匯出的 JSON 樹，結構上常含 `config`、`rooms` 等。
- 可用於本機對照、還原測試；**與執行中 App 是否一致**取決於你是否將其匯入同一專案 RTDB。

## Security Rules（本文件不寫死規則內文）

實際讀寫權限以 Firebase Console 的 **Realtime Database Rules** 為準。若後台無法讀 `rooms` 根、只能讀子路徑，程式已有對應降級行為。
