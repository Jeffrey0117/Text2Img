## Text2Img

簡單的圖片加字工具 — 在 canvas 上為圖片疊加可拖曳的中文文字並一鍵複製，純前端、零建置。

## Stack
- Node.js 原生 `http` 模組 — 無框架的靜態檔案伺服器 (`server.js`)
- 前端：純 vanilla JS + HTML Canvas 2D，無打包工具、無相依套件
- 部署：PM2 (`ecosystem.config.js` / `.pm2-ecosystem.json`) + CloudPipe，Docker (`.dockerignore`)
- `package.json` 無 dependencies，僅一個 `start` script

## Directory structure
```
text2img/                ← symlink target: workhub/text2img
  server.js              ← 靜態檔案伺服器 (port 4022)，含 MIME map + 目錄穿越防護
  img.html               ← 主頁（server 預設 "/" → img.html）：圖片 + 可拖曳文字疊加
  index.html             ← 文字轉圖片頁面（canvas 渲染純文字成圖）
  ecosystem.config.js    ← PM2 設定（精簡版）
  .pm2-ecosystem.json    ← PM2 完整設定（絕對路徑、env、autorestart）
  README.md / SPEC.md    ← 功能說明與規格
  LINEBREAK_REPORT.md    ← 換行間距修復紀錄
```
注意：專案根目錄是 symlink → `C:\Users\jeffb\Desktop\code\workhub\text2img`。

## Key concepts
- **單檔應用**：每個 HTML 自帶全部 CSS + JS（inline `<script>`），開瀏覽器即用，server 只是選配。
- **server.js**：把 `/` 導向 `img.html`，依副檔名查 MIME 回傳，`startsWith(__dirname)` 防目錄穿越，404/403 fallback。Port 由 `process.env.PORT` 控制，預設 4022。
- **img.html 狀態模型**：`state.textItems = [{ id, x, y, text, fontSize, fontFamily, color, bgColor }]`，多個文字物件可在 canvas 上滑鼠拖曳（mousedown/move/up）、雙擊編輯；另含 `canvasBg`、`canvasPadding`。
- **圖片輸入**：點擊、拖放 (dragover/drop)、Ctrl+V 貼上三種方式。
- **輸出**：canvas → 一鍵複製到剪貼簿。
- **主題切換**：CSS 變數 (`--bg`/`--surface`/`--text` 等)，深色為預設、`:root.light` 為淺色，狀態存 `localStorage.theme`。
- **收藏字體**：星星 toggle，收藏清單存 `localStorage.favoriteFonts` (JSON 陣列)，收藏的字體排序置頂。
- 多種中文字型，可調字級、顏色、背景色、間距。

## Commands
```bash
npm start          # node server.js — 啟動靜態伺服器（預設 :4022）
# 或直接用瀏覽器開啟 img.html / index.html，無需 server

pm2 start ecosystem.config.js   # 用 PM2 常駐（autorestart, max_restarts:5）
```

## Coding rules
- 維持零相依、零建置：不要引入框架或打包工具，新功能直接寫進對應 HTML 的 inline JS/CSS。
- 持久化偏好一律走 `localStorage`（如 `theme`、`favoriteFonts`）。
- 主題色彩透過 CSS 變數，勿在元件硬寫顏色。
- 修改 server 時保留目錄穿越防護與 MIME 對應表。
