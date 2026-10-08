# DEPLOY ONCE

- GitHub：`saintdou-weng/vrt-asset` 以本包取代，Pages 用 main / root。網址 https://saintdou-weng.github.io/vrt-asset/
- Telegram：顯示名 VRT ERP、技術名 PSJERP、指令 /erp /psj /psjerp。沿用現有 Token（不更換），只放 Apps Script。
- Apps Script：專案 PSJERP、時區 Asia/Phnom_Penh、Web App（Me / Anyone）。改程式後一律「管理部署作業 → 編輯 → 新版本」，網址不變。
- 雲端同步：索引 PSJERP_ERPDATA ＋ 每種資料一個 PSJERP_ERP_*（一批次一分頁）；讀寫都要同步密碼；每台裝置輸入一次。
- 排程：週一至週六約 15:30；週日與 HOLIDAYS 跳過。
- Legacy ERP 永遠唯讀；ERP 來源資料在雲端鎖定，只能改備註 / 標籤。
