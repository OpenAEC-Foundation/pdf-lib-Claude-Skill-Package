# PDFDocument Examples

## Create a New PDF and Save to File (Node.js)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';
import fs from 'fs';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage();
const { height } = page.getSize();

page.drawText('Hello from pdf-lib!', {
  x: 50,
  y: height - 50,
  size: 24,
  font,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
fs.writeFileSync('output.pdf', pdfBytes);
```

---

## Create Without Auto-Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

// Prevents pdf-lib from setting producer, creator, and dates
const pdfDoc = await PDFDocument.create({ updateMetadata: false });

// Metadata fields are all undefined at this point
console.log(pdfDoc.getProducer());  // undefined
console.log(pdfDoc.getCreator());   // undefined
```

---

## Load Existing PDF from File

```typescript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

const existingPdfBytes = fs.readFileSync('input.pdf');
const pdfDoc = await PDFDocument.load(existingPdfBytes);

console.log(`Pages: ${pdfDoc.getPageCount()}`);
console.log(`Title: ${pdfDoc.getTitle()}`);
```

---

## Load from Base64 String

```typescript
import { PDFDocument } from 'pdf-lib';

const base64Pdf = 'JVBERi0xLjcK...';  // base64-encoded PDF
const pdfDoc = await PDFDocument.load(base64Pdf);
```

---

## Load Preserving Original Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  updateMetadata: false,
});

// Original metadata is preserved
console.log(pdfDoc.getProducer());       // Original producer
console.log(pdfDoc.getCreationDate());   // Original creation date
```

---

## Load Encrypted PDF

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,
});

console.log(`Encrypted: ${pdfDoc.isEncrypted}`);  // true

// You can read/modify the document, but pdf-lib CANNOT:
// - Decrypt password-protected content
// - Encrypt output PDFs with passwords
```

---

## Load with Strict Parsing

```typescript
import { PDFDocument } from 'pdf-lib';

try {
  const pdfDoc = await PDFDocument.load(pdfBytes, {
    throwOnInvalidObject: true,
  });
} catch (error) {
  console.error('PDF contains malformed objects:', error.message);
}
```

---

## Load with All Options

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
  throwOnInvalidObject: false,
  updateMetadata: false,
  capNumbers: true,
});
```

---

## Save with Default Options

```typescript
const pdfBytes = await pdfDoc.save();
// Returns Uint8Array with object streams enabled, default page added if empty,
// 50 objects per tick, and field appearances updated
```

---

## Save Without Object Streams (Maximum Compatibility)

```typescript
const pdfBytes = await pdfDoc.save({
  useObjectStreams: false,
});
// Larger file size but works with PDF readers that only support pre-1.5 format
```

---

## Save Skipping Form Field Appearance Updates

```typescript
const pdfBytes = await pdfDoc.save({
  updateFieldAppearances: false,
});
// Faster save when form fields have not been modified
```

---

## Save Without Adding Default Empty Page

```typescript
const pdfDoc = await PDFDocument.create();
// Document has 0 pages

const pdfBytes = await pdfDoc.save({
  addDefaultPage: false,
});
// Saves a PDF with 0 pages (some viewers may not handle this well)
```

---

## Save with Custom Objects Per Tick

```typescript
const pdfBytes = await pdfDoc.save({
  objectsPerTick: 100,
});
// Processes 100 objects per event loop tick instead of default 50
// Higher values = faster save but longer main thread blocks
```

---

## Save as Base64

```typescript
const base64String = await pdfDoc.saveAsBase64();
// Returns plain base64 string like "JVBERi0xLjcK..."
```

---

## Save as Data URI for HTML Embedding

```typescript
const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });
// Returns "data:application/pdf;base64,JVBERi0xLjcK..."

// Use in an iframe
document.getElementById('pdf-viewer').src = dataUri;

// Use in an anchor tag for download
const link = document.createElement('a');
link.href = dataUri;
link.download = 'document.pdf';
link.click();
```

---

## Set Complete Document Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();

pdfDoc.setTitle('Q4 Financial Report', { showInWindowTitleBar: true });
pdfDoc.setAuthor('Finance Team');
pdfDoc.setSubject('Quarterly financial overview and projections');
pdfDoc.setKeywords(['finance', 'quarterly', 'report', '2024']);
pdfDoc.setCreator('ReportGenerator v3.1');
pdfDoc.setProducer('pdf-lib');
pdfDoc.setCreationDate(new Date('2024-12-01T00:00:00Z'));
pdfDoc.setModificationDate(new Date());
pdfDoc.setLanguage('en-US');
```

---

## Read Document Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(pdfBytes, { updateMetadata: false });

const title = pdfDoc.getTitle();               // string | undefined
const author = pdfDoc.getAuthor();             // string | undefined
const subject = pdfDoc.getSubject();           // string | undefined
const keywords = pdfDoc.getKeywords();         // string | undefined (comma-separated!)
const creator = pdfDoc.getCreator();           // string | undefined
const producer = pdfDoc.getProducer();         // string | undefined
const creationDate = pdfDoc.getCreationDate(); // Date | undefined
const modDate = pdfDoc.getModificationDate();  // Date | undefined

console.log(`Title: ${title}`);
console.log(`Author: ${author}`);
console.log(`Keywords: ${keywords}`);  // "finance,quarterly,report,2024"
```

---

## Keywords Setter/Getter Asymmetry Handling

```typescript
// Setting keywords (accepts string[])
pdfDoc.setKeywords(['pdf', 'generation', 'typescript']);

// Getting keywords (returns string, NOT string[])
const keywordsString = pdfDoc.getKeywords();  // "pdf,generation,typescript"

// To get back an array, split manually:
const keywordsArray = keywordsString?.split(',').map(k => k.trim()) ?? [];
```

---

## Set Title with Window Title Bar Display

```typescript
pdfDoc.setTitle('My Important Document', {
  showInWindowTitleBar: true,
});
// PDF viewers will show "My Important Document" in the title bar
// instead of the filename
```

---

## Set Document Language

```typescript
// RFC 3066 language tags
pdfDoc.setLanguage('en-US');    // American English
pdfDoc.setLanguage('en-GB');    // British English
pdfDoc.setLanguage('nl-NL');    // Dutch
pdfDoc.setLanguage('de-DE');    // German
pdfDoc.setLanguage('fr-FR');    // French
pdfDoc.setLanguage('ja');       // Japanese
pdfDoc.setLanguage('zh-CN');    // Simplified Chinese
```

---

## Attach a File to PDF

```typescript
import { PDFDocument } from 'pdf-lib';
import fs from 'fs';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
page.drawText('See attached CSV for raw data.', { x: 50, y: 700, size: 14 });

const csvContent = 'Name,Value\nAlpha,100\nBeta,200';
const csvBytes = new TextEncoder().encode(csvContent);

await pdfDoc.attach(csvBytes, 'data.csv', {
  mimeType: 'text/csv',
  description: 'Raw data export from the dashboard',
  creationDate: new Date(),
  modificationDate: new Date(),
});

const pdfBytes = await pdfDoc.save();
fs.writeFileSync('report-with-attachment.pdf', pdfBytes);
```

---

## Attach Multiple Files

```typescript
const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();

// Attach a JSON config file
await pdfDoc.attach(jsonBytes, 'config.json', {
  mimeType: 'application/json',
  description: 'Application configuration',
});

// Attach an image
await pdfDoc.attach(imageBytes, 'logo.png', {
  mimeType: 'image/png',
  description: 'Company logo',
});

// Attach a text log
await pdfDoc.attach(logBytes, 'build.log', {
  mimeType: 'text/plain',
  description: 'Build log output',
});

const pdfBytes = await pdfDoc.save();
```

---

## Add JavaScript Action

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();

// Alert on document open (Adobe Acrobat only)
pdfDoc.addJavaScript('showWelcome',
  'app.alert("Welcome! This document contains important information.");'
);

// Print dialog on open
pdfDoc.addJavaScript('autoPrint',
  'this.print({bUI: true, bSilent: false, bShrinkToFit: true});'
);

const pdfBytes = await pdfDoc.save();
```

---

## Copy Document (Shallow)

```typescript
import { PDFDocument } from 'pdf-lib';

const originalDoc = await PDFDocument.load(pdfBytes);
const copiedDoc = await originalDoc.copy();

// copiedDoc has the same pages but:
// - NO form fields (AcroForm)
// - NO outlines (bookmarks)
// - NO other catalog-level structures

const copiedBytes = await copiedDoc.save();
```

---

## Check Document Properties

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(pdfBytes);

// Check encryption
console.log(`Encrypted: ${pdfDoc.isEncrypted}`);

// Access low-level properties
console.log(`Default word breaks: ${pdfDoc.defaultWordBreaks}`);

// Access catalog (advanced usage)
const catalog = pdfDoc.catalog;

// Access context (advanced usage)
const context = pdfDoc.context;
```

---

## Complete End-to-End: Create, Populate, Metadata, Attach, Save

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';
import fs from 'fs';

async function createCompleteDocument(): Promise<Uint8Array> {
  // 1. Create document
  const pdfDoc = await PDFDocument.create();

  // 2. Set metadata
  pdfDoc.setTitle('Invoice #2024-001', { showInWindowTitleBar: true });
  pdfDoc.setAuthor('Billing System');
  pdfDoc.setSubject('Monthly invoice');
  pdfDoc.setKeywords(['invoice', 'billing', '2024']);
  pdfDoc.setCreator('InvoiceApp v1.0');
  pdfDoc.setLanguage('en-US');

  // 3. Add content
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);
  const page = pdfDoc.addPage();
  const { width, height } = page.getSize();

  page.drawText('INVOICE', {
    x: 50,
    y: height - 50,
    size: 30,
    font: boldFont,
    color: rgb(0, 0, 0),
  });

  page.drawText('Invoice #2024-001\nDate: 2024-12-01\nTotal: $1,250.00', {
    x: 50,
    y: height - 100,
    size: 12,
    font,
    lineHeight: 18,
    color: rgb(0.2, 0.2, 0.2),
  });

  // 4. Attach supporting data
  const csvData = 'Item,Qty,Price\nWidget A,5,100.00\nWidget B,3,250.00';
  const csvBytes = new TextEncoder().encode(csvData);

  await pdfDoc.attach(csvBytes, 'line-items.csv', {
    mimeType: 'text/csv',
    description: 'Invoice line items',
    creationDate: new Date(),
    modificationDate: new Date(),
  });

  // 5. Save
  return await pdfDoc.save();
}

const pdfBytes = await createCompleteDocument();
fs.writeFileSync('invoice.pdf', pdfBytes);
```
