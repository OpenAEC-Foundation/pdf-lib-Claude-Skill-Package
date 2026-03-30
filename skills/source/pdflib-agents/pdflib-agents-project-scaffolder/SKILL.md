---
name: pdflib-agents-project-scaffolder
description: >
  Use when scaffolding a new pdf-lib project or adding pdf-lib to an existing
  project. Prevents incomplete setup: ensures TypeScript config, fontkit
  registration, and correct imports are all in place from the start.
  Covers Node.js and browser variants, TypeScript setup, dependency installation.
  Keywords: scaffold, setup, new project, boilerplate, TypeScript, fontkit,
  npm install, create PDF project, start pdf-lib, generate PDF app,
  getting started with pdf-lib.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Project Scaffolder

> Agent skill for generating complete pdf-lib project structures with correct configuration, dependencies, and boilerplate code.

## Quick Reference

### Decision Tree: Which Template?

```
Need a pdf-lib project?
+-- New standalone project?
|   +-- Node.js (server-side, CLI, scripts) -> Node.js Template
|   +-- Browser (client-side, web app) -> Browser Template
+-- Adding pdf-lib to existing project?
|   +-- Has package.json? -> Add Dependencies + Import Pattern
|   +-- No package.json? -> Initialize with Node.js Template
+-- What PDF operations needed?
    +-- Generate new PDFs -> PDF Generation Boilerplate
    +-- Fill existing forms -> Form Filling Boilerplate
    +-- Merge/split PDFs -> Document Merging Boilerplate
    +-- Custom fonts (Unicode) -> Custom Font Setup Boilerplate
```

### Decision Tree: Font Setup

```
Does the project need custom fonts?
+-- Only ASCII/basic Latin text?
|   +-- StandardFonts are sufficient -> NO fontkit needed
+-- Accented characters, Unicode, or non-Latin text?
    +-- ALWAYS install @pdf-lib/fontkit
    +-- ALWAYS call pdfDoc.registerFontkit(fontkit) before embedFont()
    +-- ALWAYS provide TTF/OTF font file with required glyph coverage
```

---

## Node.js Project Template

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

> Add `"@pdf-lib/fontkit": "^1.1.1"` to dependencies ONLY when custom fonts are needed.

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
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### src/index.ts (Basic PDF Generation)

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';
import * as fs from 'fs';

async function generatePdf(): Promise<void> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const page = pdfDoc.addPage(PageSizes.A4);
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
  console.log('PDF created: output.pdf');
}

generatePdf().catch(console.error);
```

### Installation Commands

```bash
npm init -y
npm install pdf-lib
npm install --save-dev typescript @types/node
npx tsc --init
```

---

## Browser Project Template

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PDF Generator</title>
</head>
<body>
  <button id="generate">Generate PDF</button>
  <script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
  <script src="app.js"></script>
</body>
</html>
```

### app.js (Browser UMD)

```javascript
const { PDFDocument, StandardFonts, rgb, PageSizes } = PDFLib;

async function generatePdf() {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const page = pdfDoc.addPage(PageSizes.A4);
  const { height } = page.getSize();

  page.drawText('Hello from pdf-lib!', {
    x: 50,
    y: height - 50,
    size: 24,
    font,
    color: rgb(0, 0, 0),
  });

  const pdfBytes = await pdfDoc.save();
  downloadPdf(pdfBytes, 'output.pdf');
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

document.getElementById('generate').addEventListener('click', generatePdf);
```

### Browser with ES Modules (Bundler)

When using a bundler (Vite, webpack, Rollup), use standard ES imports:

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

async function generatePdf(): Promise<void> {
  const pdfDoc = await PDFDocument.create();
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
  const page = pdfDoc.addPage(PageSizes.A4);
  const { height } = page.getSize();

  page.drawText('Hello from pdf-lib!', {
    x: 50,
    y: height - 50,
    size: 24,
    font,
    color: rgb(0, 0, 0),
  });

  const pdfBytes = await pdfDoc.save();
  const blob = new Blob([pdfBytes], { type: 'application/pdf' });
  const url = URL.createObjectURL(blob);
  window.open(url);
}
```

---

## Custom Font Setup Boilerplate

ALWAYS use this pattern when the project requires Unicode text, accented characters, or any non-ASCII glyphs.

### Dependencies

```bash
npm install pdf-lib @pdf-lib/fontkit
```

### src/index.ts (Custom Font)

```typescript
import { PDFDocument, rgb, PageSizes } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import * as fs from 'fs';

async function generatePdfWithCustomFont(): Promise<void> {
  const pdfDoc = await PDFDocument.create();

  // ALWAYS register fontkit BEFORE embedding custom fonts
  pdfDoc.registerFontkit(fontkit);

  const fontBytes = fs.readFileSync('fonts/MyFont.ttf');
  const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

  const page = pdfDoc.addPage(PageSizes.A4);
  const { height } = page.getSize();

  page.drawText('Unicode text with custom font', {
    x: 50,
    y: height - 50,
    size: 18,
    font: customFont,
    color: rgb(0, 0, 0),
  });

  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync('output.pdf', pdfBytes);
}

generatePdfWithCustomFont().catch(console.error);
```

> ALWAYS pass `{ subset: true }` to `embedFont()` for production builds to reduce file size.

---

## Form Filling Boilerplate

### src/fill-form.ts

```typescript
import { PDFDocument } from 'pdf-lib';
import * as fs from 'fs';

async function fillForm(): Promise<void> {
  const existingPdfBytes = fs.readFileSync('template.pdf');
  const pdfDoc = await PDFDocument.load(existingPdfBytes);
  const form = pdfDoc.getForm();

  // ALWAYS enumerate fields first to discover exact names
  const fields = form.getFields();
  fields.forEach((field) => {
    const name = field.getName();
    const type = field.constructor.name;
    console.log(`Field: "${name}" (${type})`);
  });

  // Fill text fields using exact names from enumeration
  form.getTextField('name').setText('John Doe');
  form.getTextField('email').setText('john@example.com');

  // Check checkboxes
  form.getCheckBox('agree').check();

  // Select dropdown values
  form.getDropdown('country').select('Netherlands');

  // Flatten to make fields non-editable
  form.flatten();

  const pdfBytes = await pdfDoc.save();
  fs.writeFileSync('filled-form.pdf', pdfBytes);
}

fillForm().catch(console.error);
```

> ALWAYS enumerate fields with `form.getFields()` before filling. Field names are case-sensitive and use fully qualified dot-notation (e.g., `"form.personal.name"`).

---

## Document Merging Boilerplate

### src/merge.ts

```typescript
import { PDFDocument } from 'pdf-lib';
import * as fs from 'fs';

async function mergePdfs(inputPaths: string[], outputPath: string): Promise<void> {
  const mergedPdf = await PDFDocument.create();

  for (const inputPath of inputPaths) {
    const pdfBytes = fs.readFileSync(inputPath);
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const pageCount = sourcePdf.getPageCount();
    const indices = Array.from({ length: pageCount }, (_, i) => i);
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  const mergedBytes = await mergedPdf.save();
  fs.writeFileSync(outputPath, mergedBytes);
  console.log(`Merged ${inputPaths.length} PDFs into: ${outputPath}`);
}

mergePdfs(['doc1.pdf', 'doc2.pdf', 'doc3.pdf'], 'merged.pdf').catch(console.error);
```

> ALWAYS use `copyPages()` to transfer pages between documents. NEVER add pages directly from another document — this causes errors.

---

## File Output Patterns

### Node.js: Write to File

```typescript
import * as fs from 'fs';

const pdfBytes: Uint8Array = await pdfDoc.save();
fs.writeFileSync('output.pdf', pdfBytes);
```

### Node.js: Write to Stream

```typescript
import * as fs from 'fs';

const pdfBytes: Uint8Array = await pdfDoc.save();
const stream = fs.createWriteStream('output.pdf');
stream.write(Buffer.from(pdfBytes));
stream.end();
```

### Browser: Download

```typescript
const pdfBytes: Uint8Array = await pdfDoc.save();
const blob = new Blob([pdfBytes], { type: 'application/pdf' });
const url = URL.createObjectURL(blob);
const link = document.createElement('a');
link.href = url;
link.download = 'output.pdf';
link.click();
URL.revokeObjectURL(url);
```

### Browser: Open in New Tab

```typescript
const pdfBytes: Uint8Array = await pdfDoc.save();
const blob = new Blob([pdfBytes], { type: 'application/pdf' });
const url = URL.createObjectURL(blob);
window.open(url);
```

### Browser: Embed as Data URI

```typescript
const dataUri: string = await pdfDoc.saveAsBase64({ dataUri: true });
const iframe = document.createElement('iframe');
iframe.src = dataUri;
document.body.appendChild(iframe);
```

---

## Critical Rules

1. **ALWAYS** use `async/await` — all major pdf-lib operations return Promises
2. **ALWAYS** `await` calls to `PDFDocument.create()`, `PDFDocument.load()`, `embedFont()`, `embedPng()`, `embedJpg()`, `copyPages()`, `save()`, and `saveAsBase64()`
3. **NEVER** `await` synchronous drawing methods: `drawText()`, `drawImage()`, `drawRectangle()`, etc.
4. **NEVER** `await` `embedStandardFont()` — it is the ONLY synchronous embed method
5. **ALWAYS** register fontkit with `pdfDoc.registerFontkit(fontkit)` BEFORE calling `embedFont()` with custom font bytes
6. **NEVER** use StandardFonts for non-ASCII text — they ONLY support WinAnsi encoding
7. **ALWAYS** use `copyPages()` when transferring pages between documents
8. **ALWAYS** pin the pdf-lib CDN version in production browser builds (e.g., `pdf-lib@1.17.1`)
9. **ALWAYS** use `rgb()` with values 0.0-1.0, NEVER 0-255

## Reference Files

- [references/methods.md](references/methods.md) — Setup and configuration API reference
- [references/examples.md](references/examples.md) — Complete runnable project templates
- [references/anti-patterns.md](references/anti-patterns.md) — Common setup mistakes and fixes
