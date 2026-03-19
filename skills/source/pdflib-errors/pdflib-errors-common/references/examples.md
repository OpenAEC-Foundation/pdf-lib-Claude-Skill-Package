# Error Reproduction & Fix Examples

Complete runnable examples demonstrating each common error and its fix.

---

## Example 1: Forgotten `await` — Silent Failure

### Broken Code
```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function createBrokenPdf(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // BUG: Missing await — font is a Promise<PDFFont>, not PDFFont
  const font = pdfDoc.embedFont(StandardFonts.Helvetica);
  page.drawText('Hello World', {
    x: 50,
    y: 500,
    font,  // Passing a Promise — will throw or silently fail
    size: 24,
  });

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function createFixedPdf(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  const font = await pdfDoc.embedFont(StandardFonts.Helvetica); // FIXED: added await
  page.drawText('Hello World', {
    x: 50,
    y: 500,
    font,
    size: 24,
  });

  return await pdfDoc.save();
}
```

---

## Example 2: Coordinate System — Text at Wrong Position

### Broken Code
```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function brokenCoordinates(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage([595.28, 841.89]); // A4
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

  // BUG: Developer expects this near the top, but y=50 is near the BOTTOM
  page.drawText('This should be at the top', {
    x: 50,
    y: 50,
    font,
    size: 18,
  });

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

async function fixedCoordinates(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage([595.28, 841.89]); // A4
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const { height } = page.getSize();

  // FIXED: Subtract from height to position from top
  page.drawText('This should be at the top', {
    x: 50,
    y: height - 50,
    font,
    size: 18,
  });

  return await pdfDoc.save();
}
```

---

## Example 3: Cross-Document Page Copy

### Broken Code
```typescript
import { PDFDocument } from 'pdf-lib';

async function brokenMerge(pdfABytes: Uint8Array, pdfBBytes: Uint8Array): Promise<Uint8Array> {
  const pdfA = await PDFDocument.load(pdfABytes);
  const pdfB = await PDFDocument.load(pdfBBytes);
  const merged = await PDFDocument.create();

  // BUG: Cannot add page from another document directly
  const page = pdfA.getPage(0);
  merged.addPage(page); // THROWS ERROR

  return await merged.save();
}
```

### Fixed Code
```typescript
import { PDFDocument } from 'pdf-lib';

async function fixedMerge(pdfABytes: Uint8Array, pdfBBytes: Uint8Array): Promise<Uint8Array> {
  const pdfA = await PDFDocument.load(pdfABytes);
  const pdfB = await PDFDocument.load(pdfBBytes);
  const merged = await PDFDocument.create();

  // FIXED: Use copyPages on the TARGET document, then addPage
  const [pageFromA] = await merged.copyPages(pdfA, [0]);
  merged.addPage(pageFromA);

  const [pageFromB] = await merged.copyPages(pdfB, [0]);
  merged.addPage(pageFromB);

  return await merged.save();
}
```

---

## Example 4: Form Field Name Discovery

### Broken Code
```typescript
import { PDFDocument } from 'pdf-lib';

async function brokenFormFill(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes);
  const form = pdfDoc.getForm();

  // BUG: Guessing field name — case-sensitive, may need full path
  form.getTextField('name').setText('John Doe');        // May throw
  form.getTextField('email').setText('john@example.com'); // May throw

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument } from 'pdf-lib';

async function fixedFormFill(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes);
  const form = pdfDoc.getForm();

  // FIXED: Enumerate fields first to discover exact names
  const fields = form.getFields();
  for (const field of fields) {
    console.log(`Type: ${field.constructor.name}, Name: "${field.getName()}"`);
  }
  // Output might reveal: "form.personal.Name", "form.contact.Email"

  // Use the EXACT names discovered from enumeration
  form.getTextField('form.personal.Name').setText('John Doe');
  form.getTextField('form.contact.Email').setText('john@example.com');

  return await pdfDoc.save();
}
```

---

## Example 5: Color Values — 0-255 vs 0-1

### Broken Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function brokenColors(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // BUG: Using 0-255 range instead of 0-1
  page.drawRectangle({
    x: 50,
    y: 400,
    width: 200,
    height: 100,
    color: rgb(255, 0, 0),       // Creates WHITE, not red
    borderColor: rgb(0, 0, 255), // Creates BLACK, not blue
    borderWidth: 2,
  });

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function fixedColors(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // FIXED: Using 0.0-1.0 range
  page.drawRectangle({
    x: 50,
    y: 400,
    width: 200,
    height: 100,
    color: rgb(1, 0, 0),       // Correct red
    borderColor: rgb(0, 0, 1), // Correct blue
    borderWidth: 2,
  });

  return await pdfDoc.save();
}
```

---

## Example 6: drawEllipse — Wrong Parameter Names

### Broken Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function brokenEllipse(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // BUG: Using width/height — these are IGNORED by drawEllipse
  page.drawEllipse({
    x: 200,
    y: 400,
    width: 150,   // IGNORED — not a valid parameter
    height: 75,   // IGNORED — not a valid parameter
    color: rgb(0, 0, 1),
  });

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function fixedEllipse(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // FIXED: Using xScale/yScale
  page.drawEllipse({
    x: 200,
    y: 400,
    xScale: 150,  // Horizontal radius
    yScale: 75,   // Vertical radius
    color: rgb(0, 0, 1),
  });

  return await pdfDoc.save();
}
```

---

## Example 7: drawLine — Wrong Parameter Name

### Broken Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function brokenLine(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // BUG: Using borderWidth — drawLine uses thickness
  page.drawLine({
    start: { x: 50, y: 500 },
    end: { x: 500, y: 500 },
    borderWidth: 3, // IGNORED
    color: rgb(0, 0, 0),
  });

  return await pdfDoc.save();
}
```

### Fixed Code
```typescript
import { PDFDocument, rgb } from 'pdf-lib';

async function fixedLine(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();
  const page = pdfDoc.addPage();

  // FIXED: Using thickness
  page.drawLine({
    start: { x: 50, y: 500 },
    end: { x: 500, y: 500 },
    thickness: 3, // Correct parameter name
    color: rgb(0, 0, 0),
  });

  return await pdfDoc.save();
}
```

---

## Example 8: save() Side Effects

### Preserving Field Appearances
```typescript
import { PDFDocument } from 'pdf-lib';

async function saveWithoutAppearanceUpdate(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes);
  const form = pdfDoc.getForm();

  form.getTextField('name').setText('John Doe');

  // Prevent save() from auto-updating field appearances
  return await pdfDoc.save({ updateFieldAppearances: false });
}
```

### Avoiding Auto-Added Blank Page
```typescript
import { PDFDocument } from 'pdf-lib';

async function safeEmptyDocument(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create();

  // ALWAYS add at least one page before saving
  pdfDoc.addPage();

  // Or disable auto-page: save({ addDefaultPage: false })
  // But this may produce an invalid PDF
  return await pdfDoc.save();
}
```

---

## Example 9: Keywords Asymmetry

```typescript
import { PDFDocument } from 'pdf-lib';

async function keywordsExample(): Promise<void> {
  const pdfDoc = await PDFDocument.create();

  // Set takes string[]
  pdfDoc.setKeywords(['invoice', 'financial', 'Q4']);

  // Get returns string (comma-separated), NOT string[]
  const raw: string | undefined = pdfDoc.getKeywords();
  console.log(raw);  // "invoice,financial,Q4"

  // Parse back to array
  const keywords: string[] = raw?.split(',') ?? [];
  console.log(keywords);  // ['invoice', 'financial', 'Q4']
}
```
