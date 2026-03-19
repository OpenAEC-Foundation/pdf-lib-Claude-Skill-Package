# PDF Generation Anti-Patterns

## AP-1: Forgetting to Await Async Methods

The most common runtime error. `embedFont()`, `embedPng()`, `embedJpg()`, `save()`, and `create()` all return Promises.

```typescript
// BROKEN — font is a Promise<PDFFont>, not a PDFFont
const font = pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font }); // Runtime error or silent failure

// CORRECT — ALWAYS await async methods
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font });
```

## AP-2: Embedding Fonts/Images Inside a Loop

Embedding the same font or image on every page wastes memory and CPU. Embed ONCE, reuse everywhere.

```typescript
// BROKEN — embeds the same font N times
for (let i = 0; i < 50; i++) {
  const page = pdfDoc.addPage();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica); // WRONG: inside loop
  page.drawText('Content', { font, x: 50, y: 700, size: 12 });
}

// CORRECT — embed ONCE before the loop
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
for (let i = 0; i < 50; i++) {
  const page = pdfDoc.addPage();
  page.drawText('Content', { font, x: 50, y: 700, size: 12 });
}
```

## AP-3: Coordinate System Confusion (y=0 at Top)

PDF coordinates have origin at bottom-left. y=0 is the BOTTOM of the page, not the top.

```typescript
// BROKEN — text appears at the bottom of the page, not the top
page.drawText('Title', { x: 50, y: 50, size: 24, font });

// CORRECT — subtract from height to position from top
const { height } = page.getSize();
page.drawText('Title', { x: 50, y: height - 50, size: 24, font });
```

## AP-4: No Content Overflow Detection

pdf-lib does NOT auto-paginate. Text drawn below y=0 is silently clipped.

```typescript
// BROKEN — content below the page is invisible
let y = 800;
for (const line of hundredsOfLines) {
  page.drawText(line, { x: 50, y, size: 12, font }); // y goes negative, text lost
  y -= 16;
}

// CORRECT — check Y position, add new page when needed
const bottomMargin = 50;
let y = height - 50;
let page = pdfDoc.addPage(PageSizes.A4);

for (const line of hundredsOfLines) {
  if (y < bottomMargin) {
    page = pdfDoc.addPage(PageSizes.A4);
    y = height - 50;
  }
  page.drawText(line, { x: 50, y, size: 12, font });
  y -= 16;
}
```

## AP-5: Adding Headers/Footers During Content Generation

If you add headers during content creation, you cannot know the total page count for "Page X of Y".

```typescript
// BROKEN — page count unknown during generation
function addPage(pdfDoc: PDFDocument): PDFPage {
  const page = pdfDoc.addPage();
  // Total pages unknown here — "Page 1 of ???"
  page.drawText(`Page ${pdfDoc.getPageCount()} of ???`, {
    x: 250, y: 30, size: 10, font
  });
  return page;
}

// CORRECT — add headers/footers AFTER all content pages exist
// 1. Generate all content pages
// 2. Then iterate all pages to add headers/footers
const pages = pdfDoc.getPages();
const total = pages.length;
pages.forEach((page, i) => {
  page.drawText(`Page ${i + 1} of ${total}`, {
    x: 250, y: 30, size: 10, font
  });
});
```

## AP-6: Using rgb() with 0-255 Values

pdf-lib `rgb()` takes float values between 0.0 and 1.0, NOT integer values 0-255.

```typescript
// BROKEN — creates wrong colors (values overflow)
const red = rgb(255, 0, 0);     // NOT red
const blue = rgb(0, 0, 255);    // NOT blue

// CORRECT — use 0.0 to 1.0 range
const red = rgb(1, 0, 0);
const blue = rgb(0, 0, 1);
const gray = rgb(0.5, 0.5, 0.5);

// Converting from 0-255 to 0-1:
const customColor = rgb(66 / 255, 135 / 255, 245 / 255);
```

## AP-7: Expecting maxWidth to Paginate Across Pages

`maxWidth` on `drawText()` wraps text within a single draw call. It does NOT create new pages.

```typescript
// BROKEN — very long text wraps but overflows below the page
page.drawText(veryLongParagraph, {
  x: 50,
  y: 100, // Only 100pt from bottom — wrapped text will go below 0
  size: 12,
  font,
  maxWidth: 500,
  lineHeight: 16,
});

// CORRECT — split text manually and use content overflow detection
// Or ensure sufficient space before drawing wrapped text
const estimatedLines = Math.ceil(veryLongParagraph.length / 80);
const spaceNeeded = estimatedLines * 16;

if (currentY - spaceNeeded < bottomMargin) {
  page = pdfDoc.addPage(PageSizes.A4);
  currentY = topY;
}

page.drawText(veryLongParagraph, {
  x: 50,
  y: currentY,
  size: 12,
  font,
  maxWidth: 500,
  lineHeight: 16,
});
currentY -= spaceNeeded;
```

## AP-8: Not Setting lineHeight for Multi-Line Text

Without `lineHeight`, multi-line text (with `\n`) uses a default spacing that may overlap.

```typescript
// BROKEN — lines may overlap or have inconsistent spacing
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50,
  y: 700,
  size: 14,
  font,
});

// CORRECT — ALWAYS set lineHeight for multi-line text
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50,
  y: 700,
  size: 14,
  font,
  lineHeight: 20, // Slightly larger than font size
});
```

## AP-9: Distorting Image Aspect Ratio

Setting arbitrary width and height without maintaining the original aspect ratio.

```typescript
// BROKEN — image is distorted (stretched/squished)
page.drawImage(image, {
  x: 50,
  y: 50,
  width: 400,
  height: 100, // Arbitrary — likely wrong aspect ratio
});

// CORRECT — use scale() or scaleToFit() to preserve aspect ratio
const dims = image.scaleToFit(400, 300);
page.drawImage(image, {
  x: 50,
  y: 50,
  width: dims.width,
  height: dims.height,
});
```

## AP-10: Using Unsupported Image Formats

pdf-lib ONLY supports PNG and JPEG. No other formats work.

```typescript
// BROKEN — these methods do NOT exist
await pdfDoc.embedGif(gifBytes);   // NO
await pdfDoc.embedBmp(bmpBytes);   // NO
await pdfDoc.embedSvg(svgString);  // NO
await pdfDoc.embedWebp(webpBytes); // NO

// CORRECT — convert to PNG or JPEG first, then embed
const pngImage = await pdfDoc.embedPng(pngBytes);
const jpgImage = await pdfDoc.embedJpg(jpgBytes);
```

## AP-11: Generating Empty Documents Without Content

By default, `save()` adds a blank page if the document has no pages (`addDefaultPage: true`). This can create unexpected output.

```typescript
// SURPRISE — generates a PDF with one blank page
const pdfDoc = await PDFDocument.create();
const pdfBytes = await pdfDoc.save(); // Contains one blank page

// EXPLICIT — disable default page if generating conditionally
const pdfDoc = await PDFDocument.create();
// ... conditional content that might add zero pages ...
const pdfBytes = await pdfDoc.save({ addDefaultPage: false });
```

## AP-12: Not Setting Document Metadata

Professional documents should ALWAYS include metadata for PDF viewers and search indexing.

```typescript
// BROKEN — anonymous document with no title or author
const pdfDoc = await PDFDocument.create();
// ... content ...
const pdfBytes = await pdfDoc.save();

// CORRECT — set metadata for professional output
const pdfDoc = await PDFDocument.create();
pdfDoc.setTitle('Monthly Report - March 2025', { showInWindowTitleBar: true });
pdfDoc.setAuthor('Acme Corporation');
pdfDoc.setSubject('Financial Summary');
pdfDoc.setKeywords(['report', 'finance', 'march']);
pdfDoc.setCreator('Report Generator v2.0');
pdfDoc.setCreationDate(new Date());
pdfDoc.setLanguage('en-US');
// ... content ...
const pdfBytes = await pdfDoc.save();
```

## AP-13: Using Standard Fonts for Non-Latin Characters

Standard fonts (Helvetica, Times, Courier) ONLY support WinAnsi encoding (basic Latin characters).

```typescript
// BROKEN — crashes with non-WinAnsi characters
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Price: 500', { font }); // FAILS if  is Unicode yen symbol

// CORRECT — use custom font with fontkit for any non-Latin text
import fontkit from '@pdf-lib/fontkit';
pdfDoc.registerFontkit(fontkit);
const font = await pdfDoc.embedFont(customFontBytes);
page.drawText('Price: 500', { font }); // Works with proper font
```

## Summary Table

| # | Anti-Pattern | Consequence | Fix |
|---|-------------|-------------|-----|
| AP-1 | Missing `await` on async methods | Promise passed instead of value | ALWAYS `await` async methods |
| AP-2 | Embedding fonts/images in loops | Memory waste, slow generation | Embed ONCE before loops |
| AP-3 | y=0 at top assumption | Content at bottom of page | Use `height - offset` |
| AP-4 | No overflow detection | Content clipped below page | Check Y, add pages dynamically |
| AP-5 | Headers during generation | Wrong "Page X of Y" | Add headers AFTER all pages |
| AP-6 | rgb(255, 0, 0) | Wrong colors | Use 0.0-1.0 range |
| AP-7 | maxWidth = auto-pagination | Text overflows page | Manual overflow handling |
| AP-8 | Missing lineHeight | Lines overlap | ALWAYS set lineHeight |
| AP-9 | Arbitrary width+height on images | Distorted images | Use scale()/scaleToFit() |
| AP-10 | GIF/BMP/SVG/WebP | Method does not exist | Convert to PNG/JPEG first |
| AP-11 | Empty document saved | Unexpected blank page | Set `addDefaultPage: false` |
| AP-12 | No metadata | Anonymous document | ALWAYS set title, author, dates |
| AP-13 | Standard fonts for Unicode | WinAnsi encoding error | Use custom fonts with fontkit |
