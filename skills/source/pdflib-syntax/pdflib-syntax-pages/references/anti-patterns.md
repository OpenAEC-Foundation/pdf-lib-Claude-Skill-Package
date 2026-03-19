# pdflib-syntax-pages — Anti-Patterns and Common Mistakes

## AP-1: Hardcoding Page Dimensions

**NEVER** hardcode dimension values. ALWAYS use `PageSizes` or `getSize()`.

```typescript
// WRONG — magic numbers, error-prone
const page = pdfDoc.addPage([595, 842]);
page.drawText('Hello', { x: 50, y: 792 });

// CORRECT — use PageSizes enum and getSize()
import { PageSizes } from 'pdf-lib';
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();
page.drawText('Hello', { x: 50, y: height - 50 });
```

## AP-2: Forgetting Bottom-Left Origin

The PDF coordinate system has its origin at the BOTTOM-LEFT. Y increases UPWARD. This is the opposite of HTML/Canvas.

```typescript
// WRONG — places text at bottom, not top
page.drawText('Title', { x: 50, y: 50 });

// CORRECT — calculate from top using height
const { height } = page.getSize();
page.drawText('Title', { x: 50, y: height - 50 });
```

## AP-3: Removing Pages from Lowest Index First

When removing multiple pages, removing from the lowest index first causes all subsequent indices to shift, leading to wrong pages being removed.

```typescript
// WRONG — after removing index 1, what was index 3 is now index 2
pdfDoc.removePage(1);
pdfDoc.removePage(3); // removes the WRONG page

// CORRECT — remove from highest index first
pdfDoc.removePage(3);
pdfDoc.removePage(1); // indices below 3 are unaffected
```

**Rule**: ALWAYS remove pages from highest index to lowest when removing multiple pages.

## AP-4: Adding Pages from Another Document Without copyPages()

NEVER add a page directly from another document. Pages contain internal references that are document-specific.

```typescript
// WRONG — throws error, page belongs to sourceDoc
const sourcePage = sourceDoc.getPage(0);
targetDoc.addPage(sourcePage);

// CORRECT — copy first, then add
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);
```

## AP-5: Using Non-90-Degree Rotation Values

`setRotation()` ONLY accepts multiples of 90 degrees. Other values produce undefined behavior.

```typescript
// WRONG — 45 degrees is not valid for page rotation
page.setRotation(degrees(45));

// CORRECT — use multiples of 90
page.setRotation(degrees(90));
page.setRotation(degrees(180));
page.setRotation(degrees(270));
```

Note: To rotate CONTENT (text, images) by arbitrary angles, use the `rotate` option on `drawText()` or `drawImage()` instead.

## AP-6: Confusing scale() vs scaleContent() vs scaleAnnotations()

These three methods have DIFFERENT effects. Using the wrong one produces unexpected results.

```typescript
// scale() — changes page size AND content AND annotations
page.scale(0.5, 0.5);
// Result: page is now half size, content is half size, annotations are half size

// scaleContent() — changes content ONLY, page size stays the same
page.scaleContent(0.5, 0.5);
// Result: content shrinks to 50% but sits in bottom-left of original-size page

// scaleAnnotations() — changes annotations ONLY
page.scaleAnnotations(0.5, 0.5);
// Result: only annotation positions/sizes change
```

**Rule**: Use `scale()` when you want to resize the entire page proportionally. Use `scaleContent()` when you want to shrink content within the existing page boundary (then use `translateContent()` to reposition).

## AP-7: Not Centering Content After scaleContent()

After calling `scaleContent()`, the content anchors at the bottom-left corner. ALWAYS translate to center if needed.

```typescript
// WRONG — content is scaled but stuck in bottom-left corner
page.scaleContent(0.5, 0.5);

// CORRECT — scale then center
page.scaleContent(0.5, 0.5);
const { width, height } = page.getSize();
page.translateContent(width * 0.25, height * 0.25);
```

## AP-8: Assuming CropBox Equals MediaBox

The CropBox defaults to the MediaBox if not explicitly set, but they are NOT always equal — especially in PDFs from external sources.

```typescript
// WRONG — assuming page size equals visible area
const { width, height } = page.getSize(); // this is MediaBox-based

// CORRECT — check CropBox for visible area when working with existing PDFs
const cropBox = page.getCropBox();
// Use cropBox.width and cropBox.height for the visible region
```

## AP-9: Setting Box Values with Wrong Parameter Order

Box setters take `(x, y, width, height)` — NOT `(left, bottom, right, top)`.

```typescript
// WRONG — these are not corner coordinates
page.setTrimBox(10, 10, 590, 830); // This sets width=590, height=830 starting at (10,10)

// Make sure you understand: x=10, y=10, width=590, height=830
// The box goes from (10, 10) to (600, 840)
```

## AP-10: Forgetting That insertPage() Shifts Indices

After `insertPage(0, ...)`, ALL existing pages shift up by one index.

```typescript
const pdfDoc = await PDFDocument.load(existingPdfBytes);
const originalFirstPage = pdfDoc.getPage(0);

// Insert at index 0
pdfDoc.insertPage(0, PageSizes.Letter);

// WRONG — this is now the INSERTED page, not the original first page
const page = pdfDoc.getPage(0);

// CORRECT — original first page is now at index 1
const page = pdfDoc.getPage(1);
```

## AP-11: Using getPageIndices() When getPageCount() Suffices

`getPageIndices()` creates an array `[0, 1, 2, ...]` which allocates memory unnecessarily for simple iteration.

```typescript
// WRONG — unnecessary array allocation
const indices = pdfDoc.getPageIndices();
for (const i of indices) {
  const page = pdfDoc.getPage(i);
  // ...
}

// CORRECT — use getPages() for iteration
const pages = pdfDoc.getPages();
for (const page of pages) {
  // ...
}

// CORRECT — use getPageCount() for index-based access
const count = pdfDoc.getPageCount();
for (let i = 0; i < count; i++) {
  const page = pdfDoc.getPage(i);
  // ...
}
```

`getPageIndices()` is useful primarily when you need a subset of indices for `copyPages()`.

## AP-12: Mixing Up Page Cursor with Drawing Coordinates

The page cursor (`moveTo`, `moveDown`, etc.) is separate from the `x`/`y` options passed to `drawText()`. When you pass explicit `x`/`y` to `drawText()`, the cursor position is IGNORED for that call.

```typescript
// The cursor position is only used by drawText() when x/y are NOT provided
page.moveTo(100, 500);
page.drawText('Uses cursor position'); // drawn at (100, 500)

page.moveTo(100, 500);
page.drawText('Ignores cursor', { x: 200, y: 300 }); // drawn at (200, 300), NOT (100, 500)
```

## AP-13: Creating Landscape by Rotation Instead of Dimensions

Using `setRotation(degrees(90))` to create a landscape page rotates content too, causing confusion. ALWAYS create landscape pages by swapping width and height.

```typescript
// WRONG — rotates the page AND its content
const page = pdfDoc.addPage(PageSizes.A4);
page.setRotation(degrees(90));

// CORRECT — create with landscape dimensions
const [w, h] = PageSizes.A4;
const page = pdfDoc.addPage([h, w]); // swap width and height
```
