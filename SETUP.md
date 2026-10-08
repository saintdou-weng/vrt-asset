# PSJERP — 安裝 / 更新步驟

GitHub repo：`saintdou-weng/vrt-asset`　網頁：`https://saintdou-weng.github.io/vrt-asset/`

## A. 從 v0.5.1 更新到 v0.6.0（全部 ERP 報表、拖檔即匯入、雲端拆檔）

1. **Apps Script**：打開 PSJERP 專案 → `PSJERP.gs` 內容全選刪掉 → 貼上「PSJERP_AppsScript專用_v0.6.0.gs」（Token、Chat ID、同步密碼都已填好）→ 儲存。
2. **部署 → 管理部署作業 → 鉛筆（編輯）→ 版本選「新版本」→ 部署**。/exec 網址不變，Telegram 不用重設。
3. 上方選函式 `CHECK_CLOUD` → 執行 → 若要求 Google 權限就允許。
4. **GitHub**：把本包 `index.html`、`PSJERP.gs`（密鑰空白版）、文件上傳覆蓋。
5. 打開網頁 → 把 `下載\ERP_export` 裡的檔案直接拖進網頁（可以一次一整個資料夾的檔）→ 確認匯入。雲端會自動上傳；第一次上傳 BOM（35,000 張訂單）約需 10 分鐘，網頁右上角雲朵會顯示進度。

v0.5.1 已上傳的資料不會遺失：網頁照樣讀得到；同一個檔重匯一次就會搬到新格式。

## B. 全新安裝（只有第一次）

1. GitHub：repo 刪除舊檔，上傳本資料夾全部檔案（含 `.nojekyll`）；Settings → Pages：main / root。
2. Apps Script：新專案 **PSJERP**，貼上已填好的 .gs；Project Settings → 時區 `Asia/Phnom_Penh`。
3. Deploy → New deployment → Web app：Execute as **Me**、Who has access **Anyone**，複製 `/exec`。
4. 執行一次 `INSTALL_ONCE()`（建立 PSJERP_DATA、Telegram 選單、webhook、/erp /psj /psjerp、每日排程）。
5. 執行一次 `CHECK_CLOUD()`（建立 PSJERP_ERPDATA）。
6. 網頁 → 設定 → 後端連線貼 `/exec` → 測試；雲端同步輸入同步密碼。

## Google 試算表

| 試算表 | 內容 |
|---|---|
| `PSJERP_DATA` | SETTINGS / MENU / HOLIDAYS / 各種 LOG（Telegram、排程） |
| `PSJERP_ERPDATA` | 索引：ERP_FILES（每個匯入批次一列，記錄放在哪個試算表、哪個分頁）、NOTES（備註）、TEMPLATES（欄位對應範本） |
| `PSJERP_ERP_<資料集>` | 資料：ORDERS、BOM、PURCHASE、PURCHASE_PRICE、STOCK_MOVE、FABRIC、ACCESS、FG_STOCK、FG、SHIP、SMV、LABOR_SUMMARY；**每個匯入批次一個分頁**（例 `cst580_2024-01-01_2024-06-30`）；快到 900 萬格時自動開 `_2`、`_3` |

這些試算表都是網頁自動寫的，請不要手動改內容；要重來就在網頁重新匯入同一份 ERP 檔（同範圍＝取代）。

## 維護（不用改程式）

- 選單：改 `MENU` 表。廠休日：改 `HOLIDAYS` 表。同步時間：改 `SETTINGS` 的 `DAILY_SYNC_TIME` 後執行 `REINSTALL_TRIGGER()`。
- 換同步密碼：改 .gs 最上面 `SYNC_KEY_DIRECT` → 儲存 → 部署新版本 → 每台裝置重新輸入。
