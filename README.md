# Office Order System（辦公室訂飲料/便當系統）

本專案提供「飲料 + 便當」的辦公室訂購系統，前端部署在 GitHub Pages，後端使用 Google Apps Script Web App 串接 Google Sheets。

這份 README 目的：
- 讓第一次接觸的 AI/工程師清楚知道「已完成什麼、還剩什麼」
- 提供完整的執行步驟、資料表結構、API 與驗證流程
- 確保後續接手可無痛延續開發

---

## 目前完成狀態（概覽）

已完成：
- 前端骨架（Vue 3 + Vite + Tailwind + Hash Router + PWA manifest）
- 訂購頁 / 我的訂單 / 統計 / 管理者頁 UI（含 mock + API wrapper）
- GAS Web App 基礎架構 + action router
- 使用者端 API：getCurrentOrders / getProducts / submitOrder / updateOrder / cancelOrder / getMyOrders / getOrderSessions / getStatistics
- 管理者 API：adminLogin / uploadData / openOrder / closeOrder / getStores / toggleActive / exportOrders
- Sheets schema 文件 + idempotent 初始化腳本
- 部署與手動 QA 文件

尚待完成（需要實際部署與資料）：
- 端到端手動驗收（需 GAS Web App URL + Sheets 實際資料）
- 最終驗收證據（curl/log）

---

## 技術架構

前端（GitHub Pages）：
- Vue 3 + Vite + Tailwind CSS
- Hash Router（避免 GitHub Pages refresh 404）
- PWA 只做 manifest，不做 Service Worker

後端（Google Apps Script Web App）：
- doGet/doPost action router
- POST 使用 `application/x-www-form-urlencoded`
- 回應 JSON（ContentService）
- Script Properties 保存敏感資訊（AdminPassword / TokenSecret / SpreadsheetId）

資料庫：
- Google Sheets（Config / Stores / Products / CurrentOrder / Orders）

---

## 已定案的產品決策

- 使用者身分：只輸入姓名（localStorage）
- 訂單場次：手動開/關
- 同時開單：最多 2 個 open 場次（drink + meal 各一個）
- 飲料選項：全站預設 + 店家覆寫（不做品項覆寫）
  - Size：大杯 / 中杯 / 小杯
  - Sugar：正常 / 半糖 / 三分糖 / 一分糖 / 無糖（存中文）
  - Ice：正常冰 / 少冰 / 微冰 / 去冰 / 溫 / 熱
- 管理者：
  - 密碼在部署時設定一次（不寫進文件/前端）
  - adminToken 有效期 6 小時；存 sessionStorage；登出清除
  - 後續管理 API 一律用 POST body 傳 adminToken
- 上傳：CSV 管線分隔，JSON 陣列；Upsert（同 ID 更新）
- 通知：不做（Email/Line 都不做）
- 深色模式：不做

---

## 專案結構

```
.
├── backend/apps-script/      # GAS 原始碼（Web App）
│   ├── Code.gs               # doGet/doPost + router
│   ├── actions.gs            # 使用者端 API
│   ├── orderActions.gs       # 訂單操作、統計、匯出
│   ├── adminActions.gs       # 管理者登入/開關單/上傳/匯出
│   ├── dataStore.gs          # Sheets 存取與通用工具
│   └── sheetsInit.gs         # Sheets 初始化（可重跑）
├── docs/
│   ├── sheets-schema.md      # Sheets schema + 初始化說明
│   ├── manual-qa.md          # curl 手動驗證指令
│   └── deployment.md         # 部署流程
├── public/
│   ├── icon.svg
│   └── manifest.webmanifest
├── src/
│   ├── pages/                # Order/MyOrders/Stats/Admin
│   ├── services/api.js       # API wrapper
│   ├── styles/               # tokens.css + main.css
│   ├── App.vue
│   └── main.js
├── index.html
├── vite.config.js
├── tailwind.config.cjs
└── package.json
```

---

## 需要的套件與工具

前端依賴（package.json）：
- vue
- vue-router

前端 devDependencies：
- vite
- @vitejs/plugin-vue
- tailwindcss
- postcss
- autoprefixer
- typescript（目前未啟用 ts 專案，但保留）

後端（GAS）：
- 不需 npm 套件，直接在 Apps Script 環境執行

---

## 環境設定

前端 `.env.local`（根目錄新增）：

```
VITE_API_BASE_URL=https://script.google.com/macros/s/XXXX/exec
```

Apps Script Script Properties（後台設定）：
- SpreadsheetId
- AdminPassword
- TokenSecret

---

## 執行方式（前端）

```bash
npm install
npm run dev
```

建置與預覽：

```bash
npm run build
npm run preview
```

---

## Sheets 初始化

請參考 `docs/sheets-schema.md`：
- 可手動建立 tabs + headers
- 或把 `backend/apps-script/sheetsInit.gs` 加進 GAS 專案，執行 `initSpreadsheet('SPREADSHEET_ID')`

---

## 手動驗證（API）

請參考 `docs/manual-qa.md` 的 curl 指令。

亦可使用腳本：

```bash
source scripts/env.sample
bash scripts/manual-qa.sh
```

## 範例資料（可直接上傳）

位於 `docs/sample-data/`：

- `stores.csv` / `stores.json`
- `products.csv` / `products.json`

可用腳本快速上傳：

```bash
bash scripts/seed-data.sh
```

## 管理者輔助腳本

- `scripts/admin-login.sh`
- `scripts/open-sessions.sh`
- `scripts/close-session.sh`
- `scripts/export-orders.sh`

---

## 開發進度清單（計畫同步）

已完成：
- 1 前端骨架
- 2 GAS 骨架
- 3 Sheets schema 初始化
- 4 getCurrentOrders + getProducts
- 5 submitOrder（LockService）
- 6 訂購頁 UI + bottom sheet
- 7 update/cancel/myOrders/orderSessions
- 8 getStatistics + exportOrders
- 9 管理者後台（登入/上傳/開關單/登出）

待完成：
- 10 部署流程與最終驗收（需 GAS Web App URL + 實際資料）
- 11~13 最終驗收項目（API 實際呼叫）

---

## 接下來要做什麼（給下一個 AI/工程師）

1. 建立或綁定 Google Sheet，完成 schema 初始化（`docs/sheets-schema.md`）。
2. 在 Apps Script 設定 Script Properties（SpreadsheetId / AdminPassword / TokenSecret）。
3. 部署 GAS Web App（AnyOne 可存取），取得 URL。
4. 設定前端 `.env.local`（VITE_API_BASE_URL）。
5. 依 `docs/manual-qa.md` 跑 curl 驗證並補齊驗收證據。

---

## 重要限制（勿違反）

- 不使用自訂 headers（避免 CORS/preflight）
- 不做 Service Worker / 通知 / 深色模式 / 付款 / 帳號系統
- Token 只放 POST body，前端用 sessionStorage

---

## 參考文件

- `docs/sheets-schema.md`
- `docs/manual-qa.md`
- `docs/deployment.md`
- `.sisyphus/plans/office-order-system.md`
