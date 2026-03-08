# pdf-summary-slides

PDF 摘要投影片產生器 — Claude Code / Cowork Skill

自動閱讀 PDF 文件，根據目錄與標題結構切分內容，逐一仔細閱讀後整理成詳盡的學術筆記風格摘要投影片（.pptx），並適當引用原文。

## 功能

- 自動辨識 PDF 章節結構（目錄、標題層級）
- 逐章節深度閱讀，產出詳盡的筆記式摘要
- 投影片數量 ≈ PDF 頁數 ÷ 5，依重要程度分配
- 每張投影片包含完整要點 + 原文引用
- 學術筆記風格（米白背景、深藍標題、紅色重點標記）
- 自動存檔至「摘要投影片」目錄

## 安裝

```bash
# Claude Code
claude install-skill pdf-summary-slides.skill

# 或直接把 pdf-summary-slides/ 資料夾放到你的 skills 目錄
```

## 觸發方式

上傳 PDF 後說：

- 「幫我看這個 PDF」
- 「幫我做 PDF 摘要」
- 「把 PDF 整理成投影片」
- 「幫我整理一下」

## 依賴

- `pdfplumber`（Python）— PDF 文字提取
- `pptxgenjs`（Node.js）— 投影片產生

## 授權

MIT
