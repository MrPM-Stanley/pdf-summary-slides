---
name: pdf-summary-slides
description: "PDF 摘要投影片產生器。自動閱讀 PDF 文件，根據目錄與標題結構切分內容，逐一仔細閱讀後整理成詳盡的學術筆記風格摘要投影片（.pptx），並適當引用原文。投影片數量依 PDF 頁數按比例決定（約每 5 頁產生 1 張投影片），但分配由內容重要程度決定。務必在以下情境觸發此 skill：當使用者提供 PDF 檔案並要求摘要、做筆記、做投影片、整理重點；當使用者說「幫我看這個 PDF」「幫我做 PDF 摘要」「把 PDF 整理成投影片」「PDF 筆記」；當使用者提到 PDF 並希望產出投影片、簡報、筆記；即使使用者只是貼了一個 PDF 並說「幫我整理一下」也應觸發。不要在使用者只是想合併、分割、填表等 PDF 操作時觸發——只在需要「閱讀理解 + 產出投影片」時觸發。"
---

# PDF 摘要投影片產生器

將 PDF 文件轉換為詳盡的學術筆記風格摘要投影片，使用 Stanley 的深海軍藍專業視覺風格。

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

引用格式：在投影片上用引用區塊呈現（深藍左邊線 + 淡藍灰底色），前面加上「原文：」標記。

---

## 第三步：建立投影片

### 使用 PptxGenJS

```bash
# 確保已安裝
npm list -g pptxgenjs || npm install -g pptxgenjs
```

### Stanley 深海軍藍風格設計規範

這套投影片的目的是事後複習用的詳盡筆記。視覺風格採用 Stanley 的深海軍藍專業配色，乾淨俐落。

**配色方案：**
```
主色（深海軍藍）:  #19315A  — 標題列背景、分隔頁背景、底部裝飾線
內文深藍:          #1F3864  — 一般內文、bullet points
純黑:              #000000  — 頁碼方塊
純白:              #FFFFFF  — 深色背景上的文字、投影片底色
強調紅:            #C53030  — 原文引用的「原文：」標記
引用區塊背景:      #EDF2F7  — 引用段落的底色
引用區塊文字:      #4A5568  — 引用段落的文字顏色
引用邊線/標籤:     #19315A  — 引用左邊裝飾線、章節標籤底色
頁碼參考文字:      #A0AEC0  — PDF 頁碼標註
```

**字體配置：**
- 標題列白字：`Microsoft JhengHei`（微軟正黑體），粗體
- 章節標題：`Microsoft JhengHei`，粗體
- 內文：`Microsoft JhengHei`
- 英文/數字/小字：`Arial` 或 `Calibri`
- 引用文字：`Microsoft JhengHei` + 斜體

**排版原則：**
- 每張內容頁頂部有深藍標題列（白字標題）
- 每張內容頁底部有細深藍裝飾線
- 右下角有黑底白字頁碼方塊
- 白色背景，文字左對齊
- 分隔頁和小結頁使用全版深藍背景
- 不要用漸層、陰影、圓角
- 不要在標題下方加裝飾底線

### 投影片結構

#### 1. 封面（1 張）

深海軍藍全版背景，大標題靠左。

#### 2. 目錄頁（1 張）

白底，列出所有章節標題及對應的投影片頁碼。

#### 3. 章節分隔頁

每個大章節開始前，插入一張深藍全版背景的分隔頁，白色外框矩形 + 白字標題居中。

#### 4. 內容投影片（主體）

每張投影片的標準佈局：
```
┌──────────────────────────────────────────┐
│▓▓▓▓▓ 章節標題（白字）▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│  ← 深藍標題列
│                                          │
│ [標籤] 要點一的詳細說明，包含足夠脈絡     │
│                                          │
│ • 要點二的詳細說明                        │
│                                          │
│ • 要點三                                 │
│                                          │
│ ┃ 原文：「直接引用的原文段落」            │  ← 淡藍灰底 + 深藍左邊線
│                                          │
│▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔│  ← 底部細裝飾線
│                               p.XX  [N] │  ← 頁碼參考 + 頁碼方塊
└──────────────────────────────────────────┘
```

**內容投影片的要素：**
- 頂部深藍標題列 + 白字標題（這是最重要的識別元素）
- 左上角可選的章節標籤（深藍背景白字小方塊）
- 3-6 個詳細的要點（每個要點是完整的句子）
- 至少 1 個原文引用區塊（淡藍灰背景 + 深藍左邊線）
- 底部細深藍裝飾線
- 右下角黑底白字頁碼方塊
- 頁碼方塊左側可標註 PDF 頁碼範圍

**要點的寫法：**
- 每個要點應該是完整的陳述句，讓讀者不看原文也能理解
- 不要只寫「XXX 很重要」，要寫「XXX 之所以重要，是因為 YYY，這導致了 ZZZ」
- 包含具體的數據、時間、比例等細節
- 如有因果關係，明確寫出「因為...所以...」

#### 5. 總結頁（1 張）

深藍全版背景，白字列出整份文件的核心要點回顧（5-8 個最重要的 takeaway）。

### JavaScript 建立投影片的完整模板

```javascript
const pptxgen = require("pptxgenjs");

const pres = new pptxgen();
pres.layout = "LAYOUT_16x9";
pres.author = "PDF Summary Generator";
pres.title = "PDF 摘要筆記";

// === 顏色常數（Stanley 深海軍藍風格）===
const COLORS = {
  headerBg: "19315A",     // 標題列背景
  headerText: "FFFFFF",   // 標題列白字
  bodyText: "1F3864",     // 內文深藍
  black: "000000",        // 頁碼方塊
  white: "FFFFFF",        // 背景、深色上的文字
  accentRed: "C53030",    // 原文引用「原文：」標記
  quoteBg: "EDF2F7",      // 引用區塊背景
  quoteText: "4A5568",    // 引用文字
  quoteLine: "19315A",    // 引用左邊裝飾線
  tagBg: "19315A",        // 章節標籤背景
  tagText: "FFFFFF",      // 章節標籤文字
  pageRef: "A0AEC0",      // PDF 頁碼參考文字
};

// === 字體常數 ===
const FONTS = {
  heading: "Microsoft JhengHei",
  body: "Microsoft JhengHei",
  english: "Arial",
  small: "Calibri",
};

// === 共用元素：標題列 + 底部裝飾線 + 頁碼方塊 ===
function addCommonElements(slide, titleText, slideNumber) {
  // 頂部深藍標題列
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 0, w: 10, h: 0.72,
    fill: { color: COLORS.headerBg }
  });
  slide.addText(titleText, {
    x: 0.5, y: 0, w: 9, h: 0.72,
    fontSize: 20, fontFace: FONTS.heading,
    color: COLORS.headerText, bold: true,
    valign: "middle", margin: 0
  });

  // 底部細裝飾線
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 5.58, w: 10, h: 0.045,
    fill: { color: COLORS.headerBg }
  });

  // 頁碼方塊（右下角，黑底白字）
  if (slideNumber !== undefined) {
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fill: { color: COLORS.black }
    });
    slide.addText(String(slideNumber), {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fontSize: 12, fontFace: FONTS.english,
      color: COLORS.white, align: "center", valign: "middle", margin: 0
    });
  }
}

// === 封面 ===
function createCoverSlide(title, subtitle, date, pageCount) {
  const slide = pres.addSlide();

  // 全版深藍背景
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 0, w: 10, h: 5.625,
    fill: { color: COLORS.headerBg }
  });

  // 主標題
  slide.addText(title, {
    x: 0.6, y: 1.5, w: 7, h: 1.8,
    fontSize: 36, fontFace: FONTS.heading,
    color: COLORS.white, bold: true,
    valign: "middle", margin: 0
  });

  // 副標題「摘要筆記」
  slide.addText(subtitle || "摘要筆記", {
    x: 0.6, y: 3.3, w: 7, h: 0.7,
    fontSize: 22, fontFace: FONTS.heading,
    color: COLORS.white, valign: "top", margin: 0
  });

  // 日期 + 頁數資訊
  slide.addText(`${date}  |  原文共 ${pageCount} 頁`, {
    x: 0.6, y: 4.3, w: 5, h: 0.5,
    fontSize: 14, fontFace: FONTS.small,
    color: COLORS.white, valign: "top", margin: 0
  });

  return slide;
}

// === 目錄頁 ===
function createTocSlide(tocItems, slideNumber) {
  // tocItems = [{ title: "章節名稱", page: 3 }, ...]
  const slide = pres.addSlide();
  addCommonElements(slide, "目錄", slideNumber);

  const tocText = tocItems.map((item, i) => ({
    text: `${item.title}  ···  ${item.page}`,
    options: {
      breakLine: true,
      fontSize: 14,
      fontFace: FONTS.body,
      color: COLORS.bodyText,
      paraSpaceAfter: 8,
    }
  }));

  slide.addText(tocText, {
    x: 0.8, y: 1.0, w: 8.4, h: 4.0,
    valign: "top"
  });

  return slide;
}

// === 章節分隔頁 ===
function createSectionSlide(sectionTitle, slideNumber) {
  const slide = pres.addSlide();

  // 全版深藍背景
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 0, w: 10, h: 5.625,
    fill: { color: COLORS.headerBg }
  });

  // 白色外框矩形
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 1.5, y: 1.2, w: 7, h: 3.2,
    fill: { type: "none" },
    line: { color: COLORS.white, width: 2 }
  });

  // 章節標題
  slide.addText(sectionTitle, {
    x: 2, y: 1.8, w: 6, h: 2.0,
    fontSize: 28, fontFace: FONTS.heading,
    color: COLORS.white, bold: true,
    align: "center", valign: "middle", margin: 0
  });

  // 頁碼方塊
  if (slideNumber !== undefined) {
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fill: { color: COLORS.black }
    });
    slide.addText(String(slideNumber), {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fontSize: 12, fontFace: FONTS.english,
      color: COLORS.white, align: "center", valign: "middle", margin: 0
    });
  }

  return slide;
}

// === 內容投影片 ===
function createContentSlide(chapterTag, title, bullets, quote, pageRef, slideNumber) {
  const slide = pres.addSlide();
  addCommonElements(slide, title, slideNumber);

  // 章節標籤（標題列下方，小深藍色塊）
  if (chapterTag) {
    let tagLabel = chapterTag;
    if (tagLabel.includes("：")) tagLabel = tagLabel.split("：")[0];
    if (tagLabel.length > 6) tagLabel = tagLabel.substring(0, 6);
    const tagW = Math.max(0.8, tagLabel.length * 0.22 + 0.3);
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.5, y: 0.85, w: tagW, h: 0.3,
      fill: { color: COLORS.tagBg }
    });
    slide.addText(tagLabel, {
      x: 0.5, y: 0.85, w: tagW, h: 0.3,
      fontSize: 10, fontFace: FONTS.small,
      color: COLORS.tagText, align: "center", valign: "middle", margin: 0
    });
  }

  // 要點區域起始位置（有標籤時往下移）
  const bulletY = chapterTag ? 1.3 : 0.9;
  const totalTextLen = bullets.reduce((s, b) => s + b.length, 0);
  const bulletFontSize = totalTextLen > 350 ? 10.5 : totalTextLen > 250 ? 11 : 13;
  const bulletH = quote ? (bullets.length > 4 ? 2.4 : 1.9) : 3.8;

  const bulletItems = bullets.map((b, i) => ({
    text: b,
    options: {
      bullet: true,
      breakLine: true,
      fontSize: bulletFontSize,
      fontFace: FONTS.body,
      color: COLORS.bodyText,
      paraSpaceAfter: 6,
      lineSpacingMultiple: 1.2,
    },
  }));

  slide.addText(bulletItems, {
    x: 0.5, y: bulletY, w: 9.0, h: bulletH,
    valign: "top",
  });

  // 引用區塊
  if (quote) {
    const quoteY = bulletY + bulletH + 0.15;
    const quoteH = Math.min(1.2, Math.max(0.8, quote.length * 0.012 + 0.4));

    // 引用背景
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.5, y: quoteY, w: 9.0, h: quoteH,
      fill: { color: COLORS.quoteBg }
    });
    // 左邊深藍裝飾線
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 0.5, y: quoteY, w: 0.06, h: quoteH,
      fill: { color: COLORS.quoteLine }
    });
    // 引用文字
    slide.addText([
      { text: "原文：", options: { bold: true, fontSize: 11, fontFace: FONTS.body, color: COLORS.accentRed } },
      { text: `「${quote}」`, options: { italic: true, fontSize: 11, fontFace: FONTS.body, color: COLORS.quoteText } },
    ], {
      x: 0.75, y: quoteY + 0.06, w: 8.5, h: quoteH - 0.12,
      valign: "top",
    });
  }

  // PDF 頁碼參考（頁碼方塊左側）
  if (pageRef) {
    slide.addText(pageRef, {
      x: 7.0, y: 5.18, w: 2.1, h: 0.3,
      fontSize: 9, fontFace: FONTS.small,
      color: COLORS.pageRef, align: "right", italic: true, margin: 0
    });
  }

  return slide;
}

// === 總結頁 ===
function createSummarySlide(takeaways, slideNumber) {
  // takeaways = ["要點一", "要點二", ...]
  const slide = pres.addSlide();

  // 全版深藍背景
  slide.addShape(pres.shapes.RECTANGLE, {
    x: 0, y: 0, w: 10, h: 5.625,
    fill: { color: COLORS.headerBg }
  });

  // 標題
  slide.addText("重點回顧", {
    x: 0.6, y: 0.4, w: 8.8, h: 0.7,
    fontSize: 26, fontFace: FONTS.heading,
    color: COLORS.white, bold: true, margin: 0
  });

  // 要點列表
  const items = takeaways.map((t, i) => ({
    text: t,
    options: {
      bullet: true,
      breakLine: true,
      fontSize: 14,
      fontFace: FONTS.body,
      color: COLORS.white,
      paraSpaceAfter: 8,
      lineSpacingMultiple: 1.3,
    }
  }));

  slide.addText(items, {
    x: 0.6, y: 1.3, w: 8.8, h: 3.8,
    valign: "top"
  });

  // 頁碼方塊
  if (slideNumber !== undefined) {
    slide.addShape(pres.shapes.RECTANGLE, {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fill: { color: COLORS.black }
    });
    slide.addText(String(slideNumber), {
      x: 9.3, y: 5.15, w: 0.5, h: 0.35,
      fontSize: 12, fontFace: FONTS.english,
      color: COLORS.white, align: "center", valign: "middle", margin: 0
    });
  }

  return slide;
}

// === 使用範例 ===
createCoverSlide("文件標題", "摘要筆記", "2026-03-11", 100);
createTocSlide([
  { title: "第一章：導論", page: 3 },
  { title: "第二章：方法論", page: 5 },
], 2);
createSectionSlide("PART I：導論", 3);
createContentSlide(
  "第1章",
  "章節標題放在深藍標題列中",
  [
    "第一個要點的完整說明，包含足夠脈絡讓讀者理解",
    "第二個要點，附帶具體數據或例子",
    "第三個要點，說明因果關係：因為 A 所以 B",
  ],
  "這裡放原文中特別重要或精闢的段落",
  "p.1-5",
  4
);
createSummarySlide([
  "核心要點一", "核心要點二", "核心要點三"
], 20);

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

## 風格設計原則

1. **頂部標題列是靈魂**：每張內容頁都必須有深藍標題列 + 白字標題，這是最核心的識別元素
2. **底部裝飾線不可省略**：細深藍線收束頁面，和頂部標題列形成視覺框架
3. **分隔頁和總結頁用全深藍背景**：打破節奏，區分章節
4. **色彩克制**：只用定義好的色彩系統，不要自行發明新顏色
5. **乾淨扁平**：不要用漸層、陰影、圓角。風格是扁平直接的
6. **不要加標題底線裝飾**：不要在標題下方加裝飾線，那是 AI 風格的通病
7. **頁碼一致**：右下角黑底白字小方塊，所有頁面都要有
8. **字體統一**：中文一律微軟正黑體，英文用 Arial

---

## 已知注意事項（從實測中學到的）

1. **章節標籤必須簡短**：內容投影片標題列下方的標籤只用簡短標識（如「第1章」「前言」「01」），不要放完整的章節名稱，否則會溢出。
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
7. **風格一致**：所有投影片都有深藍標題列 + 底部裝飾線 + 頁碼方塊
8. **分隔頁到位**：每個大章節前有深藍全版分隔頁

---

## 依賴套件

```bash
pip install pdfplumber --break-system-packages
npm install -g pptxgenjs
```
