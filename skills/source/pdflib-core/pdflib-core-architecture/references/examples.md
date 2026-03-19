# Working Code Examples — pdf-lib 1.x

All examples are TypeScript with proper imports. Verified against official pdf-lib documentation.

---

## 1. Create a New PDF with Text

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage(PageSizes.A4);
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const { width, height } = page.getSize();

page.drawText('Hello, pdf-lib!', {
  x: 50,
  y: height - 50,
  size: 24,
  font,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

---

## 2. Load and Modify an Existing PDF

```typescript
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib';

const existingPdfBytes: Uint8Array = /* load from file or fetch */;

const pdfDoc = await PDFDocument.load(existingPdfBytes);
const font = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
const pages = pdfDoc.getPages();
const firstPage = pages[0];
const { width, height } = firstPage.getSize();

firstPage.drawText('WATERMARK', {
  x: width / 4,
  y: height / 2,
  size: 60,
  font,
  color: rgb(0.9, 0.1, 0.1),
  rotate: degrees(-45),
  opacity: 0.3,
});

const pdfBytes = await pdfDoc.save();
```

---

## 3. Custom Font with fontkit (Unicode Support)

```typescript
import { PDFDocument, rgb } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

const fontBytes: Uint8Array = /* load TTF/OTF font file */;

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes);

const page = pdfDoc.addPage();
const { height } = page.getSize();

page.drawText('Unicode text with custom font', {
  x: 50,
  y: height - 50,
  size: 20,
  font: customFont,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

---

## 4. Font Subsetting (Smaller File Size)

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

const fontBytes: Uint8Array = /* load font */;

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);

// subset: true includes ONLY glyphs used in the document
const font = await pdfDoc.embedFont(fontBytes, { subset: true });

const page = pdfDoc.addPage();
page.drawText('Only these glyphs are embedded', {
  x: 50,
  y: 700,
  size: 16,
  font,
});

const pdfBytes = await pdfDoc.save();
```

---

## 5. Embed and Draw Images

```typescript
import { PDFDocument } from 'pdf-lib';

const jpgBytes: Uint8Array = /* load JPEG */;
const pngBytes: Uint8Array = /* load PNG */;

const pdfDoc = await PDFDocument.create();
const jpgImage = await pdfDoc.embedJpg(jpgBytes);
const pngImage = await pdfDoc.embedPng(pngBytes);

const page = pdfDoc.addPage();

// Scale to 50% of original size (preserves aspect ratio)
const jpgDims = jpgImage.scale(0.5);
page.drawImage(jpgImage, {
  x: 50,
  y: 500,
  width: jpgDims.width,
  height: jpgDims.height,
});

// Fit within bounds (preserves aspect ratio)
const pngDims = pngImage.scaleToFit(300, 200);
page.drawImage(pngImage, {
  x: 50,
  y: 250,
  width: pngDims.width,
  height: pngDims.height,
});

const pdfBytes = await pdfDoc.save();
```

---

## 6. Center Text Horizontally

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage(PageSizes.Letter);
const font = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

const text = 'Centered Title';
const fontSize = 30;
const textWidth = font.widthOfTextAtSize(text, fontSize);
const pageWidth = page.getWidth();
const pageHeight = page.getHeight();

page.drawText(text, {
  x: (pageWidth - textWidth) / 2,
  y: pageHeight - 100,
  size: fontSize,
  font,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

---

## 7. Multiline Text with Line Height

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const { height } = page.getSize();

page.drawText('Line 1\nLine 2\nLine 3\nLine 4', {
  x: 50,
  y: height - 100,
  size: 14,
  font,
  lineHeight: 20,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

---

## 8. Text Wrapping with maxWidth

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const { height } = page.getSize();

const longText = 'This is a long paragraph that will automatically wrap ' +
  'when it reaches the maximum width boundary specified in the options.';

page.drawText(longText, {
  x: 50,
  y: height - 100,
  size: 14,
  font,
  maxWidth: 400,
  lineHeight: 18,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

---

## 9. Merge Multiple PDFs

```typescript
import { PDFDocument } from 'pdf-lib';

async function mergePdfs(pdfByteArrays: Uint8Array[]): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create();

  for (const pdfBytes of pdfByteArrays) {
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const pageCount = sourcePdf.getPageCount();
    const indices = Array.from({ length: pageCount }, (_, i) => i);
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  return mergedPdf.save();
}
```

---

## 10. Extract Specific Pages

```typescript
import { PDFDocument } from 'pdf-lib';

async function extractPages(
  pdfBytes: Uint8Array,
  pageIndices: number[],
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  const copiedPages = await newPdf.copyPages(sourcePdf, pageIndices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}

// Usage: extract pages 0, 2, and 4
const result = await extractPages(originalBytes, [0, 2, 4]);
```

---

## 11. Set Document Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();

pdfDoc.setTitle('Annual Report 2024', { showInWindowTitleBar: true });
pdfDoc.setAuthor('John Doe');
pdfDoc.setSubject('Financial Overview');
pdfDoc.setKeywords(['finance', 'annual', 'report']);
pdfDoc.setCreator('My Application');
pdfDoc.setProducer('pdf-lib');
pdfDoc.setCreationDate(new Date());
pdfDoc.setModificationDate(new Date());
pdfDoc.setLanguage('en-US');

const page = pdfDoc.addPage();
// ... add content ...

const pdfBytes = await pdfDoc.save();
```

---

## 12. Fill an Existing Form

```typescript
import { PDFDocument } from 'pdf-lib';

const formPdfBytes: Uint8Array = /* load form PDF */;
const pdfDoc = await PDFDocument.load(formPdfBytes);
const form = pdfDoc.getForm();

// Discover field names first
const fields = form.getFields();
fields.forEach((field) => {
  console.log(`${field.constructor.name}: "${field.getName()}"`);
});

// Fill fields by exact name
form.getTextField('name').setText('Jane Smith');
form.getTextField('email').setText('jane@example.com');
form.getCheckBox('agree_terms').check();
form.getDropdown('country').select('Netherlands');

// Flatten to make read-only
form.flatten();

const pdfBytes = await pdfDoc.save();
```

---

## 13. Draw Shapes

```typescript
import { PDFDocument, rgb, degrees } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();

// Filled rectangle with border
page.drawRectangle({
  x: 50,
  y: 600,
  width: 200,
  height: 100,
  color: rgb(0.9, 0.9, 1),
  borderColor: rgb(0, 0, 0.8),
  borderWidth: 2,
});

// Circle
page.drawCircle({
  x: 400,
  y: 650,
  size: 50,
  color: rgb(1, 0.8, 0),
  borderColor: rgb(0.8, 0.5, 0),
  borderWidth: 1.5,
});

// Line
page.drawLine({
  start: { x: 50, y: 500 },
  end: { x: 500, y: 500 },
  thickness: 2,
  color: rgb(0, 0, 0),
});

// SVG path
page.drawSvgPath('M 0,0 L 100,0 L 50,80 Z', {
  x: 200,
  y: 350,
  color: rgb(0, 0.6, 0),
  borderColor: rgb(0, 0.4, 0),
  borderWidth: 1,
});

const pdfBytes = await pdfDoc.save();
```

---

## 14. Embed Page from Another PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const sourcePdfBytes: Uint8Array = /* load source PDF */;

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();

// Embed first page from source as a drawable element
const [embeddedPage] = await pdfDoc.embedPdf(sourcePdfBytes, [0]);

// Draw the embedded page scaled down
page.drawPage(embeddedPage, {
  x: 50,
  y: 400,
  width: 250,
  height: 350,
});

const pdfBytes = await pdfDoc.save();
```

---

## 15. Save as Base64 Data URI (Browser)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

page.drawText('Browser PDF', {
  x: 50,
  y: 700,
  size: 24,
  font,
  color: rgb(0, 0, 0),
});

// Get data URI for embedding in iframe or download
const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });

// Display in iframe
const iframe = document.createElement('iframe');
iframe.src = dataUri;
document.body.appendChild(iframe);
```

---

## 16. Load Encrypted PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const encryptedPdfBytes: Uint8Array = /* load encrypted PDF */;

const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,
});

// Check encryption status
console.log('Is encrypted:', pdfDoc.isEncrypted);

// Modify as normal
const pages = pdfDoc.getPages();
// ... add content ...

const pdfBytes = await pdfDoc.save();
```

---

## 17. Node.js File I/O Pattern

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';
import * as fs from 'fs';

// Read existing PDF
const existingBytes = fs.readFileSync('input.pdf');
const pdfDoc = await PDFDocument.load(existingBytes);

// Modify
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.getPages()[0];
page.drawText('Added by Node.js', {
  x: 50,
  y: 50,
  size: 12,
  font,
  color: rgb(0.5, 0.5, 0.5),
});

// Save to file
const pdfBytes = await pdfDoc.save();
fs.writeFileSync('output.pdf', pdfBytes);
```

---

## 18. Add File Attachment

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();

const csvData = 'name,age\nJohn,30\nJane,25';
const csvBytes = new TextEncoder().encode(csvData);

await pdfDoc.attach(csvBytes, 'data.csv', {
  mimeType: 'text/csv',
  description: 'Exported data file',
  creationDate: new Date(),
  modificationDate: new Date(),
});

const pdfBytes = await pdfDoc.save();
```
