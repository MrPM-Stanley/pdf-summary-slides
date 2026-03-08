---
name: pdf-summary-slides
description: "PDF 摘要投影片產生器。自動閱讀 PDF 文件，根據目錄與標題結構切分內容，逐一仔細閱讀後整理成詳盡的學術筆記風格摘要投影片（.pptx），並適當引用原文。投影片數量依 PDF 頁數按比例決定（約每 5 頁產生 1 張投影片），但分配由內容重要程度決定。務必在以下情境觸發此 skill：當使用者提供 PDF 檔案並要求摘要、做筆記、做投影片、整理重點；當使用者說「幫我看這個 PDF」「幫我做 PDF 摘要」「把 PDF 整理成投影片」「PDF 筆記」；當使用者提到 PDF 並希望產出投影片、簡報、筆記；即使使用者只是貼了一個 PDF 並說「幫我整理一下」也應觸發。不要在使用者只是想合併、分割、填表等 PDF 操作時觸發——只在需要「閱讀理解 + 產出投影片」時觸發。"
---

# PDF 摘要投影片產生器

將 PDF 文件轉換為詳盡的學術筆記風格摘要投影片。

## 工作流程總覽

```
讀取 PDF → 辨識結構（目錄/標題） → 逐章節仔細閱讀 → 產生摘要筆記 → 建立投影片 → 存檔至「摘要投影片」目錄
```

---

## 第一步：讀取與分析 PDF 結構

### 提取文字

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    total_pages = len(pdf.pages)
    full_text = ""
    for page in pdf.pages:
        text = page.extract_text()
        if text:
            full_text += text + "\n\n"
```

### 計算投影片數量

投影片數量 = PDF 總頁數 ÷ 5（向上取整），但最少 5 張、最多 60 張。

這個數字是「預算」而非硬性限制。重要的章節可以分到更多張投影片，次要內容則壓縮。分配邏輯：

- 核心論點、方法論、重要發現 → 多分配投影片
- 前言、背景介紹、參考文獻 → 少分配或合併
- 圖表密集的章節 → 獨立一張投影片呈現關鍵數據

### 辨識文件結構

閱讀提取出的文字，辨識：

1. **目錄**（如果有的話）：用來了解整份文件的架構
2. **章節標題與層級**：通常可從字體大小、編號格式（1.、1.1、一、（一）等）辨識
3. **內容區塊邊界**：確定每個章節的起止位置

將文件切分為邏輯區塊，每個區塊對應一個標題或子標題。

---

## 第二步：逐章節深度閱讀與筆記整理

這是最關鍵的步驟。對每個章節區塊：

### 閱讀策略

1. **完整閱讀**該章節的所有內容，不要跳讀
2. **識別核心概念**：這個章節在說什麼？主要論點是什麼？
3. **標記重要細節**：數據、定義、因果關係、比較、流程步驟
4. **挑選值得引用的原文**：特別精闢的表述、關鍵定義、重要結論

### 筆記整理原則

每個章節的筆記應包含：

- **主旨摘要**：用自己的話概括這個章節的核心內容（2-3 句）
- **關鍵要點**：以條列方式列出重要觀點、發現、步驟
- **重要細節**：數據、時間、人物、比較等具體資訊
- **原文引用**：用引號標記直接引用的原文段落，標註「原文：『...』」
- **補充說明**：對複雜概念的額外解釋或跨章節的關聯

### 引用原文的時機

以下情況應引用原文：

- 作者提出的定義或專有名詞解釋
- 關鍵結論或發現的精確表述
- 特別有說服力或具代表性的論述
- 數據或統計結果的精確描述

引用格式：在投影片上用不同顏色或縮排呈現，前面加上「原文：」標記。

---

## 第三步：建立投影片

### 使用 PptxGenJS

```bash
# 確保已安裝
npm list -g pptxgenjs || npm install -g pptxgenjs
npm list -g react-icons react react-dom sharp 2>/dev/null || npm install -g react-icons react react-dom sharp
```

### 學術筆記風格設計規範

這套投影片的目的是事後複習用的詳盡筆記，而非簡報用途。因此：

**配色方案 — 溫暖學術風：**
- 背景：`F8F6F0`（米白色，像筆記紙）
- 主要文字：`2D2D2D`（深灰，非純黑，閱讀舒適）
- 標題：`1A365D`（深藍，學術感）
- 重點標記：`C53030`（暗紅色，像紅筆畫重點）
- 引用區塊背景：`EDF2F7`（淡藍灰）
- 引用文字：`4A5568`（中灰）
- 章節標籤：`2B6CB0`（藍色）

**字體配置：**
- 標題字體：`Cambria`（有學術感的襯線體）
- 內文字體：`Calibri`（清晰易讀）
- 引用字體：`Calibri` + 斜體

**排版原則：**
- 左對齊為主（學術閱讀習慣）
- 內容充實但留適當呼吸空間
- 條列項目之間適度留白
- 引用段落明顯縮排並加背景色塊

### 投影片結構

#### 1. 封面（1 張）

```
[深藍背景 1A365D]

PDF 文件標題
──────────
摘要筆記
日期 | 頁數資訊
```

#### 2. 目錄頁（1 張）

列出所有章節標題及對應的投影片頁碼，讓讀者快速定位。

#### 3. 內容投影片（主體）

每張投影片的標準佈局：

```
┌─────────────────────────────────────────┐
│ [章節標籤] 第X章                          │
│                                          │
│ 章節標題                                  │
│ ─────────                                │
│                                          │
│ ▸ 要點一的詳細說明，包含足夠的脈絡         │
│   讓讀者理解完整意思                       │
│                                          │
│ ▸ 要點二的詳細說明                        │
│                                          │
│ ▸ 要點三                                 │
│                                          │
│ ┌────────────────────────────────────┐   │
│ │ 原文：「這裡放直接引用的原文段落，  │   │
│ │ 用不同背景色突顯」                  │   │
│ └────────────────────────────────────┘   │
│                                    p.XX  │
└─────────────────────────────────────────┘
```

**內容投影片的要素：**
- 左上角的章節標籤（小字，藍色背景色塊）
- 清晰的章節標題
- 3-6 個詳細的要點（每個要點是完整的句子，不是關鍵字）
- 至少 1 個原文引用區塊（淡藍灰背景）
- 右下角標註該內容對應 PDF 的頁碼範圍

**要點的寫法：**
- 每個要點應該是完整的陳述句，讓讀者不看原文也能理解
- 不要只寫「XXX 很重要」，要寫「XXX 之所以重要，是因為 YYY，這導致了 ZZZ」
- 包含具體的數據、時間、比例等細節
- 如有因果關係，明確寫出「因為...所以...」

#### 4. 總結頁（1 張）

整份文件的核心要點回顧（5-8 個最重要的 takeaway）。

### JavaScript 建立投影片的模板

```javascript
const pptxgen = require("pptxgenjs");

const pres = new pptxgen();
pres.layout = "LAYOUT_16x9";
pres.author = "PDF Summary Generator";
pres.title = "PDF 摘要筆記";

// === 顏色常數 ===
const COLORS = {
  bg: "F8F6F0",
  title: "1A365D",
  text: "2D2D2D",
  accent: "C53030",
  quoteBg: "EDF2F7",
  quoteText: "4A5568",
  tagBg: "2B6CB0",
  tagText: "FFFFFF",
  coverBg: "1A365D",
  coverText: "FFFFFF",
  divider: "CBD5E0",
};

// === 封面 ===
function createCoverSlide(title, date, pageCount) {
  const slide = pres.addSlide();
  slide.background = { color: COLORS.coverBg };

  slide.addText(title, {
    x: 0.8, y: 1.2, w: 8.4, h: 2,
    fontSize: 32, fontFace: "Cambria",
    color: COLORS.coverText, bold: true,
    align: "left", valign: "middle",
  });

  // 分隔線
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0.8, y: 3.2, w: 3, h: 0.04,
    fill: { color: "FFFFFF", transparency: 60 },
  });

  slide.addText("摘要筆記", {
    x: 0.8, y: 3.5, w: 8.4, h: 0.6,
    fontSize: 20, fontFace: "Calibri",
    color: COLORS.coverText, italic: true,
    align: "left",
  });

  slide.addText(`${date}  |  原文共 ${pageCount} 頁`, {
    x: 0.8, y: 4.5, w: 8.4, h: 0.5,
    fontSize: 12, fontFace: "Calibri",
    color: "A0AEC0", align: "left",
  });
}

// === 內容投影片 ===
function createContentSlide(chapterTag, title, bullets, quote, pageRef) {
  const slide = pres.addSlide();
  slide.background = { color: COLORS.bg };

  // 章節標籤（截斷過長的標籤文字，只保留簡短標識如「步驟四」）
  let tagLabel = chapterTag;
  if (tagLabel.includes("：")) tagLabel = tagLabel.split("：")[0];
  if (tagLabel.length > 4) tagLabel = tagLabel.substring(0, 4);
  const tagW = Math.max(0.8, tagLabel.length * 0.22 + 0.25);
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0.5, y: 0.3, w: tagW, h: 0.35,
    fill: { color: COLORS.tagBg },
  });
  slide.addText(tagLabel, {
    x: 0.5, y: 0.3, w: tagW, h: 0.35,
    fontSize: 10, fontFace: "Calibri",
    color: COLORS.tagText, align: "center", valign: "middle",
    margin: 0,
  });

  // 標題
  slide.addText(title, {
    x: 0.5, y: 0.8, w: 9, h: 0.6,
    fontSize: 22, fontFace: "Cambria",
    color: COLORS.title, bold: true,
    align: "left", margin: 0,
  });

  // 標題下方分隔線
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0.5, y: 1.45, w: 2, h: 0.03,
    fill: { color: COLORS.divider },
  });

  // 要點（根據有無引用和要點數量動態調整高度和字體大小）
  const bulletY = 1.7;
  const totalTextLen = bullets.reduce((s, b) => s + b.length, 0);
  const bulletFontSize = totalTextLen > 350 ? 10.5 : totalTextLen > 250 ? 11 : 13;
  const bulletH = quote ? (bullets.length > 4 ? 2.6 : 2.0) : 3.2;

  const bulletItems = bullets.map((b, i) => ({
    text: b,
    options: {
      bullet: { code: "25B8" }, // ▸
      breakLine: true,
      fontSize: bulletFontSize,
      fontFace: "Calibri",
      color: COLORS.text,
      paraSpaceAfter: 6,
      indentLevel: 0,
    },
  }));

  slide.addText(bulletItems, {
    x: 0.5, y: bulletY, w: 9, h: bulletH,
    valign: "top",
  });

  // 引用區塊
  if (quote) {
    const quoteY = bulletY + bulletH + 0.15;
    // 引用背景
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.5, y: quoteY, w: 9, h: 1.1,
      fill: { color: COLORS.quoteBg },
    });
    // 左邊裝飾線
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.5, y: quoteY, w: 0.06, h: 1.1,
      fill: { color: COLORS.tagBg },
    });
    // 引用文字
    slide.addText([
      { text: "原文：", options: { bold: true, fontSize: 11, color: COLORS.tagBg } },
      { text: `「${quote}」`, options: { italic: true, fontSize: 11, color: COLORS.quoteText } },
    ], {
      x: 0.75, y: quoteY + 0.08, w: 8.5, h: 0.94,
      valign: "top",
    });
  }

  // 頁碼參考
  if (pageRef) {
    slide.addText(pageRef, {
      x: 7.5, y: 5.1, w: 2, h: 0.4,
      fontSize: 9, fontFace: "Calibri",
      color: "A0AEC0", align: "right",
      italic: true,
    });
  }
}

// === 使用範例 ===
createCoverSlide("文件標題", "2026-03-08", 100);
createContentSlide(
  "第1章",
  "章節標題",
  [
    "第一個要點的完整說明，包含足夠脈絡讓讀者理解",
    "第二個要點，附帶具體數據或例子",
    "第三個要點，說明因果關係",
  ],
  "這裡放原文中特別重要或精闢的段落",
  "p.1-5"
);

pres.writeFile({ fileName: "摘要投影片.pptx" });
```

---

## 第四步：存檔

投影片必須存放在「摘要投影片」目錄中：

```javascript
const fs = require("fs");
const path = require("path");

// 建立「摘要投影片」目錄（如果不存在）
const outputDir = path.join(process.env.OUTPUT_DIR || ".", "摘要投影片");
if (!fs.existsSync(outputDir)) {
  fs.mkdirSync(outputDir, { recursive: true });
}

// 用 PDF 檔名作為投影片檔名
const outputPath = path.join(outputDir, `${pdfBaseName}_摘要.pptx`);
pres.writeFile({ fileName: outputPath });
```

在 Cowork 環境中，輸出目錄為：
```
/sessions/.../mnt/outputs/摘要投影片/
```

---

## 已知注意事項（從實測中學到的）

1. **章節標籤必須簡短**：內容投影片左上角的標籤只用簡短標識（如「步驟四」「前言」「01」），不要放完整的章節名稱，否則會溢出。
2. **動態字體大小**：當一張投影片的要點文字總量超過 250 字時，自動縮小字體（從 13pt 降到 11pt 或 10.5pt），避免內容溢出。
3. **不要使用 ROUNDED_RECTANGLE 配合重疊裝飾**：會有圓角無法被矩形覆蓋的問題，改用 RECTANGLE。
4. **pptxgenjs 選項物件不可重用**：每次呼叫都必須建立新的選項物件（如 shadow），因為 pptxgenjs 會就地修改物件。
5. **pdfplumber 警告是正常的**：某些 PDF 會大量輸出 "Could not get FontBBox" 警告，不影響文字提取，可用 `warnings.filterwarnings('ignore')` 抑制。
6. **大型 PDF 需分章節處理**：超過 100 頁的 PDF 應先將文字按章節分割存檔，再逐章節閱讀整理，以確保深度閱讀品質。

---

## 品質檢查清單

完成投影片後，逐項確認：

1. **內容完整性**：PDF 中的每個主要章節都有對應的投影片
2. **詳盡程度**：每張投影片有 3-6 個實質要點（完整句子，非關鍵字）
3. **引用品質**：每張內容投影片至少有一段原文引用
4. **頁碼對應**：每張投影片標註了對應的 PDF 頁碼範圍
5. **投影片數量**：與 PDF 頁數比例合理（約 1:5）
6. **目錄頁**：目錄頁的頁碼與實際投影片對應正確
7. **視覺一致**：所有投影片風格統一，無遺漏的格式問題

---

## 依賴套件

```bash
pip install pdfplumber --break-system-packages
npm install -g pptxgenjs
```
