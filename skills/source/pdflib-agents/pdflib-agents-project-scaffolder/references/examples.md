# Complete Project Templates

## Template 1: Node.js PDF Generator (Minimal)

### Directory Structure

```
my-pdf-generator/
├── package.json
├── tsconfig.json
└── src/
    └── index.ts
```

### package.json

```json
{
  "name": "my-pdf-generator",
  "version": "1.0.0",
  "description": "PDF generation with pdf-lib",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc && node dist/index.js"
  },
  "dependencies": {
    "pdf-lib": "^1.17.1"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### src/index.ts

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';
import * as fs from 'fs';
import * as path from 'path';

async function main(): Promise<void> {
  const pdfDoc = await PDFDocument.create();

  pdfDoc.setTitle('Sample Document');
  pdfDoc.setAuthor('My Application');
  pdfDoc.setCreator('pdf-lib');

  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  // Title
  const title = 'Sample PDF Document';
  const titleSize = 28;
  const titleWidth = boldFont.widthOfTextAtSize(title, titleSize);
  page.drawText(title, {
    x: (width - titleWidth) / 2,
    y: height - 72,
    size: titleSize,
    font: boldFont,
    color: rgb(0, 0.2, 0.4),
  });

  // Body text
  page.drawText('This document was generated with pdf-lib.', {
    x: 72,
    y: height - 120,
    size: 14,
    font,
    color: rgb(0, 0, 0),
  });

  // Horizontal rule
  page.drawLine({
    start: { x: 72, y: height - 140 },
    end: { x: width - 72, y: height - 140 },
    thickness: 1,
    color: rgb(0.7, 0.7, 0.7),
  });

  // Multiline content
  const bodyText = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit.\nSed do eiusmod tempor incididunt ut labore et dolore magna aliqua.\nUt enim ad minim veniam, quis nostrud exercitation.';
  page.drawText(bodyText, {
    x: 72,
    y: height - 180,
    size: 12,
    font,
    color: rgb(0.2, 0.2, 0.2),
    lineHeight: 18,
  });

  const outputPath = path.join(process.cwd(), 'output.pdf');
  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync(outputPath, pdfBytes);
  console.log(`PDF created: ${outputPath}`);
}

main().catch(console.error);
```

### Setup Commands

```bash
mkdir my-pdf-generator && cd my-pdf-generator
npm init -y
npm install pdf-lib
npm install --save-dev typescript @types/node
mkdir src
# Create tsconfig.json and src/index.ts
npm run dev
```

---

## Template 2: Node.js with Custom Fonts (Unicode Support)

### Directory Structure

```
my-unicode-pdf/
├── package.json
├── tsconfig.json
├── fonts/
│   └── NotoSans-Regular.ttf
└── src/
    └── index.ts
```

### package.json

```json
{
  "name": "my-unicode-pdf",
  "version": "1.0.0",
  "description": "PDF generation with custom Unicode fonts",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "tsc && node dist/index.js"
  },
  "dependencies": {
    "pdf-lib": "^1.17.1",
    "@pdf-lib/fontkit": "^1.1.1"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### src/index.ts

```typescript
import { PDFDocument, rgb, PageSizes } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import * as fs from 'fs';
import * as path from 'path';

async function main(): Promise<void> {
  const pdfDoc = await PDFDocument.create();

  // ALWAYS register fontkit BEFORE embedding custom fonts
  pdfDoc.registerFontkit(fontkit);

  const fontPath = path.join(process.cwd(), 'fonts', 'NotoSans-Regular.ttf');
  const fontBytes = fs.readFileSync(fontPath);

  // ALWAYS use subset: true for production to reduce file size
  const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

  const page = pdfDoc.addPage(PageSizes.A4);
  const { height } = page.getSize();

  // Custom font supports full Unicode
  page.drawText('Hello World - English', {
    x: 50,
    y: height - 50,
    size: 18,
    font: customFont,
    color: rgb(0, 0, 0),
  });

  page.drawText('Accented: cafe, naive, resume', {
    x: 50,
    y: height - 80,
    size: 14,
    font: customFont,
    color: rgb(0.3, 0.3, 0.3),
  });

  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync('output.pdf', pdfBytes);
  console.log('PDF with custom font created: output.pdf');
}

main().catch(console.error);
```

---

## Template 3: Form Filling Project

### Directory Structure

```
my-form-filler/
├── package.json
├── tsconfig.json
├── templates/
│   └── form-template.pdf
└── src/
    └── fill-form.ts
```

### package.json

```json
{
  "name": "my-form-filler",
  "version": "1.0.0",
  "description": "PDF form filling with pdf-lib",
  "main": "dist/fill-form.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/fill-form.js",
    "dev": "tsc && node dist/fill-form.js"
  },
  "dependencies": {
    "pdf-lib": "^1.17.1"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### src/fill-form.ts

```typescript
import { PDFDocument } from 'pdf-lib';
import * as fs from 'fs';
import * as path from 'path';

interface FormData {
  [fieldName: string]: string | boolean;
}

async function discoverFields(pdfPath: string): Promise<void> {
  const pdfBytes = fs.readFileSync(pdfPath);
  const pdfDoc = await PDFDocument.load(pdfBytes);
  const form = pdfDoc.getForm();
  const fields = form.getFields();

  console.log(`Found ${fields.length} form fields:`);
  fields.forEach((field) => {
    const name = field.getName();
    const type = field.constructor.name;
    console.log(`  "${name}" (${type})`);
  });
}

async function fillForm(
  templatePath: string,
  outputPath: string,
  data: FormData,
): Promise<void> {
  const pdfBytes = fs.readFileSync(templatePath);
  const pdfDoc = await PDFDocument.load(pdfBytes);
  const form = pdfDoc.getForm();

  for (const [fieldName, value] of Object.entries(data)) {
    if (typeof value === 'string') {
      form.getTextField(fieldName).setText(value);
    } else if (typeof value === 'boolean') {
      const checkbox = form.getCheckBox(fieldName);
      if (value) {
        checkbox.check();
      } else {
        checkbox.uncheck();
      }
    }
  }

  // Flatten to make fields non-editable
  form.flatten();

  const filledBytes = await pdfDoc.save();
  fs.writeFileSync(outputPath, filledBytes);
  console.log(`Filled form saved: ${outputPath}`);
}

async function main(): Promise<void> {
  const templatePath = path.join(process.cwd(), 'templates', 'form-template.pdf');

  // Step 1: Discover field names
  await discoverFields(templatePath);

  // Step 2: Fill the form with data
  const formData: FormData = {
    'name': 'John Doe',
    'email': 'john@example.com',
    'date': '2024-01-15',
    'agree_terms': true,
  };

  await fillForm(templatePath, 'filled-output.pdf', formData);
}

main().catch(console.error);
```

---

## Template 4: Document Merger

### Directory Structure

```
my-pdf-merger/
├── package.json
├── tsconfig.json
├── input/
│   ├── doc1.pdf
│   ├── doc2.pdf
│   └── doc3.pdf
└── src/
    └── merge.ts
```

### src/merge.ts

```typescript
import { PDFDocument } from 'pdf-lib';
import * as fs from 'fs';
import * as path from 'path';

async function mergePdfs(
  inputPaths: string[],
  outputPath: string,
): Promise<void> {
  const mergedPdf = await PDFDocument.create();

  for (const inputPath of inputPaths) {
    const pdfBytes = fs.readFileSync(inputPath);
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const pageCount = sourcePdf.getPageCount();
    const indices = Array.from({ length: pageCount }, (_, i) => i);

    // ALWAYS use copyPages() — NEVER add pages directly from another document
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  mergedPdf.setTitle('Merged Document');
  mergedPdf.setCreator('pdf-lib merger');

  const mergedBytes = await mergedPdf.save();
  fs.writeFileSync(outputPath, mergedBytes);
  console.log(
    `Merged ${inputPaths.length} PDFs (${mergedPdf.getPageCount()} pages) into: ${outputPath}`,
  );
}

async function splitPdf(
  inputPath: string,
  outputDir: string,
): Promise<void> {
  const pdfBytes = fs.readFileSync(inputPath);
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();

  if (!fs.existsSync(outputDir)) {
    fs.mkdirSync(outputDir, { recursive: true });
  }

  for (let i = 0; i < pageCount; i++) {
    const newPdf = await PDFDocument.create();
    const [page] = await newPdf.copyPages(sourcePdf, [i]);
    newPdf.addPage(page);

    const outputPath = path.join(outputDir, `page-${i + 1}.pdf`);
    const pageBytes = await newPdf.save();
    fs.writeFileSync(outputPath, pageBytes);
  }

  console.log(`Split ${pageCount} pages into: ${outputDir}`);
}

async function main(): Promise<void> {
  const inputDir = path.join(process.cwd(), 'input');
  const files = fs.readdirSync(inputDir)
    .filter((f) => f.endsWith('.pdf'))
    .map((f) => path.join(inputDir, f))
    .sort();

  if (files.length === 0) {
    console.error('No PDF files found in input/ directory');
    process.exit(1);
  }

  // Merge all PDFs
  await mergePdfs(files, 'merged-output.pdf');

  // Split a PDF into individual pages
  // await splitPdf('merged-output.pdf', 'output-pages');
}

main().catch(console.error);
```

---

## Template 5: Browser App (Vanilla HTML + JS)

### Directory Structure

```
my-browser-pdf/
├── index.html
└── app.js
```

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PDF Generator</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 600px; margin: 40px auto; padding: 0 20px; }
    button { padding: 12px 24px; font-size: 16px; cursor: pointer; margin: 8px 4px; }
    input[type="file"] { margin: 8px 0; display: block; }
    #status { margin-top: 16px; color: #555; }
  </style>
</head>
<body>
  <h1>PDF Generator</h1>

  <h2>Create New PDF</h2>
  <button id="btn-create">Generate Sample PDF</button>

  <h2>Fill Existing Form</h2>
  <input type="file" id="file-input" accept=".pdf" />
  <button id="btn-fill" disabled>Fill Form</button>

  <h2>Merge PDFs</h2>
  <input type="file" id="merge-input" accept=".pdf" multiple />
  <button id="btn-merge" disabled>Merge Selected PDFs</button>

  <p id="status"></p>

  <!-- ALWAYS pin version in production -->
  <script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

### app.js

```javascript
const { PDFDocument, StandardFonts, rgb, PageSizes } = PDFLib;

function setStatus(message) {
  document.getElementById('status').textContent = message;
}

function downloadPdf(pdfBytes, filename) {
  const blob = new Blob([pdfBytes], { type: 'application/pdf' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = filename;
  link.click();
  URL.revokeObjectURL(url);
}

// --- Create New PDF ---
async function createPdf() {
  setStatus('Generating PDF...');
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

  const page = pdfDoc.addPage(PageSizes.A4);
  const { width, height } = page.getSize();

  page.drawText('Browser-Generated PDF', {
    x: 50,
    y: height - 60,
    size: 28,
    font: boldFont,
    color: rgb(0, 0.2, 0.5),
  });

  page.drawText('Created with pdf-lib in the browser.', {
    x: 50,
    y: height - 100,
    size: 14,
    font,
    color: rgb(0.2, 0.2, 0.2),
  });

  page.drawText(new Date().toISOString(), {
    x: 50,
    y: height - 130,
    size: 10,
    font,
    color: rgb(0.5, 0.5, 0.5),
  });

  const pdfBytes = await pdfDoc.save();
  downloadPdf(pdfBytes, 'generated.pdf');
  setStatus('PDF downloaded!');
}

// --- Fill Form ---
async function fillForm(file) {
  setStatus('Filling form...');
  const arrayBuffer = await file.arrayBuffer();
  const pdfDoc = await PDFDocument.load(arrayBuffer);
  const form = pdfDoc.getForm();

  const fields = form.getFields();
  let filledCount = 0;

  fields.forEach((field) => {
    const name = field.getName();
    const type = field.constructor.name;
    console.log(`Field: "${name}" (${type})`);

    if (type === 'PDFTextField') {
      form.getTextField(name).setText(`Sample: ${name}`);
      filledCount++;
    }
  });

  form.flatten();

  const pdfBytes = await pdfDoc.save();
  downloadPdf(pdfBytes, 'filled-form.pdf');
  setStatus(`Filled ${filledCount} text fields and downloaded!`);
}

// --- Merge PDFs ---
async function mergePdfs(files) {
  setStatus('Merging PDFs...');
  const mergedPdf = await PDFDocument.create();

  for (const file of files) {
    const arrayBuffer = await file.arrayBuffer();
    const sourcePdf = await PDFDocument.load(arrayBuffer);
    const pageCount = sourcePdf.getPageCount();
    const indices = Array.from({ length: pageCount }, (_, i) => i);
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  const pdfBytes = await mergedPdf.save();
  downloadPdf(pdfBytes, 'merged.pdf');
  setStatus(`Merged ${files.length} PDFs (${mergedPdf.getPageCount()} pages)!`);
}

// --- Event Listeners ---
document.getElementById('btn-create').addEventListener('click', createPdf);

const fileInput = document.getElementById('file-input');
const btnFill = document.getElementById('btn-fill');
fileInput.addEventListener('change', () => {
  btnFill.disabled = !fileInput.files.length;
});
btnFill.addEventListener('click', () => {
  if (fileInput.files.length) fillForm(fileInput.files[0]);
});

const mergeInput = document.getElementById('merge-input');
const btnMerge = document.getElementById('btn-merge');
mergeInput.addEventListener('change', () => {
  btnMerge.disabled = mergeInput.files.length < 2;
});
btnMerge.addEventListener('click', () => {
  if (mergeInput.files.length >= 2) mergePdfs(Array.from(mergeInput.files));
});
```

---

## Template 6: Adding pdf-lib to an Existing Project

When adding pdf-lib to an existing Node.js/TypeScript project, follow these steps:

### Step 1: Install Dependencies

```bash
npm install pdf-lib
# Only if custom fonts are needed:
npm install @pdf-lib/fontkit
```

### Step 2: Verify tsconfig.json Compatibility

Ensure these settings are present (add if missing):

```json
{
  "compilerOptions": {
    "esModuleInterop": true,
    "skipLibCheck": true,
    "lib": ["ES2020"]
  }
}
```

- `esModuleInterop: true` is REQUIRED for `import fontkit from '@pdf-lib/fontkit'`
- `skipLibCheck: true` prevents type conflicts with pdf-lib's bundled types
- `lib` must include `ES2020` or later for `Uint8Array` and `ArrayBuffer` support

### Step 3: Create a PDF Utility Module

```typescript
// src/utils/pdf-generator.ts
import { PDFDocument, StandardFonts, rgb, PDFFont, PDFPage } from 'pdf-lib';

export async function createDocument(): Promise<PDFDocument> {
  return await PDFDocument.create();
}

export async function addTextPage(
  doc: PDFDocument,
  text: string,
  options?: { fontSize?: number; fontName?: StandardFonts },
): Promise<PDFPage> {
  const font = await doc.embedFont(options?.fontName ?? StandardFonts.Helvetica);
  const page = doc.addPage();
  const { height } = page.getSize();

  page.drawText(text, {
    x: 50,
    y: height - 50,
    size: options?.fontSize ?? 14,
    font,
    color: rgb(0, 0, 0),
  });

  return page;
}

export async function saveToBytes(doc: PDFDocument): Promise<Uint8Array> {
  return await doc.save();
}
```
