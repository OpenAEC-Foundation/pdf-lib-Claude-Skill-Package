# pdflib-syntax-images — Code Examples

## Basic Image Embedding and Drawing

```typescript
import { PDFDocument } from 'pdf-lib';

// Load image bytes (from file system, fetch, etc.)
const jpgImageBytes: Uint8Array = /* ... */;
const pngImageBytes: Uint8Array = /* ... */;

const pdfDoc = await PDFDocument.create();
const jpgImage = await pdfDoc.embedJpg(jpgImageBytes);
const pngImage = await pdfDoc.embedPng(pngImageBytes);

const page = pdfDoc.addPage();

// Draw JPG at original size
page.drawImage(jpgImage, { x: 50, y: 400 });

// Draw PNG at original size
page.drawImage(pngImage, { x: 50, y: 200 });

const pdfBytes = await pdfDoc.save();
```

## Embedding from Base64 String

```typescript
import { PDFDocument } from 'pdf-lib';

const base64Png = 'iVBORw0KGgoAAAANSUhEUg...'; // base64 without prefix
const dataUriJpg = 'data:image/jpeg;base64,/9j/4AAQ...'; // data URI with prefix

const pdfDoc = await PDFDocument.create();

// Both formats work for both methods
const pngImage = await pdfDoc.embedPng(base64Png);
const jpgImage = await pdfDoc.embedJpg(dataUriJpg);
```

## Embedding from Fetch (Browser/Node 18+)

```typescript
import { PDFDocument } from 'pdf-lib';

const pngUrl = 'https://example.com/logo.png';
const pngBytes = await fetch(pngUrl).then((res) => res.arrayBuffer());

const pdfDoc = await PDFDocument.create();
const pngImage = await pdfDoc.embedPng(pngBytes);

const page = pdfDoc.addPage();
const dims = pngImage.scale(0.5);
page.drawImage(pngImage, {
  x: 50, y: 50,
  width: dims.width, height: dims.height,
});

const pdfBytes = await pdfDoc.save();
```

## Embedding from Node.js File System

```typescript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

const jpgBytes = fs.readFileSync('/path/to/photo.jpg');

const pdfDoc = await PDFDocument.create();
const jpgImage = await pdfDoc.embedJpg(jpgBytes);

const page = pdfDoc.addPage();
page.drawImage(jpgImage, { x: 50, y: 50 });

const pdfBytes = await pdfDoc.save();
fs.writeFileSync('/path/to/output.pdf', pdfBytes);
```

## Aspect Ratio: Uniform Scale

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedPng(pngBytes);

// Scale to 25% of original size — aspect ratio ALWAYS preserved
const dims = image.scale(0.25);

const page = pdfDoc.addPage();
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});

const pdfBytes = await pdfDoc.save();
```

## Aspect Ratio: Fit Within Bounds

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedJpg(jpgBytes);

// Fit within 400x300 area — aspect ratio ALWAYS preserved
// The returned dimensions will NEVER exceed 400 wide or 300 tall
const dims = image.scaleToFit(400, 300);

const page = pdfDoc.addPage();
page.drawImage(image, {
  x: 100, y: 250,
  width: dims.width,
  height: dims.height,
});

const pdfBytes = await pdfDoc.save();
```

## Aspect Ratio: Manual Calculation (Fixed Width)

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedPng(pngBytes);

// Set exact width, calculate height to preserve aspect ratio
const targetWidth = 400;
const scaleFactor = targetWidth / image.width;
const targetHeight = image.height * scaleFactor;

const page = pdfDoc.addPage();
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: targetHeight,
});

const pdfBytes = await pdfDoc.save();
```

## Aspect Ratio: Manual Calculation (Fixed Height)

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedJpg(jpgBytes);

// Set exact height, calculate width to preserve aspect ratio
const targetHeight = 200;
const scaleFactor = targetHeight / image.height;
const targetWidth = image.width * scaleFactor;

const page = pdfDoc.addPage();
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: targetHeight,
});

const pdfBytes = await pdfDoc.save();
```

## Centering an Image on a Page

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedPng(pngBytes);

const page = pdfDoc.addPage(PageSizes.A4);
const { width: pageWidth, height: pageHeight } = page.getSize();

// Scale image to fit within page with margins
const maxWidth = pageWidth - 100;  // 50pt margin on each side
const maxHeight = pageHeight - 100;
const dims = image.scaleToFit(maxWidth, maxHeight);

// Center horizontally and vertically
const x = (pageWidth - dims.width) / 2;
const y = (pageHeight - dims.height) / 2;

page.drawImage(image, {
  x, y,
  width: dims.width,
  height: dims.height,
});

const pdfBytes = await pdfDoc.save();
```

## Image with Rotation and Opacity

```typescript
import { PDFDocument, degrees } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const image = await pdfDoc.embedJpg(jpgBytes);
const dims = image.scale(0.5);

const page = pdfDoc.addPage();

// Draw rotated and semi-transparent
page.drawImage(image, {
  x: 200,
  y: 300,
  width: dims.width,
  height: dims.height,
  rotate: degrees(30),
  opacity: 0.75,
});

const pdfBytes = await pdfDoc.save();
```

## Watermark Pattern (Image on Existing PDF)

```typescript
import { PDFDocument, degrees } from 'pdf-lib';

const existingPdfBytes: Uint8Array = /* ... */;
const watermarkPngBytes: Uint8Array = /* ... */;

const pdfDoc = await PDFDocument.load(existingPdfBytes);
const watermark = await pdfDoc.embedPng(watermarkPngBytes);

const pages = pdfDoc.getPages();
for (const page of pages) {
  const { width, height } = page.getSize();
  const dims = watermark.scaleToFit(width * 0.5, height * 0.5);

  page.drawImage(watermark, {
    x: (width - dims.width) / 2,
    y: (height - dims.height) / 2,
    width: dims.width,
    height: dims.height,
    opacity: 0.15,
    rotate: degrees(-45),
  });
}

const pdfBytes = await pdfDoc.save();
```

## Multiple Images on One Page

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage([612, 792]); // Letter size

const logo = await pdfDoc.embedPng(logoBytes);
const photo = await pdfDoc.embedJpg(photoBytes);
const chart = await pdfDoc.embedPng(chartBytes);

// Logo in top-right corner
const logoDims = logo.scaleToFit(100, 50);
page.drawImage(logo, {
  x: 612 - logoDims.width - 30,
  y: 792 - logoDims.height - 30,
  width: logoDims.width,
  height: logoDims.height,
});

// Photo centered in middle of page
const photoDims = photo.scaleToFit(400, 250);
page.drawImage(photo, {
  x: (612 - photoDims.width) / 2,
  y: 400,
  width: photoDims.width,
  height: photoDims.height,
});

// Chart at bottom
const chartDims = chart.scaleToFit(500, 200);
page.drawImage(chart, {
  x: (612 - chartDims.width) / 2,
  y: 50,
  width: chartDims.width,
  height: chartDims.height,
});

const pdfBytes = await pdfDoc.save();
```

## Embedding a Page from Another PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const mainPdfBytes: Uint8Array = /* ... */;
const stampPdfBytes: Uint8Array = /* ... */;

const mainDoc = await PDFDocument.load(mainPdfBytes);

// Embed first page of another PDF
const [stampPage] = await mainDoc.embedPdf(stampPdfBytes);

const page = mainDoc.getPage(0);
page.drawPage(stampPage, {
  x: 50,
  y: 50,
  width: 200,
  height: 280,
});

const pdfBytes = await mainDoc.save();
```

## Embedding a Page with Clipping (Bounding Box)

```typescript
import { PDFDocument } from 'pdf-lib';

const sourceDoc = await PDFDocument.load(sourcePdfBytes);
const sourcePage = sourceDoc.getPage(0);

const targetDoc = await PDFDocument.create();

// Embed only a portion of the source page (crop to top-right quadrant)
const embeddedPage = await targetDoc.embedPage(sourcePage, {
  left: 300,
  bottom: 400,
  right: 600,
  top: 800,
});

const page = targetDoc.addPage();
page.drawPage(embeddedPage, { x: 50, y: 50 });

const pdfBytes = await targetDoc.save();
```

## Reusing the Same Image on Multiple Pages

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();

// Embed ONCE — reuse on every page
const headerImage = await pdfDoc.embedPng(headerBytes);
const headerDims = headerImage.scaleToFit(500, 60);

for (let i = 0; i < 5; i++) {
  const page = pdfDoc.addPage();
  const { width, height } = page.getSize();

  // Same image drawn on each page — no extra file size per page
  page.drawImage(headerImage, {
    x: (width - headerDims.width) / 2,
    y: height - headerDims.height - 20,
    width: headerDims.width,
    height: headerDims.height,
  });
}

const pdfBytes = await pdfDoc.save();
```
