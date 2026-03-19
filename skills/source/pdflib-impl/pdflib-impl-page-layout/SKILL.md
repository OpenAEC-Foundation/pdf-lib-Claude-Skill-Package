---
name: pdflib-impl-page-layout
description: "Guides page layout strategies in pdf-lib including coordinate system helpers, margin management, text centering, multi-column layouts, content flow across pages, and top-down positioning patterns. Activates when positioning content on PDF pages, calculating layout, centering elements, or managing content flow."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Page Layout in pdf-lib

## Critical: PDF Coordinate System

pdf-lib uses **bottom-left origin**. `(0, 0)` is the bottom-left corner of the page. The Y axis increases **upward**. This is the opposite of HTML/CSS where `(0, 0)` is the top-left.

```typescript
// ALWAYS get page dimensions first
const { width, height } = page.getSize();

// Top of page: y is NEAR height
page.drawText('Top', { x: 50, y: height - 50 });

// Bottom of page: y is NEAR 0
page.drawText('Bottom', { x: 50, y: 50 });
```

NEVER assume `y: 0` is the top of the page. ALWAYS use `height - offset` for top-down positioning.

---

## Quick Reference

| Task | Pattern |
|------|---------|
| Top-down Y position | `y = height - offset` |
| Horizontal center | `x = (pageWidth - elementWidth) / 2` |
| Vertical center | `y = (pageHeight - elementHeight) / 2` |
| Margin-aware area | `{ x: margin, y: margin, width: w - 2*margin, height: h - 2*margin }` |
| Text width | `font.widthOfTextAtSize(text, fontSize)` |
| Text height | `font.heightAtSize(fontSize)` |
| Content overflow | `if (cursorY - lineHeight < bottomMargin) { /* new page */ }` |

---

## Decision Tree: Choosing a Layout Strategy

```
What layout do you need?
|
+-- Single block of text at a fixed position?
|   -> Use direct drawText() with calculated x, y
|
+-- Sequential content flowing top to bottom?
|   -> Use the cursorY pattern (see "Top-Down CursorY Pattern")
|
+-- Content that may exceed one page?
|   -> Use content flow with page break detection (see "Content Flow Across Pages")
|
+-- Centered title or header?
|   -> Use horizontal centering formula (see "Centering Elements")
|
+-- Multi-column layout (e.g., two-column report)?
|   -> Use column layout pattern (see references/examples.md)
|
+-- Margins and content area?
|   -> Define a contentArea object (see "Margin Management")
```

---

## Top-Down CursorY Pattern

ALWAYS use a `cursorY` variable when placing sequential content from top to bottom. Start at `height - topMargin` and decrement after each element.

```typescript
const { width, height } = page.getSize();
const margin = 50;
let cursorY = height - margin; // Start near top (bottom-left origin!)

// Title
page.drawText('Document Title', {
  x: margin,
  y: cursorY,
  size: 24,
  font: boldFont,
});
cursorY -= 36; // Move down (subtract = move toward bottom)

// Subtitle
page.drawText('A subtitle here', {
  x: margin,
  y: cursorY,
  size: 14,
  font: regularFont,
});
cursorY -= 28;

// Body text
page.drawText('Body paragraph content...', {
  x: margin,
  y: cursorY,
  size: 12,
  font: regularFont,
  lineHeight: 16,
  maxWidth: width - 2 * margin,
});
```

NEVER increment `cursorY` to move down the page. ALWAYS subtract because Y increases upward in PDF coordinates.

---

## Margin Management

ALWAYS define a content area when building structured documents. This prevents content from touching page edges.

```typescript
const { width, height } = page.getSize();
const margin = { top: 72, right: 72, bottom: 72, left: 72 }; // 1 inch = 72pt

const contentArea = {
  x: margin.left,
  y: margin.bottom,
  width: width - margin.left - margin.right,
  height: height - margin.top - margin.bottom,
};

// Start cursor at top of content area (bottom-left origin!)
let cursorY = height - margin.top;

// Check if content fits
const bottomLimit = margin.bottom;
if (cursorY - lineHeight < bottomLimit) {
  // Content would overflow — add new page
}
```

---

## Centering Elements

### Horizontal Centering

```typescript
// Center text horizontally
const text = 'Centered Title';
const fontSize = 24;
const textWidth = font.widthOfTextAtSize(text, fontSize);
const x = (page.getWidth() - textWidth) / 2;

page.drawText(text, { x, y: cursorY, size: fontSize, font });
```

### Vertical Centering

```typescript
// Center text vertically on page
const textHeight = font.heightAtSize(fontSize);
const y = (page.getHeight() - textHeight) / 2;

page.drawText(text, { x: margin, y, size: fontSize, font });
```

### Center an Image

```typescript
const dims = image.scaleToFit(maxWidth, maxHeight);
const x = (page.getWidth() - dims.width) / 2;
const y = (page.getHeight() - dims.height) / 2;

page.drawImage(image, { x, y, width: dims.width, height: dims.height });
```

ALWAYS use `font.widthOfTextAtSize()` to measure text width. NEVER estimate or hardcode text dimensions.

---

## Content Flow Across Pages

When content exceeds a single page, ALWAYS check for overflow before drawing and create a new page when needed.

```typescript
async function drawFlowingContent(
  pdfDoc: PDFDocument,
  lines: string[],
  font: PDFFont,
  fontSize: number,
): Promise<void> {
  const margin = 50;
  const lineHeight = fontSize * 1.4;

  let page = pdfDoc.addPage(PageSizes.A4);
  let { width, height } = page.getSize();
  let cursorY = height - margin;
  const bottomLimit = margin;

  for (const line of lines) {
    // Check if we need a new page BEFORE drawing
    if (cursorY - lineHeight < bottomLimit) {
      page = pdfDoc.addPage(PageSizes.A4);
      ({ width, height } = page.getSize());
      cursorY = height - margin;
    }

    page.drawText(line, {
      x: margin,
      y: cursorY,
      size: fontSize,
      font,
      maxWidth: width - 2 * margin,
    });

    cursorY -= lineHeight;
  }
}
```

NEVER draw text at a negative Y position or below the bottom margin. ALWAYS check `cursorY - lineHeight < bottomLimit` before each draw call.

---

## Text Wrapping with maxWidth

pdf-lib supports automatic word wrapping via the `maxWidth` option. It breaks text at word boundaries (spaces by default).

```typescript
const { width } = page.getSize();
const margin = 50;

page.drawText(longParagraph, {
  x: margin,
  y: cursorY,
  size: 12,
  font,
  maxWidth: width - 2 * margin, // Wrap within margins
  lineHeight: 16,
});
```

**Estimating wrapped text height** (for cursor advancement):

```typescript
function estimateWrappedHeight(
  text: string,
  font: PDFFont,
  fontSize: number,
  maxWidth: number,
  lineHeight: number,
): number {
  const textWidth = font.widthOfTextAtSize(text, fontSize);
  const lineCount = Math.ceil(textWidth / maxWidth);
  return lineCount * lineHeight;
}

// Advance cursor by estimated height
cursorY -= estimateWrappedHeight(text, font, 12, contentWidth, 16);
```

NEVER forget to advance `cursorY` after drawing wrapped text. The wrapped text occupies multiple lines of vertical space.

---

## Page Size Reference

| Size | Points (w x h) | Inches |
|------|----------------|--------|
| A4 | 595.28 x 841.89 | 8.27 x 11.69 |
| Letter | 612 x 792 | 8.5 x 11 |
| Legal | 612 x 1008 | 8.5 x 14 |
| A3 | 841.89 x 1190.55 | 11.69 x 16.54 |

Use `PageSizes.A4`, `PageSizes.Letter`, etc. from pdf-lib. Units are PDF points (1 point = 1/72 inch).

---

## Reference Files

- [references/methods.md](references/methods.md) — Positioning and measurement API methods
- [references/examples.md](references/examples.md) — Layout patterns: margins, centering, columns, flow
- [references/anti-patterns.md](references/anti-patterns.md) — Coordinate mistakes and layout anti-patterns
