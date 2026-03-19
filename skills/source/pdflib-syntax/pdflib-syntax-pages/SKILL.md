---
name: pdflib-syntax-pages
description: "Guides page operations in pdf-lib including adding, inserting, removing, and getting pages, page dimensions, PageSizes enum, rotation, page boxes, and content transformations. Activates when adding pages, setting page sizes, rotating pages, or manipulating page dimensions."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-syntax-pages — Page Operations in pdf-lib

## Quick Reference

```typescript
import { PDFDocument, PageSizes, degrees } from 'pdf-lib';

// Add pages
const page = pdfDoc.addPage();                    // default size
const page = pdfDoc.addPage([612, 792]);          // custom [width, height]
const page = pdfDoc.addPage(PageSizes.A4);        // named size

// Insert / remove
const page = pdfDoc.insertPage(0);                // insert at index
pdfDoc.removePage(0);                             // remove by index

// Get pages
const page = pdfDoc.getPage(0);                   // single page by index
const pages = pdfDoc.getPages();                  // all pages
const count = pdfDoc.getPageCount();              // total count
const indices = pdfDoc.getPageIndices();           // [0, 1, 2, ...]

// Dimensions
const { width, height } = page.getSize();
page.setSize(612, 792);
page.getWidth(); page.setWidth(612);
page.getHeight(); page.setHeight(792);

// Rotation (multiples of 90 ONLY)
page.setRotation(degrees(90));
const rotation = page.getRotation();              // Rotation object

// Content transformations
page.translateContent(50, 100);                   // shift content
page.scale(0.5, 0.5);                            // scale size + content + annotations
page.scaleContent(0.5, 0.5);                     // scale content only
page.scaleAnnotations(0.5, 0.5);                 // scale annotations only
```

## Critical Warnings

### Coordinate System: Bottom-Left Origin
PDF coordinates start at the BOTTOM-LEFT corner. Y=0 is the BOTTOM of the page.
This is the OPPOSITE of HTML/CSS/Canvas where Y=0 is the top.

```typescript
const { width, height } = page.getSize();

// Top of page — use height minus offset
page.drawText('Near top', { x: 50, y: height - 50 });

// Bottom of page — use small y value
page.drawText('Near bottom', { x: 50, y: 50 });
```

ALWAYS retrieve page dimensions with `getSize()` before positioning content. NEVER hardcode dimension values — use `PageSizes` or `getSize()` instead.

### Units: PDF Points
All dimensions are in PDF points. 1 point = 1/72 inch.
- Letter = 612 x 792 pt (8.5 x 11 inches)
- A4 = 595.28 x 841.89 pt (210 x 297 mm)

### Rotation: Multiples of 90 Only
ALWAYS pass a multiple of 90 to `setRotation()`. Values like `degrees(45)` are NOT valid for page rotation. Use `degrees(0)`, `degrees(90)`, `degrees(180)`, or `degrees(270)`.

### Cross-Document Pages: ALWAYS Use copyPages()
NEVER add a page from one document directly to another. ALWAYS use `copyPages()` first.

```typescript
// WRONG — throws error
targetDoc.addPage(sourceDoc.getPage(0));

// CORRECT
const [copied] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copied);
```

## Essential Patterns

### Pattern 1: Create Document with Specific Page Size

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();
// width = 595.28, height = 841.89
```

### Pattern 2: Add Multiple Pages with Different Sizes

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();

const coverPage = pdfDoc.addPage(PageSizes.Letter);
const contentPage = pdfDoc.addPage(PageSizes.A4);
const widePage = pdfDoc.addPage([1200, 600]); // custom landscape
```

### Pattern 3: Insert Page at Specific Position

```typescript
const pdfDoc = await PDFDocument.create();
pdfDoc.addPage();  // index 0
pdfDoc.addPage();  // index 1

// Insert title page at the beginning
const titlePage = pdfDoc.insertPage(0, PageSizes.Letter);
// titlePage is now index 0, previous pages shift to 1 and 2
```

### Pattern 4: Remove Pages from Existing PDF

```typescript
const pdfDoc = await PDFDocument.load(existingPdfBytes);
const pageCount = pdfDoc.getPageCount();

// Remove last page
pdfDoc.removePage(pageCount - 1);

// Remove first page
pdfDoc.removePage(0);
// After removal, all subsequent indices shift down by 1
```

### Pattern 5: Iterate All Pages

```typescript
const pdfDoc = await PDFDocument.load(existingPdfBytes);
const pages = pdfDoc.getPages();

for (const page of pages) {
  const { width, height } = page.getSize();
  // Process each page...
}
```

### Pattern 6: Rotate a Page

```typescript
import { degrees } from 'pdf-lib';

const page = pdfDoc.getPage(0);
page.setRotation(degrees(90));   // rotate 90 degrees clockwise

// Read current rotation
const current = page.getRotation(); // Rotation object with angle property
```

### Pattern 7: Set Page Boxes for Print Production

```typescript
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();

// Media box = physical page (set automatically by addPage)
// Trim box = final cut size (3mm bleed inset)
const bleed = 8.5; // 3mm in points
page.setTrimBox(bleed, bleed, width - bleed, height - bleed);

// Bleed box = area including bleed allowance
page.setBleedBox(0, 0, width, height);
```

### Pattern 8: Scale Page Content

```typescript
const page = pdfDoc.getPage(0);

// Scale everything (page size, content, and annotations) to 50%
page.scale(0.5, 0.5);

// Scale only the drawn content (not page size or annotations)
page.scaleContent(0.5, 0.5);

// Shift content position
page.translateContent(50, 100); // move right 50pt, up 100pt
```

### Pattern 9: Page-Level Font Defaults

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage(PageSizes.Letter);

// Set defaults — all subsequent drawText calls use these
page.setFont(font);
page.setFontSize(12);
page.setFontColor(rgb(0, 0, 0));
page.setLineHeight(16);

// These calls use the page defaults (no font/size/color needed)
const { height } = page.getSize();
page.drawText('Line 1', { x: 50, y: height - 50 });
page.drawText('Line 2', { x: 50, y: height - 70 });

// Override defaults for a specific call
page.drawText('Bold line', { x: 50, y: height - 90, size: 18 });
```

### Pattern 10: Position Cursor for Sequential Drawing

```typescript
const page = pdfDoc.addPage(PageSizes.Letter);
const { height } = page.getSize();

// Start at top-left
page.moveTo(50, height - 50);

// Move down for each line
page.moveDown(20);
const pos1 = page.getPosition(); // { x: 50, y: height - 70 }

page.moveDown(20);
page.moveRight(100);
const pos2 = page.getPosition(); // { x: 150, y: height - 90 }

// Reset to origin (0, 0)
page.resetPosition();
```

## Decision Tree: Which Page Method?

```
Need to work with pages?
+-- Creating a new page?
|   +-- At the end of the document -> addPage()
|   +-- At a specific position -> insertPage(index)
|   +-- With standard size -> addPage(PageSizes.A4)
|   +-- With custom size -> addPage([width, height])
+-- Accessing existing pages?
|   +-- Single page by index -> getPage(index)
|   +-- All pages -> getPages()
|   +-- Page count -> getPageCount()
+-- Removing a page?
|   +-- By index -> removePage(index)
+-- Changing page size?
|   +-- Both dimensions -> setSize(width, height)
|   +-- Width only -> setWidth(width)
|   +-- Height only -> setHeight(height)
+-- Rotating a page?
|   +-- Set rotation -> setRotation(degrees(90))
|   +-- Read rotation -> getRotation()
+-- Scaling content?
|   +-- Everything (size + content + annotations) -> scale(x, y)
|   +-- Content only -> scaleContent(x, y)
|   +-- Annotations only -> scaleAnnotations(x, y)
|   +-- Shift content position -> translateContent(x, y)
+-- Print production boxes?
    +-- Physical page -> getMediaBox() / setMediaBox()
    +-- Visible area -> getCropBox() / setCropBox()
    +-- Bleed area -> getBleedBox() / setBleedBox()
    +-- Final trim -> getTrimBox() / setTrimBox()
    +-- Art extent -> getArtBox() / setArtBox()
```

## PageSizes Reference (Common Sizes)

| Name | Width (pt) | Height (pt) | Millimeters | Use Case |
|------|-----------|------------|-------------|----------|
| `PageSizes.A3` | 841.89 | 1190.55 | 297 x 420 | Posters, drawings |
| `PageSizes.A4` | 595.28 | 841.89 | 210 x 297 | Standard documents (intl) |
| `PageSizes.A5` | 419.53 | 595.28 | 148 x 210 | Booklets, flyers |
| `PageSizes.Letter` | 612.0 | 792.0 | 216 x 279 | Standard documents (US) |
| `PageSizes.Legal` | 612.0 | 1008.0 | 216 x 356 | Legal documents (US) |
| `PageSizes.Tabloid` | 792.0 | 1224.0 | 279 x 432 | Newspapers, large prints |
| `PageSizes.Executive` | 521.86 | 756.0 | 184 x 267 | Executive memos |
| `PageSizes.Folio` | 612.0 | 936.0 | 216 x 330 | Folio documents |

All 55 sizes (A0-A10, B0-B10, C0-C10, RA0-RA4, SRA0-SRA4, and 5 NA sizes) are available. See [references/methods.md](references/methods.md) for the complete table.

## Page Boxes Explained

| Box | Purpose | Default |
|-----|---------|---------|
| **MediaBox** | Physical page boundary (paper size) | Set by `addPage()` |
| **CropBox** | Region displayed/printed by viewer | Falls back to MediaBox |
| **BleedBox** | Area for printer bleed allowance | Falls back to CropBox |
| **TrimBox** | Intended final page size after trimming | Falls back to CropBox |
| **ArtBox** | Extent of meaningful page content | Falls back to CropBox |

ALWAYS set TrimBox and BleedBox for print-ready PDFs. NEVER assume the viewer defaults match your intent.

## Reference Links

- [Method Signatures](references/methods.md) — All page operation method signatures
- [Code Examples](references/examples.md) — Page manipulation patterns
- [Anti-Patterns](references/anti-patterns.md) — Common page operation mistakes
- [pdf-lib API: PDFPage](https://pdf-lib.js.org/docs/api/classes/pdfpage)
- [pdf-lib API: PDFDocument](https://pdf-lib.js.org/docs/api/classes/pdfdocument)
