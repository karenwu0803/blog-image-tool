# CLAUDE.md

## Project Overview

部落格圖片處理工具 (Blog Image Tool) — 純前端單檔應用，用於部落格文章圖片的壓縮、縮放、裁切與 LOGO 疊加。

- **線上版**: https://karenwu0803.github.io/blog-image-tool/
- **GitHub**: https://github.com/karenwu0803/blog-image-tool

## Tech Stack

- 純 HTML/CSS/JS，零外部依賴
- 單一檔案 `index.html`（約 736 行，含 HTML + CSS + JS）
- Canvas API 處理圖片（縮放、裁切、銳化、壓縮、LOGO 疊加）
- localStorage 儲存 LOGO（base64）

## Architecture

所有程式碼包在一個 IIFE 中，無全域變數。核心結構：

- `state` — 可變狀態物件（srcImg, srcFile, logoImg, cropOff, outBlob, busy, pendingRender...）
- `dom` — DOM 元素快取，由 `cacheDom()` 初始化
- `bindEvents()` — 集中綁定所有事件，不使用 inline handler
- `drawCanvas(skipSharpen)` — 快速繪製預覽（拖曳時 skipSharpen=true 跳過銳化）
- `render()` — 完整處理管線（繪製 + 壓縮），含 busy/pendingRender 佇列機制
- `refreshLogoUI()` — LOGO 狀態 UI 更新（使用 createElement，不用 innerHTML）
- `loadFile()` — 檔案載入與驗證（大小 20MB、尺寸 8192px 上限）

## Key Constants

| 常數 | 值 | 說明 |
|------|------|------|
| TARGET_W | 800 | 輸出寬度 |
| TARGET_H | 450 | 輸出高度 |
| MAX_BYTES | 153600 | 壓縮目標 150KB |
| MAX_FILE_SIZE | 20971520 | 上傳上限 20MB |
| MAX_DIMENSION | 8192 | 圖片尺寸上限 |
| SHARPEN_STRENGTH | 0.4 | 銳化強度 |

## Image Processing Logic

1. 寬度 >= 800px：縮放填滿 800x450（object-fit: cover），可拖曳調整裁切位置
2. 寬度 < 800px：僅壓縮，不縮放
3. 壓縮策略：二分搜尋 quality 參數（0.2–0.92）逼近 150KB 目標
4. PNG 無 quality 參數，直接輸出
5. 銳化：3x3 unsharp mask 卷積核

## Development

```bash
# 本地預覽（或用 .claude/launch.json 的 image-tool 設定）
python3 -m http.server 8080
```

## Deployment

推送到 `main` 分支會自動觸發 GitHub Actions 部署到 GitHub Pages。

工作流程：`.github/workflows/deploy.yml`

## UI Design

配色參照 tripool 官網部落格風格：
- 主色：綠松色漸層 `#5ec4a8` → `#17BFAB`
- 圓角卡片、淡灰背景 `#f5f6f8`
- 字體：system-ui + Noto Sans TC

## Conventions

- 語言：繁體中文（zh-TW）
- Commit message：中文，格式 `type: 描述`
- 不拆分檔案，維持單一 `index.html`
- 不引入外部 CDN 或框架
