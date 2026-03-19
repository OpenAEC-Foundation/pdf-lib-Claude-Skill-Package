---
name: pdflib-impl-pdf-generation
description: >
  Use when building complete PDF documents from scratch with pdf-lib — invoices,
  reports, certificates, or multi-page layouts. Prevents content overflow by
  providing page-break detection and content flow patterns.
  Covers multi-page generation, headers/footers, page numbering, table layouts, templates.
  Keywords: invoice, report, template, header, footer, page number, table, multi-page, generate.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# PDF Generation Workflows with pdf-lib

## Quick Reference

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

// Minimal generation workflow
const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();

page.drawText('Hello World', {
  x: 50,
  y: height - 50,
  size: 18,
  font,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

### Generation Lifecycle

| Step | Method | Async | Notes |
|------|--------|-------|-------|
| 1. Create document | `PDFDocument.create()` | Yes | ALWAYS await |
| 2. Embed fonts | `pdfDoc.embedFont()` | Yes | Embed ONCE, reuse across pages |
| 3. Embed images | `pdfDoc.embedPng()` / `embedJpg()` | Yes | Embed ONCE, draw many times |
| 4. Add pages | `pdfDoc.addPage()` | No | Returns `PDFPage` |
| 5. Draw content | `page.drawText()` / `drawImage()` / `drawRectangle()` | No | All sync |
| 6. Set metadata | `pdfDoc.setTitle()` etc. | No | Optional |
| 7. Save | `pdfDoc.save()` | Yes | Returns `Uint8Array` |

### Coordinate System Rules

- **Origin**: Bottom-left corner (0, 0)
- **Y axis**: Increases UPWARD (opposite of HTML/CSS)
- **Units**: PDF points (1 pt = 1/72 inch)
- ALWAYS use `height - offset` to position content from the top
- NEVER assume y=0 is the top of the page

```
(0, height) -------- (width, height)   <- top of page
    |                      |
    |     CONTENT AREA     |
    |                      |
  (0, 0) ----------- (width, 0)        <- bottom of page
```

## Decision Trees

### Which Page Size?

```
Need a page size?
+-- Standard paper size?
|   +-- International -> PageSizes.A4 (595.28 x 841.89 pt)
|   +-- North America -> PageSizes.Letter (612 x 792 pt)
|   +-- Legal paper -> PageSizes.Legal (612 x 1008 pt)
+-- Custom dimensions?
|   +-- pdfDoc.addPage([widthPt, heightPt])
+-- No preference?
    +-- pdfDoc.addPage() (default dimensions)
```

### Which Output Format?

```
Saving the PDF?
+-- Write to file (Node.js) -> pdfDoc.save() -> fs.writeFileSync('out.pdf', pdfBytes)
+-- Send as HTTP response -> pdfDoc.save() -> Buffer.from(pdfBytes)
+-- Display in browser -> pdfDoc.saveAsBase64({ dataUri: true }) -> set as iframe src
+-- Store in database -> pdfDoc.save() -> store Uint8Array as blob
+-- Email attachment -> pdfDoc.saveAsBase64() -> attach as base64
```

### Content Overflow?

```
Content might not fit on page?
+-- Track current Y position manually
+-- When Y < bottomMargin -> add new page, reset Y to top
+-- NEVER rely on pdf-lib to auto-paginate (it does NOT)
```

## Core Patterns

### Pattern 1: Multi-Page Document with Content Overflow

ALWAYS track the current Y position manually. pdf-lib has NO automatic pagination.

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

interface PageLayout {
  topMargin: number;
  bottomMargin: number;
  leftMargin: number;
  rightMargin: number;
  lineHeight: number;
  fontSize: number;
}

async function generateMultiPageDocument(
  lines: string[],
  layout: PageLayout,
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

  let page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();
  let currentY = height - layout.topMargin;

  for (const line of lines) {
    // Content overflow detection — add new page when needed
    if (currentY < layout.bottomMargin) {
      page = pdfDoc.addPage(PageSizes.A4);
      currentY = height - layout.topMargin;
    }

    page.drawText(line, {
      x: layout.leftMargin,
      y: currentY,
      size: layout.fontSize,
      font,
      color: rgb(0, 0, 0),
    });

    currentY -= layout.lineHeight;
  }

  return pdfDoc.save();
}
```

### Pattern 2: Headers and Footers on Every Page

ALWAYS add headers and footers AFTER all content pages are created, by iterating over all pages.

```typescript
function addHeadersAndFooters(
  pdfDoc: PDFDocument,
  font: PDFFont,
  headerText: string,
): void {
  const pages = pdfDoc.getPages();
  const totalPages = pages.length;

  for (let i = 0; i < totalPages; i++) {
    const page = pages[i];
    const { width, height } = page.getSize();

    // Header
    page.drawText(headerText, {
      x: 50,
      y: height - 30,
      size: 10,
      font,
      color: rgb(0.4, 0.4, 0.4),
    });

    // Header separator line
    page.drawLine({
      start: { x: 50, y: height - 35 },
      end: { x: width - 50, y: height - 35 },
      thickness: 0.5,
      color: rgb(0.7, 0.7, 0.7),
    });

    // Footer with page numbering
    const pageNumText = `Page ${i + 1} of ${totalPages}`;
    const pageNumWidth = font.widthOfTextAtSize(pageNumText, 10);
    page.drawText(pageNumText, {
      x: width - 50 - pageNumWidth,
      y: 30,
      size: 10,
      font,
      color: rgb(0.4, 0.4, 0.4),
    });

    // Footer separator line
    page.drawLine({
      start: { x: 50, y: 45 },
      end: { x: width - 50, y: 45 },
      thickness: 0.5,
      color: rgb(0.7, 0.7, 0.7),
    });
  }
}
```

### Pattern 3: Table-Like Layout

pdf-lib has NO table primitive. ALWAYS build tables manually with rectangles and text.

```typescript
interface TableColumn {
  header: string;
  width: number;
  align?: 'left' | 'right';
}

function drawTable(
  page: PDFPage,
  font: PDFFont,
  columns: TableColumn[],
  rows: string[][],
  startX: number,
  startY: number,
  rowHeight: number,
  fontSize: number,
): number {
  const headerBg = rgb(0.9, 0.9, 0.9);
  const borderColor = rgb(0.5, 0.5, 0.5);
  const totalWidth = columns.reduce((sum, col) => sum + col.width, 0);
  let currentY = startY;

  // Draw header row background
  page.drawRectangle({
    x: startX,
    y: currentY - rowHeight,
    width: totalWidth,
    height: rowHeight,
    color: headerBg,
    borderColor,
    borderWidth: 0.5,
  });

  // Draw header text
  let cellX = startX;
  for (const col of columns) {
    page.drawText(col.header, {
      x: cellX + 5,
      y: currentY - rowHeight + 5,
      size: fontSize,
      font,
      color: rgb(0, 0, 0),
    });
    cellX += col.width;
  }
  currentY -= rowHeight;

  // Draw data rows
  for (const row of rows) {
    cellX = startX;
    for (let c = 0; c < columns.length; c++) {
      // Cell border
      page.drawRectangle({
        x: cellX,
        y: currentY - rowHeight,
        width: columns[c].width,
        height: rowHeight,
        borderColor,
        borderWidth: 0.5,
      });

      // Cell text
      const text = row[c] || '';
      let textX = cellX + 5;
      if (columns[c].align === 'right') {
        const textWidth = font.widthOfTextAtSize(text, fontSize);
        textX = cellX + columns[c].width - textWidth - 5;
      }

      page.drawText(text, {
        x: textX,
        y: currentY - rowHeight + 5,
        size: fontSize,
        font,
        color: rgb(0, 0, 0),
      });
      cellX += columns[c].width;
    }
    currentY -= rowHeight;
  }

  return currentY; // Return Y position after table for further content
}
```

### Pattern 4: Centering Text

```typescript
function drawCenteredText(
  page: PDFPage,
  font: PDFFont,
  text: string,
  y: number,
  fontSize: number,
  color: Color = rgb(0, 0, 0),
): void {
  const textWidth = font.widthOfTextAtSize(text, fontSize);
  const pageWidth = page.getWidth();
  page.drawText(text, {
    x: (pageWidth - textWidth) / 2,
    y,
    size: fontSize,
    font,
    color,
  });
}
```

### Pattern 5: Save and Output

```typescript
// Save as Uint8Array (Node.js file write, HTTP response)
const pdfBytes: Uint8Array = await pdfDoc.save();

// Node.js: write to file
import { writeFileSync } from 'fs';
writeFileSync('output.pdf', pdfBytes);

// Save as Base64 (email attachment, database storage)
const base64String: string = await pdfDoc.saveAsBase64();

// Save as data URI (browser display in iframe)
const dataUri: string = await pdfDoc.saveAsBase64({ dataUri: true });
// In browser: document.getElementById('iframe').src = dataUri;
```

## Critical Rules

1. **ALWAYS await async methods**: `create()`, `embedFont()`, `embedPng()`, `embedJpg()`, `save()`, `saveAsBase64()` are ALL async. Forgetting `await` on `embedFont()` passes a Promise instead of a PDFFont to `drawText()`.

2. **ALWAYS embed fonts and images ONCE before the page loop**: Embedding is expensive. Embed at document level, then reuse the returned `PDFFont`/`PDFImage` on every page.

3. **ALWAYS track Y position manually for multi-page content**: pdf-lib does NOT auto-paginate. When `currentY < bottomMargin`, call `pdfDoc.addPage()` and reset `currentY`.

4. **ALWAYS use `height - offset` for top-down positioning**: The PDF coordinate origin is bottom-left. To place content 50pt from the top: `y = height - 50`.

5. **NEVER assume pdf-lib wraps text across pages**: `maxWidth` wraps text within a single `drawText()` call on one page. It does NOT continue onto the next page.

6. **ALWAYS add headers/footers AFTER generating all content pages**: Iterate `pdfDoc.getPages()` to stamp headers, footers, and page numbers on every page, so `totalPages` is accurate.

7. **NEVER use rgb values 0-255**: pdf-lib `rgb()` takes values 0.0 to 1.0. `rgb(255, 0, 0)` does NOT create red.

8. **ALWAYS set metadata for professional documents**:
   ```typescript
   pdfDoc.setTitle('Invoice #12345', { showInWindowTitleBar: true });
   pdfDoc.setAuthor('Company Name');
   pdfDoc.setCreator('My Application');
   pdfDoc.setCreationDate(new Date());
   ```

## Reference Links

- [Key API Methods](references/methods.md) — Font embedding, page creation, drawing methods
- [Complete Generation Examples](references/examples.md) — Invoice template, report template, multi-page document
- [Anti-Patterns](references/anti-patterns.md) — Common generation mistakes and their fixes
