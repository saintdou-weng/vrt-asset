# VRT ERP / PSJERP — Claude 完整開發指令 V2

> 使用者介面名稱：**VRT ERP**  
技術專案代號：**PSJERP**
>
> 公司：**Vantage River Textiles Co., Ltd. (VRT)**
>
> 目的：建立一套適合 VRT 小型成衣廠使用的雲端 ERP，整合現有 Legacy ERP `clothes.exe`、VRT Prod、生產資料、Account Flow、Excel/CSV、Google Sheets、Telegram，並保留各平台可獨立使用的方式。
>
> 本文件為 Claude 的**完整開發與執行指令**。請依本文件執行，不要自行改變核心架構、安全規則或資料權限。

---

# 0. 最重要的總原則

## 0.1 Legacy ERP 永遠唯讀

現有舊 ERP：

- 程式：`clothes.exe`
- 系統版本：18MC26.16
- SQL Server / Legacy ERP Database
- 現有 VRT ERP 歷史資料

**一律視為 READ ONLY。**

Claude、VRT Bridge、任何自動化工具都只能：

- 查詢
- 讀取
- 開啟報表
- 下載
- 匯出
- 擷取
- 比對
- 解析
- 建立唯讀快照

### 絕對禁止

未經 Paul 明確核可，不得對 Legacy ERP / SQL Server 執行：

- 新增
- 修改
- 編輯
- 刪除
- 存檔
- 匯入
- 更新
- 重算
- 結案
- 年度結轉
- 轉採購
- 庫存調整
- 單價修改
- 幣別匯率修改
- 寫入 SQL
- `INSERT`
- `UPDATE`
- `DELETE`
- 任何會改動正式資料的 Stored Procedure
- 任何可能寫回 Legacy ERP 的操作

如果任何功能必須寫入才可驗證：

**立即停止。**

回報 Paul：

```text
功能：
想執行的動作：
目的：
可能寫入的資料：
風險：
是否有唯讀替代方案：
```

等 Paul 明確核可後才可繼續。

---

## 0.2 VRT ERP Cloud 可以正常 CRUD

新的 VRT ERP Cloud 與 Legacy ERP 不同。

VRT ERP 必須正常支援：

- New
- Add
- Edit
- Save
- Delete
- Upload
- Download
- Import Excel
- Export Excel
- Attachment
- Merge
- Update
- Replace selected records

**不需要 Approval Workflow。**

但所有操作都必須保留 Audit Log。

---

# 1. 專案目標

建立一套：

- 雲端
- 三語
- 手機 / 平板 / PC 可用
- 操作直覺
- 介面比傳統 ERP 簡單
- 可單獨開啟每個模組
- 也可從總平台統一管理
- 可跟 Legacy ERP 即時或準即時同步
- 可跟現有 VRT Prod / Account Flow 聯結
- 可智慧匯入 Excel / CSV
- 可做資料比對
- 可發 Telegram 摘要
- 可做 BOM / Cost / WIP / FG / Inventory / Shipping / Production / Margin 分析

的 VRT 自有 ERP。

---

# 1A. 正式資料主從關係

本專案的正式原則改為：

```text
Legacy ERP = 主要正式系統 / System of Record
PSJERP Cloud = 雲端鏡像 + 報表 + 比對 + AI + Telegram
VRT Prod = 生產專業子系統
Account Flow = 財務 / WIP / 製造費用來源之一
```

不要把 PSJERP Cloud 當成取代原 ERP 的第二套主帳。

## ERP 來源欄位

從 Legacy ERP 同步進 Cloud 的正式欄位：

- 預設唯讀
- 顯示 Source = ERP
- 每日同步更新
- 保留每日 Snapshot
- 不允許直接改掉 ERP 原始值

## Cloud 可編輯資料

Cloud 正常允許新增 / 修改 / 刪除：

- Notes
- Attachments
- Tags
- Mapping
- AI remarks
- Compare results
- External Excel data
- Prod / Account Flow supplements
- Cloud-only records

若未來真的需要改 ERP 正式值：

必須回原 ERP 正常作業，或另取得 Paul 核可後處理。

---

# 2. 系統名稱

使用者看到的系統名稱：

## Web Portal

**VRT ERP**

技術專案命名統一使用：

## Technical Namespace

**PSJERP**

## Legacy Sync Agent

**PSJERP Bridge**

## Central Cloud Database

**PSJERP DataHub**

## Telegram / Notification Service

**PSJERP Notify**

## Git Repository

```text
psjerp
psjerp-bridge
```

如需第三個 repository：

```text
psjerp-ops
```

## Telegram Bot

顯示名稱：

```text
VRT ERP
```

Bot Username 首選：

```text
@PSJERP_Bot
```

若已被使用：

```text
@PSJERP_VRT_Bot
@PSJ_FactoryERP_Bot
```

## Google Apps Script

Project：

```text
PSJERP Telegram
```

主檔：

```text
PSJERP.gs
```

用途為：

- deployment scripts
- database migrations
- maintenance tools
- scheduled jobs

---

# 3. 技術架構

## 3.1 Frontend

延續現有 VRT Prod 的操作邏輯，但設計得：

- 更乾淨
- 更現代
- 更直覺
- 少橫向捲動
- 少傳統 ERP 密密麻麻欄位
- 大部分功能 1–3 次點擊可到

建議：

- HTML / CSS / JavaScript
- 可用 React / Vue，但不強制
- Mobile responsive
- Desktop sidebar
- Mobile compact navigation

---

## 3.2 Hosting

第一階段沿用 VRT 現有做法，避免新增訂閱成本：

- **GitHub Pages**：前端 HTML / JS
- **Google Apps Script Web App / API**：後端資料讀寫與 Telegram
- **Google Sheets**：雲端資料表
- **Google Drive**：Excel / CSV / PDF / 圖片 / 附件

先不要導入 Supabase、Cloudflare Paid Service 或其他新增月費平台。

---

## 3.3 Primary Data Store — Phase 1 / 2

因 VRT 已使用 Google 並已有相關訂閱，第一階段正式採：

**Google Sheets + Google Apps Script + Google Drive**

目的：

- 不增加固定月費
- 與現有 Prod / Account Flow 技術棧一致
- Telegram 整合簡單
- 維護人員容易接手
- 每日只同步一次 Legacy ERP，沒有高頻即時資料庫需求

### 不可把所有 ERP 歷史資料塞進一張 Sheet

依功能拆分 Spreadsheet / Sheet，例如：

```text
PSJERP_MASTER
PSJERP_ORDERS
PSJERP_BOM
PSJERP_PURCHASE
PSJERP_RECEIVING
PSJERP_FABRIC
PSJERP_ACCESSORIES
PSJERP_FG
PSJERP_SHIPPING
PSJERP_COST
PSJERP_SYNC_LOG
PSJERP_AUDIT
```

歷史量大的資料再依年份拆：

```text
ORDERS_2024
ORDERS_2025
ORDERS_2026

BOM_2024
BOM_2025
BOM_2026
```

前端不要直接掃整本 Sheet；Apps Script API 需：

- 依期間查詢
- 依 key 查詢
- Cache
- 分頁
- Summary table
- Daily Snapshot

這樣才能維持速度。

### 未來升級條件

只有出現以下情況才重新評估 PostgreSQL / Supabase：

- Google Sheets 已明顯變慢
- Apps Script 執行時間 / quota 成為瓶頸
- 需要大量多人同時寫入
- 需要複雜交易鎖定
- 資料量遠超 Sheets 適合範圍
- 權限需求變得非常複雜

在那之前不要為「看起來比較像正式 ERP」而新增月費。

---

## 3.4 File Storage

正式使用：

- **Google Drive**

存放：

- Excel
- CSV
- PDF
- Style Image
- BOM attachment
- Packing
- Shipment
- Invoice
- Supporting document
- Photo

---

# 4. Authentication / Login

系統需有登入保護。

## 4.1 使用者需求

第一次輸入：

```text
User
Password
```

之後可選：

```text
Trust this device
Remember me
```

Paul 的固定電腦 / 手機登入一次後，後續可直接進系統。

不要每次都要求重新輸入密碼。

---

## 4.2 安全規則

不可：

- 把 password 寫在 HTML
- 把 password 寫在 JS
- 把 token 寫在 Git
- 把 Supabase Service Role Key 放前端
- 把 Telegram Bot Token 放 Git
- 把 Legacy ERP SQL Password 放 Git

Phase 1 不使用 Supabase Auth。

優先採 VRT 現有 Google / Apps Script 架構可支援的登入方式：

- Apps Script backend 驗證
- User / Password hash 不寫在前端
- Secure session token
- Trusted Device token
- Device revoke
- Password reset
- Session revoke

Paul 固定裝置登入一次後可保留 Trusted Device。

若未來改用其他 Backend，再抽換 Authentication Adapter，不要讓 UI 綁死特定供應商。

---

# 5. 權限

V1 可以先簡化成：

- Admin
- Management
- PC
- Purchase
- Warehouse
- Production
- Shipping
- Accounting
- Viewer

注意：

**這是 VRT ERP Cloud 的權限。**

不是 Legacy ERP 權限。

Legacy ERP 對 VRT Bridge 一律：

**READ ONLY。**

---

# 6. 多語言

必須完整支援：

- 中文
- English
- ខ្មែរ Khmer

介面上使用：

```text
中 | EN | ខ្មែរ
```

所有：

- Button
- Menu
- Summary
- Status
- Error
- Telegram Summary
- User Message

都需支援三語。

---

# 7. 系統主架構

VRT ERP 必須同時提供：

## A. ERP 總平台

例如：

```text
/erp
/dashboard
```

## B. 每個模組可單獨開啟

例如：

```text
/orders
/bom
/purchase
/receiving
/fabric
/accessories
/wip
/finished-goods
/factory-cost
/shipping
/costing
/sync
/compare
/reports
```

這點非常重要。

不要做成所有人都必須先進總 ERP 再找功能。

---

# 8. 主模組

第一版核心模組：

1. Dashboard
2. Orders / PO
3. BOM
4. Purchase
5. Receiving
6. Fabric
7. Accessories
8. WIP
9. Finished Goods
10. Production
11. Factory Cost
12. Shipping
13. Costing
14. Reports
15. Smart Import
16. Sync Center
17. Compare Center
18. Settings

---

# 9. UI / UX 設計

延續 VRT Prod 的使用習慣。

每頁標準結構：

```text
Title
↓
KPI Cards
↓
D / W / M / Y / Custom
↓
Search / Filter
↓
Compact Table / Cards
↓
Detail Drawer / Detail Page
```

避免：

- 大量左右捲動
- 199 個功能全部塞在 sidebar
- 傳統 ERP 深層選單
- 大量只有代碼沒有說明

---

# 10. Dashboard

總覽至少包括：

- Open Orders
- Due Soon
- Late Orders
- Purchase Outstanding
- Fabric Stock
- Accessory Stock
- Production WIP
- Finished Goods
- Shipment Due
- Shipment Completed
- Material Cost
- Factory Cost
- Estimated Cost
- Margin
- Sync Status
- Data Exception

支援：

```text
D
W
M
Y
Custom
```

---

# 11. Legacy ERP 資料來源

依 VRT ERP V3 手冊，已知資料來源如下。

## 11.1 Orders

來源：

```text
W_COD700
```

包含：

- od_no
- my_no
- Customer
- Style
- Color
- PO
- Order Date
- Delivery
- Qty
- Price
- Currency
- Closed Status

---

## 11.2 Order Progress

來源：

```text
W_COD690
```

主要：

- Order Qty
- Cutting Qty
- Finished Goods / Stock-in Qty
- Shipment Qty

---

## 11.3 BOM

來源：

```text
W_COD550
```

內容：

- Main Material
- Trims
- Color
- Width
- Weight
- Consumption
- Total Usage
- Supplier
- Unit
- Position
- PO / Destination

---

## 11.4 SMV

來源：

```text
W_CBA190
→ 成本基數 / Cost Base
```

類型：

```text
001 SAMPLE
002 CMT
003 PRODUCTION
004 SECOND
```

量產主要使用：

```text
003 PRODUCTION
```

現有已抄錄：

```text
SMV_master_1845.xlsx
```

---

## 11.5 Purchase Price

來源：

```text
W_CPU795
```

內容：

- Purchase No
- Supplier
- Date
- Material
- Color
- Qty
- Unit
- Currency
- Unit Price
- Delivery Date
- Received Qty

---

## 11.6 Main Material Purchase

來源：

```text
W_CPU350
```

---

## 11.7 Accessories Purchase

來源：

```text
W_CPU340
```

---

## 11.8 Receiving / Fabric Inspection

來源：

```text
W_CVA300
```

及其他安全的查詢 / report。

---

## 11.9 Fabric Inventory

來源：

```text
W_CSTA10V
W_CST541
```

---

## 11.10 Accessories Inventory

來源：

```text
W_CSTA20V
W_CST551
```

---

## 11.11 Finished Goods

來源：

```text
W_CST580
W_CST302
W_COD690
```

注意：

Legacy ERP 有成品入庫資料。

但目前 Prod / Account Flow 的 Finished Goods 不代表一定是 Live Legacy ERP。

需要做跨平台比對。

---

## 11.12 Shipping

來源：

```text
W_CBL300
W_CBL510
W_CBL520
```

---

## 11.13 Estimated Cost

來源：

```text
W_CCS160
```

---

## 11.14 Actual Cost

Legacy ERP：

```text
W_CCS510
```

目前實際成本資料空白。

**禁止在 Legacy ERP 執行重新計算。**

新的 VRT ERP 自行計算成本。

---

# 12. 現有歷史資料

目前已知：

```text
Orders:
45,932 rows
36,128 orders
2014–2026

Purchase Price:
82,291 rows

Shipment:
50,811 rows

SMV:
1,845 styles

BOM:
78,617 rows
約 9,400 orders
目前約完成 26%
仍持續補齊
```

這些先作為 VRT ERP Seed Data。

---

# 13. Data Keys

必須保留 Legacy ERP 的真實 Key。

核心：

```text
od_no
od_seq
my_no
style_id
clr_no
po_no
cust_id
bl_no
purchase_no
purchase_seq
material_no
```

關係：

```text
Order Header
  od_no
      │
      └── Order Line
            od_seq
            style_id
            clr_no
            po_no
```

---

# 14. Source Metadata

每一筆同步或匯入資料至少保存：

```text
source_system
source_record_id
source_key
source_file
source_sheet
source_updated_at
import_batch_id
synced_at
checksum
```

Source System 範例：

```text
LEGACY_ERP
VRT_PROD
ACCOUNT_FLOW
EXCEL_IMPORT
GOOGLE_SHEET
VRT_ONEERP
```

---

# 15. Cloud Data Schema / Google Sheets Logical Tables

至少建立：

```text
users
roles

customers
styles
style_smv
materials
suppliers

orders
order_lines

bom_headers
bom_lines

purchase_orders
purchase_lines

receipts
fabric_inspection

inventory_movements
inventory_balance

production_status
wip_snapshots

finished_goods
finished_goods_movements

shipments
shipment_lines

factory_cost
costing

attachments

import_batches
import_log

sync_runs
sync_errors

compare_results

telegram_log

audit_log
```

---

# 16. Legacy ERP Sync 架構

優先方式：

## A. SQL Server READ ONLY

建立專用：

```text
VRT Bridge Read-Only Account
```

只能：

```sql
SELECT
```

不能：

```sql
INSERT
UPDATE
DELETE
```

不能執行會寫資料的 procedure。

---

## B. GUI / clothes.exe Fallback

如果：

- SQL Table 尚未找到
- 欄位尚未 mapping
- 某些特殊 report 只能 GUI 產出

才使用 Claude 操作 `clothes.exe`。

用途：

- 找欄位
- 驗證資料
- Export
- 報表下載
- 特殊查詢

不能做：

- Save
- Edit
- Delete
- Recalculate
- Close
- Transfer
- Adjust

---

# 17. Sync Frequency — 改為每日一次

Legacy ERP 是主要正式系統（System of Record）。

PSJERP Cloud 的任務不是取代 Legacy ERP 即時作業，而是：

- 每天把 ERP 最新資料同步上雲端
- 提供手機 / PC 查詢
- 與 Prod / Account Flow / Excel 做比對
- 保留歷史快照
- 產生 AI 分析
- 發 Telegram 摘要

因此不需要 1–5 分鐘輪詢。

## 正常排程

```text
每日一次，下午執行
預設時間：15:30
時間必須可在 Settings 修改
```

## 工作日規則

自動同步只在 VRT 工作日執行：

```text
Monday–Saturday
15:30
```

以下不執行自動同步：

- Sunday
- VRT Factory Holiday Calendar 內的節假日 / 廠休日
- 其他由 Settings 標記為 Factory Closed 的日期

不要硬編碼柬埔寨國定假日清單作唯一依據；應以 **VRT Factory Calendar** 為準，因工廠實際休假可能調整。

若當天為休假日：

```text
SKIPPED — FACTORY HOLIDAY
```

寫入 Sync Log，但不要進 Legacy ERP、不要產生「資料沒更新」異常，也不要發一般每日 ERP 摘要。

如 Paul 在休假日手動按：

```text
Sync Now
```

則可以執行一次唯讀同步。

建議流程：

```text
15:30
Check VRT Factory Calendar
↓
Working Day? YES
↓
Legacy ERP 唯讀擷取
↓
VRT/PSJERP Bridge
↓
Validate / Compare
↓
Upload Cloud
↓
建立當日 Snapshot
↓
產生 Change Summary
↓
Telegram 發送摘要
```

若某天有急件，可提供：

```text
Sync Now / 立即同步
```

但仍然只能 READ Legacy ERP。

## 同步原則

- ERP 為正式主資料來源。
- ERP 來源資料在 Cloud 預設唯讀。
- 下一次 ERP 同步以 ERP 最新值更新 Cloud Mirror。
- Cloud 可新增附件、備註、mapping、標籤、比對結果、外部資料，但不得把修改回寫 Legacy ERP。
- 每次同步保留 Snapshot Date，供月 / 年趨勢及歷史比較。
- 同步完成後才發 Telegram 摘要，避免群組看到半套資料。

---

# 18. Sync Flow

Legacy ERP = **System of Record / 正式主系統**。

Cloud = **Mirror + Analysis + Compare + Distribution Layer**。

```text
Legacy ERP
↓
READ ONLY
↓
PSJERP Bridge
↓
Staging
↓
Validate
↓
Normalize
↓
Deduplicate
↓
Map Keys
↓
Compare
↓
Upsert VRT DataHub
↓
Sync Log
↓
Refresh VRT ERP
```

---

# 19. 絕對禁止 Reverse Sync

VRT ERP 的修改：

**第一階段絕不回寫 Legacy ERP。**

流程：

```text
Legacy ERP
    ↓
VRT ERP
```

不是：

```text
Legacy ERP
    ↔
VRT ERP
```

未來 Paul 另外決定是否停用 Legacy ERP 後，再重新評估。

---

# 20. Sync Center

建立獨立：

## VRT Sync Center

顯示：

```text
Dataset
Legacy Count
Cloud Count
Last Sync
Added
Updated
Unchanged
Errors
Duration
Status
```

資料集：

- Orders
- BOM
- SMV
- Purchase
- Receiving
- Fabric
- Accessories
- WIP
- Finished Goods
- Shipping
- Cost

狀態：

```text
Synced
Syncing
Partial
Offline
Error
```

按鈕：

```text
Sync Now
View Changes
View Error
Retry
History
```

---

# 21. Smart Import Center

必須做成核心功能。

使用者不用先選檔案種類。

直接：

```text
Upload Excel / CSV
```

系統自動判斷：

- filename
- sheet name
- header
- data pattern
- key
- date
- customer
- style
- PO
- material
- quantity

支援偵測：

- Orders
- PO Detail
- BOM
- Purchase
- Receiving
- Fabric Stock
- Accessories Stock
- WIP
- Finished Goods
- Factory Cost
- Shipment
- SMV

---

# 22. Smart Import 流程

```text
Upload
↓
Detect Dataset
↓
Parse
↓
Validate
↓
Map Fields
↓
Map Keys
↓
Compare Existing
↓
Preview
↓
New
Updated
Duplicate
Error
↓
Confirm Import
↓
Save
↓
Import Log
```

---

# 23. Smart Import 預覽

例如：

```text
Detected Dataset:
BOM

Confidence:
98.6%

Source:
Legacy ERP W_COD550

Orders:
627

Material Lines:
4,826

New:
3,920

Update:
650

Duplicate:
240

Error:
16
```

---

# 24. Import 行為

支援：

- Create
- Merge
- Update
- Replace selected record
- Skip Duplicate

禁止：

- 未預覽就整庫覆蓋
- 無紀錄 overwrite
- 自動刪除舊資料

---

# 25. Compare Center

建立：

## VRT Compare Hub

比較來源：

```text
Legacy ERP
VRT Prod
Account Flow
Excel Import
Google Sheet
VRT ERP
```

支援 Key：

- od_no
- my_no
- PO
- Customer
- Style
- Color
- Material
- Date

---

# 26. Compare Status

顯示：

```text
Matched
Missing
Qty Difference
Value Difference
Date Difference
Unmapped
Duplicate
```

---

# 27. Finished Goods 比對

Finished Goods 必須同時支援：

```text
Legacy ERP FG
Account Flow smart_fg
VRT Prod FG
VRT ERP FG
```

顯示：

```text
ERP Qty
ACC Qty
PROD Qty
Cloud Qty
Difference
```

Drill-down：

- Order
- Self Key
- PO
- Style
- Color
- Date
- Qty

---

# 28. WIP 設計

Accounting WIP 與 Production WIP 不能混成同一數字。

分成：

## Production WIP Qty

件數

## ERP Derived WIP Qty

從 Legacy ERP 相關資料推算

## Accounting WIP Value

金額

---

# 29. ERP Derived WIP

如資料足夠，可用：

```text
Order Qty
Cutting Qty
Material Issue
FG Qty
Shipment Qty
```

建立 WIP View。

但所有邏輯必須：

- 可追溯
- 可顯示公式
- 可顯示來源
- 不修改 Legacy ERP

---

# 30. Factory Cost

現有 Account Flow Factory Cost 保留為獨立來源。

VRT ERP Factory Cost 要能整合：

- Payroll
- OT
- Electricity
- Water
- Repair
- Spare Parts
- Consumables
- Depreciation
- Other Expense
- Production Qty
- SMV
- Labor Hours
- Material Cost
- FG
- Shipment

---

# 31. Factory Cost KPI

計算：

```text
Factory Cost / Piece
Factory Cost / SMV Minute
Factory Cost / Customer
Factory Cost / Style
Factory Cost / Order
Factory Cost / Month
Factory Cost / Year
```

---

# 32. BOM Module

BOM 頁支援：

- Order
- Style
- Customer
- Main Fabric
- Accessories
- Consumption
- Loss %
- Supplier
- Purchase Price
- Estimated Material Cost
- Difference
- Exception

---

# 33. BOM Analysis

參考：

```text
BOM_analysis_v1.xlsx
```

至少建立：

- Summary
- Year
- Main Material Top
- Trim Top
- Supplier
- Customer
- Style Consumption
- Order Material Cost
- Exception

---

# 34. BOM Cost Logic

基本：

```text
BOM Consumption
× Order Qty
× Purchase Price
= Estimated Material Cost
```

如果有：

- Latest Price
- Historical Price
- Weighted Average

必須清楚標示採用哪一種。

---

# 35. Costing Engine

VRT ERP Costing：

```text
Material Cost
+ Labor Cost
+ Factory Overhead
+ Freight
+ Duty
+ Other
= Estimated Total Cost
```

對比：

```text
FOB
DDP
Selling Price
```

計算：

```text
Margin $
Margin %
```

---

# 36. Estimated vs Actual

任何成本頁面必須分清：

```text
Estimated Cost
Actual Cost
Imported Cost
ERP Reference Cost
```

不可混在一起。

---

# 37. Purchase Module

顯示：

- Purchase No
- Supplier
- Material
- Color
- Qty
- Unit
- Unit Price
- Currency
- Purchase Date
- ETA
- Received
- Outstanding
- Order Link
- PO Link
- Status

---

# 38. Fabric Module

顯示：

- Material No
- Fabric Name
- Color
- Lot
- Width
- Weight
- Roll
- Unit
- Receiving
- Issue
- Balance
- Allocated
- Available
- Order
- PO
- Location

---

# 39. Accessories Module

顯示：

- Item
- Material No
- Supplier
- Unit
- Receiving
- Issue
- Balance
- Order
- PO
- Location

---

# 40. Shipping Module

顯示：

- Shipment No
- Customer
- Order
- PO
- Style
- Color
- Qty
- Shipment Date
- Invoice
- Container
- Booking
- ETD
- ETA
- Status

---

# 41. Production Integration

不要重寫現有 VRT Prod。

採 Adapter / API / Data Link。

OneERP Production 頁只負責：

- 顯示
- 串接
- 比對
- KPI
- Order linkage
- Cost linkage

現有 Prod 保持獨立運作。

---

# 42. Prod Integration

至少接：

- Production Plan
- Cutting
- Sewing
- WIP
- QC
- FG
- IE / SMV
- Shipment related production status

---

# 43. Account Flow Integration

至少接：

- WIP Value
- Finished Goods
- Factory Cost
- related accounting source

Account Flow 保留獨立使用。

---

# 44. Data Source Badge

所有關鍵資料最好顯示來源：

```text
ERP
PROD
ACC
IMPORT
GS
CLOUD
```

例如：

```text
ERP FG 18,520
ACC FG 18,498
PROD FG 18,520
Difference -22
```

---

# 45. Audit Log

VRT ERP 所有 Cloud 操作都記錄：

```text
time
user
module
record_id
action
before
after
device
ip
source
```

Action：

```text
CREATE
EDIT
DELETE
RESTORE
IMPORT
UPLOAD
MERGE
EXPORT
```

---

# 46. Delete

前端：

```text
Delete
→ Confirm
→ Deleted
```

不需要 Approval。

Backend 建議預設：

**Soft Delete**

例如：

```text
deleted_at
deleted_by
```

保留 30–90 天。

---

# 47. Restore

Audit / Trash 可：

```text
Restore
```

是否提供永久 Delete 可由 Admin 設定。

---

# 48. Telegram Integration

ERP 必須建立獨立 Telegram Bot。

不要共用 Prod Bot Token。

建議：

```text
Bot:
PSJERP_Bot

GS:
VRT_ERP.gs

Git:
psjerp
```

---

# 49. Telegram 與現有群組

允許：

**Production Bot + ERP Bot 同時存在同一個 Telegram Group。**

兩個 Bot：

- 不同 Bot Token
- 不同 Code
- 不同 Log
- 不同 Config
- 可同一 Group

---

# 50. Telegram Group Routing

預設：

## VRT Production Group

- Orders
- BOM
- Purchase
- Fabric
- Accessories
- WIP
- FG
- Production
- Shipping

## Account Flow Group

- Factory Cost
- Costing
- Margin
- AP
- AR
- Financial Summary

Paul 可在 Settings 修改。

---

# 51. Telegram ERP Menu

介面方式參考目前 VRT Prod / Account Flow。

例如：

```text
🏭 VRT ERP

📊 總覽             🧠 智慧匯入

🧾 訂單 / PO        🧵 BOM

🛒 採購             📥 進貨

🧶 布料             🧷 輔料

🧶 WIP              📦 成品入庫

💰 製造費用         🚢 出貨

🧮 Costing          🔄 同步中心

🔍 資料比對         📑 報告
```

每一個 Button 直接開對應 OneERP URL。

---

# 52. Telegram Summary

每個模組都有：

```text
Send Summary
```

支援：

- 中文
- English
- Khmer

支援：

- Management
- Production
- Warehouse
- Shipping
- Account Flow

---

# 53. Telegram Retry / Queue

必須做：

- Queue
- Retry
- Sent Log
- Error Log
- Duplicate Prevention

---

# 54. 報表

所有主要 Module 都支援：

```text
D
W
M
Y
Custom
```

Export：

- Excel
- CSV
- PDF（可後做）

---

# 54A. D / W / M / Y 期間導覽必須重做

目前只顯示 D / W / M / Y 按鈕是不夠的。

所有有時間維度的頁面必須採用與 VRT Sewing 類似的「期間導覽器」。

## 基本結構

```text
[D] [W] [M] [Y] [Custom]

◀    [期間下拉選擇]    ▶

[明細] [趨勢] [比較]
```

## 日 D

```text
◀  2026-10-04  ▶
```

可點日期開 Calendar。

## 週 W

```text
◀  2026-W40  09/28–10/04  ▶
```

支援 Year + Week 下拉。

## 月 M

必須有：

```text
◀   [2026 ▼] [10月 ▼]   ▶
```

不要只顯示「M」。

使用者可：

- 前一月
- 下一月
- 下拉選年份
- 下拉選月份
- 直接跳任意歷史月份

## 年 Y

必須有：

```text
◀   [2026 ▼]   ▶
```

使用者可直接切：

```text
2024
2025
2026
2027
...
```

## 月 / 年列表

像 Sewing 一樣提供列表。

### 月列表範例

```text
Month    Order     Cut     Sew     FG      Ship     Cost
2026-06
2026-07
2026-08
2026-09
2026-10
```

點某月直接進該月明細。

### 年列表範例

```text
Year     Order     Purchase     FG     Shipment     Revenue     Cost
2024
2025
2026
```

點某年直接查看該年資料。

## 比較模式

按：

```text
比較 / Compare
```

後可選：

### 月

- 本月 vs 上月
- 本月 vs 去年同月
- 自選月份 vs 自選月份

### 年

- 今年 vs 去年
- 自選年份 vs 自選年份

顯示：

```text
Current
Previous
Difference
Difference %
```

圖表也要同時畫兩個期間。

## Flow Metrics 與 Snapshot Metrics 不可混算

這很重要。

### Flow / 期間累計資料

例如：

- Orders
- Purchase
- Receiving
- Production Output
- Shipment
- Factory Cost

月 / 年查看時用：

```text
SUM during selected period
```

### Snapshot / 時點資料

例如：

- Fabric Stock
- Accessories Stock
- WIP Balance
- FG Balance

月 / 年比較時不可把每天庫存加總。

應使用：

```text
Month End Snapshot
Year End Snapshot
```

或指定日期最近一次 Snapshot。

畫面需清楚標示：

```text
As of 2026-09-30
```

## 必須套用的頁面

至少：

- Dashboard
- Orders
- Purchase
- Receiving
- Fabric
- Accessories
- Production
- WIP
- Finished Goods
- Shipping
- Factory Cost
- Costing
- Reports

使用同一個 Period Navigator component，不要每頁自己做一套。

---

# 55. Global Search

頂部建立 Global Search。

可搜尋：

- ERP Order
- Self Key
- PO
- Customer
- Style
- Material
- Purchase No
- Shipment
- Invoice

---

# 56. Detail Drawer

點一筆 Order 後，不必離開整頁。

右側 Drawer 顯示：

```text
Order Header
PO
Style
Color
Qty
SMV
BOM
Purchase
Receiving
Stock
Production
FG
Shipment
Cost
Attachment
History
```

---

# 57. Exception Center

建立資料異常中心。

例如：

- Order Price = 0
- Qty = 0
- Old Order not closed
- Missing BOM
- Missing SMV
- SMV = 0.01 placeholder
- Purchase no link
- Stock difference
- FG difference
- Shipment difference
- Duplicate
- Unmapped Material

---

# 58. Existing Known Data Issues

Legacy ERP 現已知：

- 舊單未結案
- 單價 0
- 數量 0
- 部分異常高單價
- BOM 有舊單缺失
- 多數採購單價沒有自編單號
- BOM Export 缺料號
- W_CCS510 實際成本目前空白
- 部分款式 SMV 缺失 / 0.01 placeholder

新 ERP 不得自動「修正」Legacy ERP。

只需：

- Flag
- Compare
- Report
- Clean in Cloud mapping if approved
- Preserve original source value

---

# 59. Security

Phase 1 使用：

- Google Apps Script backend authorization
- Server-side user/role validation
- Server-side secrets / Script Properties
- HTTPS
- Google Drive / Sheet 權限
- API session validation

不要只靠前端按鈕隱藏權限。

所有真正的新增 / 修改 / 刪除請求，都要由 Apps Script backend 再檢查：

```text
session
user
role
module permission
record permission
```

即使使用者自行修改前端 JavaScript，也不能越權寫入資料。

---

# 60. Secrets

以下一律放 Server Secret：

- Telegram Bot Token
- SQL Credentials
- Google API Secret
- SMTP Secret

禁止：

- Git
- HTML
- JS bundle
- downloadable config

---

# 61. Backup

PSJERP Google Data Store 需有：

- Google Sheets 每日版本 / Snapshot
- 每日或每週 CSV / XLSX Export Backup
- Google Drive 附件備份
- Restore Test
- 保留最近 N 期備份（可在 Settings 設定）

重要主表在 Apps Script 寫入前可建立 lightweight backup / revision record。

---

# 62. Logging

系統至少有：

- application log
- sync log
- import log
- telegram log
- audit log
- error log

---

# 63. Phase 1

先做 Prototype。

不要先連 Live Legacy ERP。

使用現有匯出的：

- Orders
- Purchase Price
- Shipping
- SMV
- BOM

建立：

1. Login
2. Dashboard
3. Orders
4. BOM
5. SMV
6. Purchase
7. Shipping
8. Costing
9. Smart Import
10. Sync Center
11. Compare Center

---

# 64. Phase 2

增加：

- Receiving
- Fabric
- Accessories
- WIP
- Finished Goods
- Factory Cost
- Prod integration
- Account Flow integration
- Telegram
- Attachment
- Audit UI
- Exception Center

---

# 65. Phase 3

建立：

## PSJERP Bridge Daily Sync

優先：

SQL Server Read-only。

如果 schema 尚未確認：

使用 V3 手冊的 safe GUI / export 流程做 mapping。

---

# 66. Phase 4

建立：

- automatic reconciliation
- cost trend
- margin trend
- customer profitability
- style profitability
- supplier price trend
- material usage variance
- stock variance
- WIP variance
- shipment performance

---

# 67. 不要做的事

不要：

- 複製 Legacy ERP 199 個功能做成 199 個 Menu
- 把 Google Sheet 當正式 ERP DB
- 把 Bot Token 放 Git
- 把 SQL Password 放 JS
- 把所有資料塞成一張表
- 把 Accounting WIP 和 Production WIP 混成一個數字
- 把 Estimated Cost 當 Actual Cost
- 自動回寫 Legacy ERP
- 自動清除舊 ERP 異常資料
- 沒 Preview 就 Import
- Import 時無紀錄 overwrite
- 共用 Prod Bot Token

---

# 68. 開發前先交付

開始大規模 coding 前，Claude 必須先提供：

1. Folder Structure
2. Database Schema
3. ER Diagram
4. Module Map
5. Page Map
6. Legacy → Cloud Mapping
7. Data Key Map
8. UI Wireframe
9. Sync Architecture
10. Import Architecture
11. Telegram Architecture
12. Security Design
13. Deployment Design

Paul 看過後再進 Phase 1。

---

# 69. 建議 Folder Structure

可參考：

```text
psjerp/
│
├─ app/
│  ├─ dashboard/
│  ├─ orders/
│  ├─ bom/
│  ├─ purchase/
│  ├─ receiving/
│  ├─ fabric/
│  ├─ accessories/
│  ├─ wip/
│  ├─ finished-goods/
│  ├─ factory-cost/
│  ├─ shipping/
│  ├─ costing/
│  ├─ sync/
│  ├─ compare/
│  ├─ reports/
│  └─ settings/
│
├─ components/
├─ adapters/
│  ├─ legacy-erp/
│  ├─ vrt-prod/
│  ├─ account-flow/
│  ├─ google-sheet/
│  └─ excel-import/
│
├─ services/
│  ├─ auth/
│  ├─ import/
│  ├─ sync/
│  ├─ compare/
│  ├─ costing/
│  ├─ telegram/
│  └─ audit/
│
├─ gas/
│  ├─ PSJERP.gs
│  ├─ api/
│  ├─ auth/
│  ├─ sheets/
│  ├─ telegram/
│  └─ config/
│
├─ data-schema/
│  ├─ sheets.md
│  ├─ keys.md
│  └─ seed/
│
├─ public/
├─ docs/
└─ tests/
```

---

# 70. Acceptance Criteria

Phase 1 至少要通過：

## Login

- 第一次登入正常
- Trusted Device 正常
- 登出正常
- Session revoke 正常

## Orders

- 搜尋 ERP Order
- 搜尋 Self Key
- 搜尋 PO
- Customer / Style filter
- Order detail 正確

## BOM

- 能看到 Main / Trim
- Consumption
- Loss
- Price
- Estimated Material Cost
- Source

## SMV

- style_id 可查
- Production SMV 可顯示
- Missing / Placeholder 有 Flag

## Purchase

- Supplier
- Price
- Currency
- Qty
- Date
- ETA
- Received
- Outstanding

## Shipment

- Customer
- Order
- PO
- Style
- Qty
- Date
- Invoice

## Smart Import

- Upload
- Detect
- Preview
- New / Update / Duplicate / Error
- Confirm
- Import Log

## Compare

- 至少可比較兩來源
- 顯示 Difference
- Drill down

## Sync Center

- 顯示 dataset
- last sync
- status
- count
- error

## Audit

- Edit 有 Before / After
- Delete 有 Log
- Restore 可追蹤

---

# 71. 最後核心原則

VRT ERP 的定位不是：

「把舊 ERP 畫面搬到網頁。」

而是：

```text
Legacy ERP
+
VRT Prod
+
Account Flow
+
Warehouse Data
+
Excel
+
Google Sheets
+
Telegram
+
Cloud Database
```

整合成：

# VRT ERP

Legacy ERP 繼續作為重要歷史與正式來源之一，但保持唯讀。

VRT ERP 作為：

- Cloud Management Layer
- Data Hub
- Compare Hub
- Costing Hub
- Reporting Hub
- Integration Hub

---

# 72. Claude 執行時的工作順序

請依以下順序：

```text
1. 閱讀 VRT ERP 操作手冊 V3
2. 盤點目前可用的 Legacy ERP Export
3. 盤點現有 VRT Prod
4. 盤點現有 Account Flow
5. 建立 Data Source Map
6. 建立 Database Schema
7. 建立 Wireframe
8. 建立 Phase 1 Prototype
9. Seed Existing Data
10. 驗證 Order / BOM / SMV / Purchase / Shipping
11. 加 Smart Import
12. 加 Compare Center
13. 加 Sync Center
14. 再處理 Receiving / Inventory / WIP / FG / Factory Cost
15. 最後才做 Live VRT Bridge
```

---

# 73. 每次工作完成回報格式

Claude 每次完成任務後，用以下格式回報：

```text
完成：
-

新增：
-

修改：
-

資料來源：
-

有無碰 Legacy ERP：
-

Legacy ERP 是否完全唯讀：
YES / NO

Cloud 資料是否有新增修改：
-

已測試：
-

未完成：
-

發現問題：
-

下一步建議：
-
```

---

# 74. Safety Reminder

**任何 Legacy ERP 寫入動作都需要 Paul 明確核可。**

**VRT ERP Cloud 本身可正常新增、修改、刪除、匯入，不需要 Approval，但必須有 Audit Log。**

此規則優先於任何其他開發便利性。


---

# 75. 最終命名定案

使用者介面：

```text
VRT ERP
```

技術專案：

```text
PSJERP
```

Git：

```text
psjerp
psjerp-bridge
```

Telegram：

```text
Display Name: VRT ERP
Username: @PSJERP_Bot
```

Google Apps Script：

```text
Project: PSJERP Telegram
File: PSJERP.gs
```

Google Apps Script Project：

```text
PSJERP
```

Google Drive Root Folder：

```text
PSJERP
```

這樣員工看到的是 VRT ERP，技術層使用 PSJERP，簡短、好管理，也不需要把 Production Bot / Git 混在一起。


---

# 76. 成本控制原則 — 不新增固定訂閱

目前 PSJERP 的 Phase 1 / Phase 2 以 VRT 已在使用的服務為主：

```text
GitHub
Google Sheets
Google Apps Script
Google Drive
Telegram Bot
```

設計目標：

**先做到不新增固定月費。**

如果某項新雲端服務需要付費，Claude 不得直接導入。

必須先回報 Paul：

```text
服務：
每月費用：
為什麼需要：
目前 Google 架構做不到什麼：
是否有免費 / 已付費服務替代：
```

取得 Paul 明確同意後才可加入。

Supabase / PostgreSQL 可保留為未來升級選項，不是目前必要條件。


---

# V3 追加（2026-10-04）

## V3.1 Repository

- GitHub repo 改為 `saintdou-weng/vrt-asset`，整個 repo 只放 PSJERP，舊內容不保留。
- 網址：`https://saintdou-weng.github.io/vrt-asset/`
- `WEB_BASE_URL` 預設值同上；Telegram webhook 一律用 `/exec`（Script Property `WEBAPP_URL`）。

## V3.2 期間導覽器（已完成，v0.4）

- 所有有日期的頁面共用同一個 Period Navigator：總覽、訂單、BOM、採購、進貨、布料、輔料、生產、WIP、成品入庫、出貨、製造費用、成本、報告、比對中心、異常中心、同步紀錄、匯入紀錄、Telegram 紀錄、稽核紀錄。
- 月一定要有 ◀ ▶ + 年份下拉 + 月份下拉；週有年份 + 週次下拉；年有年份下拉；日有日曆。
- 明細 / 趨勢（月列表、年列表，點列進該期）/ 比較（上一期、去年同期、自選）。
- 本期未結束時，比較取相同天數。
- 製造費用為月資料，日 / 週期間按天數攤分並標示。

## V3.3 ERP 正式值在雲端唯讀

- 來源為 LEGACY_ERP / VRT_PROD / ACCOUNT_FLOW 的紀錄：正式欄位鎖定，只可加雲端備註與標籤，不可在雲端刪除。
- 雲端自建紀錄（VRT_ONEERP / EXCEL_IMPORT）可正常新增、修改、軟刪除。
- 智慧匯入遇到 ERP 正式值不同時：不覆蓋，寫入比對中心「匯入檔 vs ERP」。ACC / PROD / 實盤等補充欄位可更新。

## V3.4 真實資料模式 + ERP 報表匯入（v0.5，2026-10-06）

### 原則
- 預設「真實資料」：只顯示從 ERP 匯出檔匯入的數字。沒匯入的頁面顯示「還沒有真實資料」與需要的報表，不再顯示任何示範數字。
- 「示範資料」只在設定頁手動切換，畫面頂端有條紋標籤。
- 資料存在瀏覽器 IndexedDB（ERP 歷史量大，localStorage 放不下）。之後再接 Google Sheets 讓手機 / 電腦共用。

### 月報 / 年報
- 選「月」或「年」時，預設是列表（同 VRT Sewing 月年趨勢）：期間 | 指標欄 | 主指標藍字 | 百分比綠色小標，最上面是選取期間的卡片，中間長條圖，點列看該期明細。
- 明細（逐筆）只有在點列或切「明細」時才出現。
- 日 / 週 / 自訂維持「明細 / 趨勢 / 比較」。

### 已建的 ERP 報表解析器（依報表標題辨識，用實檔驗證過）
| 檔名慣例 | 報表標題 | ERP 功能 | 進到哪裡 | 重匯規則 |
|---|---|---|---|---|
| Q+年份（Q2014） | 訂單統計表(分PO) | W_COD690 | 訂單 / PO | 同年份取代 |
| R+年份（R2014） | 成品入庫資料報表 | W_CST580 | 成品入庫（含尺碼） | 同入庫日期區間取代 |
| K01 | 主料庫存明細表 | W_CSTA10V / W_CST541 | 布料（時點快照） | 同列表日期取代；不同日期保留為歷史快照 |
| K02（待實檔） | 副料庫存明細表 | W_CSTA20V / W_CST551 | 輔料（時點快照） | 同上 |
| B01, B02… | 訂單用料清單一覽表 | W_COD550 | BOM | 同自編單號區間取代 |

- 每個檔匯入前都會逐筆加總，與報表的「合計」列核對（有合計列的報表）。
- W_COD690 沒有接單日：只能進年報；匯入 W_COD700 或 BOM 後，以自編單號自動補上接單日、客戶、ERP 訂單號。
- 尚未建專用解析器（第一個實檔到了再建）：W_COD700、W_CPU795、W_CBL300、W_CBA190、W_CVA300、W_CST302、W_CPU350、W_CPU340、W_ONE102/103/104、W_CBA860。期間可用通用智慧匯入。

### 新報表加入流程
1. 下載後照慣例命名（字母 + 年份或批次）。
2. 第一份丟進智慧匯入：系統若顯示「不是已知 ERP 報表」，把這個檔給 Claude 建解析器。
3. 建好後同類檔案可一次拖多個匯入，並出現在「同步中心 → ERP 匯入清單」。

### 第一批實檔核對結果（2026-10-06）
| 檔 | 筆數 | 核對 | 發現 |
|---|---|---|---|
| Q2014 | 3,757 | 訂單量 4,984,749.2 / 入庫 2,341,801 / 出貨 2,457,399 全部 = 報表合計 | 裁剪量全為 0；343 列出貨 > 入庫；6 列訂單量 0 |
| R2014 | 3,215 | 2,660,118 件；每筆尺碼合計 = 數量 | 實際入庫日 2014-04-02 ～ 2014-12-28 |
| K01 | 427 | 743,451.08 = 報表合計（As of 2026-10-06） | 318 列疋數為負；1 列庫存為負；2 列單位 M |
| B01 | 150 張訂單 / 1,283 列 | 主料 170 列、副料 1,113 列 | 全為 AMERICAN DAWN；2 張沒有主料；沒有料號與單價 |

## V3.5 雲端同步 Google Sheets（v0.5.1，2026-10-06）

### 目的
電腦匯入的 ERP 資料，手機與其他電腦都看得到同一份。

### 架構
- 網頁仍存 IndexedDB（離線可看），另外與 Apps Script `/exec` 同步。
- Apps Script 建立試算表 `PSJERP_ERPDATA`（Script Property `ERP_SHEET_ID`）：
  - `ERP_FILES`：每個匯入批次一列（key、報表、檔名、筆數、範圍、As of、合計、核對、version、欄位、型別）。
  - 資料表：`ORDERS`(W_COD690)、`FG`(W_CST580)、`BOM`(W_COD550)、`FABRIC`(K01 主料)、`ACCESS`(K02 副料)。每列有 `import_key`。
  - `NOTES`：資料集 + 記錄 id + 備註 + 標籤。
- 文字欄設為純文字格式（`@`），日期與單號不會被 Google 自動轉換；欄位型別 n/s/z（數字 / 文字 / 全空）。
- API（POST 用 text/plain，body 帶 `api:'psjerp'` 與同步密碼 `k`；GET 帶 `action=erp_*&k=`）：`erp_manifest`、`erp_get`（分頁）、`erp_put`（分段 2,500 列，第一段先刪同 key，最後一段寫 ERP_FILES 與 version）、`erp_note`、`erp_drop`。
- 同步密碼：`SYNC_KEY_DIRECT` 或 Script Property `SYNC_KEY`；沒有密碼一律拒絕讀寫。網頁存在該裝置 localStorage `psjerp_synckey`。GitHub 版 .gs 留空。

### 同步規則
- 順序：先上傳本機未上傳批次（dirty）→ 讀 manifest → 下載雲端 version 不同的批次（剛上傳的跳過）→ 上傳排隊中的備註 → 套用雲端備註。
- 觸發：開網頁、切回網頁且距上次同步 > 3 分鐘、匯入確認後、切到真實資料模式、按「立即同步」。
- 重匯同一 import_key = 取代（雲端與本機都不會重複）。
- ERP 來源記錄鎖定，只同步備註 / 標籤。
- 右上角雲朵狀態：已同步 / 同步中 / 錯誤 / 未設定。

### 驗證（Apps Script 模擬器 + 兩個瀏覽器）
- A 匯入 4 檔上傳；B（手機寬度）下載：訂單 3,757、成品 3,215、BOM 1,283、布料 427；成品合計 2,660,118、布料 743,451.08，逐欄比對無差異。
- B 改備註 → A 收到；A 重匯 Q2014 → B 仍 3,757 列，不重複。

### 小修
- 月報 / 年報長條圖：主指標整段都是 0（例如 W_COD690 沒有接單月份）時，自動改畫有資料的指標。


## V3.6 追加（2026-10-07，v0.6.0）

### 已完成
- 全部已下載 ERP 報表都有專用解析器，辨識順序：標題 → 表頭欄名（txt / 無標題 xlsx）→ 範本。
- `handleFiles()` 多個未知檔被吃掉的 bug 已修：每個檔一列進「匯入佇列」，核對 ❌ 的預設不勾選。
- 全域拖放（任何頁面，含 `#solo-*`）；`needHTML` 顯示需要的報表、檔名、手冊 V4.1 SOP 編號與拖放區。
- 期間導覽：預設最近有資料期間（不超過今天）、下拉 ● 標示、沒資料提示＋一鍵跳轉；導覽狀態 key 改 `psjerp_nav6_`。
- 訂單：W_COD700 為主（接單日、單價、交期、結案日），W_COD690 補入庫／裁剪；沒有 W_COD700 的單仍顯示 W_COD690。出貨金額＝出貨量×訂單單價（W_COD700）。
- BOM 批次重疊：同一 od_no 以後匯入的批次為準（去重）。
- 匯出 Excel（說明＋資料，單號存文字）。
- 欄位對應範本：DB.tpl ↔ 雲端 TEMPLATES（`erp_tpl`、manifest 回傳 `tpls`）。
- 人事：不進 PSJERP；新增 `labor` 月彙總頁（部門／線別、人數、工時、直接／間接人工）。
- PSJERP.gs：`PSJERP_ERP_<資料集>` 一資料集一試算表、一批次一分頁（分頁名＝key），> 900 萬格自動開下一本；ERP_FILES 多 `sheet_id`、`tab`；舊 v0.5 格式仍可讀，重匯時自動搬。上傳不送空欄與 6 個來源欄（下載時由批次資料補回），省格數。

### 解析器（實檔驗證 2026-10-07）
| 報表 | 檔 | 驗證 |
|---|---|---|
| W_CPU350 主料採購 | PM2014–PM2026 | PM2024 475 列、進貨總金額 4,279,687.93 = 幣別合計；PM2014 多個幣別小計加總 = 3,875,940.78 |
| W_CPU340 副料採購 | PF2014–PF2026 | PF2024 2,900 列、金額 548,549.41 = 合計 |
| W_CST541／551 進耗存 | M／F2014–2026 | M2024 477 列、結存 839,615.33；F2024 5,604 列、結存 10,980,050.33；每列 期初＋入−出＋調 = 結存 |
| K02 副料庫存 | K02 | 7,120 列、7,858,270.14 = 合計 |
| W_CST302 成品庫存 | K04 | 1,310 列、483,623 件 = 合計 |
| W_CBL300 出貨 | S2014–S2026.txt | 全部 50,811 列、75,473,615 件 |
| W_CPU795 採購單價 | P2014–P2026.xlsx | 全部 82,291 列 |
| W_COD700 訂單 | Excel 匯出（工作表 自編單號／訂單日期／…） | 4,649 列測試檔；txt 版欄名（英文）已預留對應 |
| SMV | SMV_master_1845.csv | 1,845 款 |
| 全部一起 | 191 個檔 | 成品入庫 76,418,800 件、採購 82,387 列、BOM 35,031 張訂單，與 ERP 合計一致；雲端模擬（兩個瀏覽器）上下傳逐欄一致、重匯不重複 |

### 期間依據
採購（W_CPU340／350）= 採購單號年月（DA2401… = 2024-01），交貨日期另存 `eta`；進耗存＝年度（無月份）；庫存類＝As of 快照。
