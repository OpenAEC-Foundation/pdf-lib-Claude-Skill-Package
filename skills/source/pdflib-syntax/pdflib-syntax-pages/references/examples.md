# pdflib-syntax-pages — Code Examples

## Example 1: Create a Multi-Page Document with Mixed Sizes

```typescript
import { PDFDocument, PageSizes, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

// Cover page — US Letter
const cover = pdfDoc.addPage(PageSizes.Letter);
const { width: cw, height: ch } = cover.getSize();
cover.drawText('Project Report', {
  x: cw / 2 - font.widthOfTextAtSize('Project Report', 30) / 2,
  y: ch / 2,
  size: 30,
  font,
  color: rgb(0, 0, 0),
});

// Content pages — A4
for (let i = 1; i <= 5; i++) {
  const page = pdfDoc.addPage(PageSizes.A4);
  const { height } = page.getSize();
  page.drawText(`Chapter ${i}`, {
    x: 50,
    y: height - 50, // top of page (bottom-left origin)
    size: 18,
    font,
  });
}

// Appendix — landscape custom
const appendix = pdfDoc.addPage([842, 595]); // A4 landscape
const { height: ah } = appendix.getSize();
appendix.drawText('Appendix — Landscape Layout', {
  x: 50,
  y: ah - 50,
  size: 14,
  font,
});

const pdfBytes = await pdfDoc.save();
```

## Example 2: Insert a Title Page into an Existing PDF

```typescript
import { PDFDocument, PageSizes, StandardFonts, rgb } from 'pdf-lib';

const existingPdfBytes = /* Uint8Array from file read */;
const pdfDoc = await PDFDocument.load(existingPdfBytes);
const font = await pdfDoc.embedFont(StandardFonts.TimesRoman);

// Insert new page at the very beginning
const titlePage = pdfDoc.insertPage(0, PageSizes.Letter);
const { width, height } = titlePage.getSize();

titlePage.drawText('Annual Financial Report', {
  x: width / 2 - font.widthOfTextAtSize('Annual Financial Report', 28) / 2,
  y: height / 2 + 50,
  size: 28,
  font,
  color: rgb(0, 0, 0),
});

titlePage.drawText('Fiscal Year 2025', {
  x: width / 2 - font.widthOfTextAtSize('Fiscal Year 2025', 16) / 2,
  y: height / 2 - 10,
  size: 16,
  font,
  color: rgb(0.4, 0.4, 0.4),
});

const pdfBytes = await pdfDoc.save();
```

## Example 3: Remove Specific Pages from a PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes);

// Remove pages at indices 2 and 5 (remove from highest index first!)
// After removing index 5, index 2 is still valid
pdfDoc.removePage(5);
pdfDoc.removePage(2);

// ANTI-PATTERN: Removing from lowest index first causes index shift
// pdfDoc.removePage(2); // removes page 2
// pdfDoc.removePage(5); // NOW removes what was page 6!

const pdfBytes = await pdfDoc.save();
```

## Example 4: Iterate Pages and Add Page Numbers

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes);
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const pages = pdfDoc.getPages();
const totalPages = pages.length;

for (let i = 0; i < totalPages; i++) {
  const page = pages[i];
  const { width, height } = page.getSize();
  const text = `Page ${i + 1} of ${totalPages}`;
  const textWidth = font.widthOfTextAtSize(text, 10);

  // Bottom center of each page (bottom-left origin, y = 30)
  page.drawText(text, {
    x: (width - textWidth) / 2,
    y: 30,
    size: 10,
    font,
    color: rgb(0.5, 0.5, 0.5),
  });
}

const pdfBytes = await pdfDoc.save();
```

## Example 5: Rotate Pages in an Existing PDF

```typescript
import { PDFDocument, degrees } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes);

// Rotate first page 90 degrees clockwise
const firstPage = pdfDoc.getPage(0);
firstPage.setRotation(degrees(90));

// Rotate all landscape pages to portrait
const pages = pdfDoc.getPages();
for (const page of pages) {
  const { width, height } = page.getSize();
  if (width > height) {
    // Page is landscape — rotate to portrait
    page.setRotation(degrees(90));
  }
}

const pdfBytes = await pdfDoc.save();
```

## Example 6: Set Up Print Production Boxes

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();

// 3mm bleed = 8.504 points (3 * 72 / 25.4)
const bleed = 8.504;

// MediaBox is set automatically by addPage (full page including bleed area)
// For print production, extend media box to include bleed:
page.setMediaBox(
  -bleed, -bleed,
  width + 2 * bleed, height + 2 * bleed
);

// BleedBox — area where content extends for bleed
page.setBleedBox(
  -bleed, -bleed,
  width + 2 * bleed, height + 2 * bleed
);

// TrimBox — final cut size (the actual A4 dimensions)
page.setTrimBox(0, 0, width, height);

// ArtBox — safe area for text/content (10mm margin)
const margin = 28.346; // 10mm in points
page.setArtBox(margin, margin, width - 2 * margin, height - 2 * margin);

const pdfBytes = await pdfDoc.save();
```

## Example 7: Scale and Translate Page Content

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes);
const page = pdfDoc.getPage(0);

// Scale content to 75% — page size stays the same, content shrinks
page.scaleContent(0.75, 0.75);

// Shift content to center it after scaling
const { width, height } = page.getSize();
const xOffset = (width * 0.25) / 2;  // center the 75% content
const yOffset = (height * 0.25) / 2;
page.translateContent(xOffset, yOffset);

const pdfBytes = await pdfDoc.save();
```

## Example 8: Use Page Cursor for Sequential Layout

```typescript
import { PDFDocument, PageSizes, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage(PageSizes.Letter);

// Set page-level defaults
page.setFont(font);
page.setFontSize(12);
page.setFontColor(rgb(0, 0, 0));
page.setLineHeight(16);

const { height } = page.getSize();

// Start cursor at top-left with margin
page.moveTo(50, height - 50);

// Draw heading
page.drawText('Document Title', {
  size: 24, // overrides page default
});

// Move down for content
page.moveDown(40);
page.drawText('This is the first paragraph of content.');

page.moveDown(20);
page.drawText('This is the second paragraph.');

// Check cursor position
const { x, y } = page.getPosition();
// x = 50, y = height - 50 - 40 - 20 = height - 110
```

## Example 9: Create Landscape Page

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();

// Method 1: Swap width and height of a standard size
const [a4Width, a4Height] = PageSizes.A4; // [595.28, 841.89]
const landscapePage = pdfDoc.addPage([a4Height, a4Width]); // [841.89, 595.28]

// Method 2: Custom dimensions
const widePage = pdfDoc.addPage([1000, 500]);

const pdfBytes = await pdfDoc.save();
```

## Example 10: Read All Box Values from an Existing PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes);
const page = pdfDoc.getPage(0);

const mediaBox = page.getMediaBox();
const cropBox = page.getCropBox();
const bleedBox = page.getBleedBox();
const trimBox = page.getTrimBox();
const artBox = page.getArtBox();

// Each returns { x: number, y: number, width: number, height: number }
console.log('MediaBox:', mediaBox);
console.log('CropBox:', cropBox);
console.log('BleedBox:', bleedBox);
console.log('TrimBox:', trimBox);
console.log('ArtBox:', artBox);

// Rotation
const rotation = page.getRotation();
console.log('Rotation:', rotation.angle, 'degrees');

// Dimensions
const { width, height } = page.getSize();
console.log('Size:', width, 'x', height, 'points');
```
