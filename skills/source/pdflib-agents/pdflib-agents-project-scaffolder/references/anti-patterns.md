# Common Setup Mistakes and Fixes

## AP-001: Missing fontkit Registration

**Symptom:** Error when calling `embedFont()` with custom font bytes (TTF/OTF).

**Broken:**
```typescript
const pdfDoc = await PDFDocument.create();
// MISSING: pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes); // CRASHES
```

**Fix:** ALWAYS register fontkit BEFORE embedding custom fonts.
```typescript
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);                  // MUST come first
const font = await pdfDoc.embedFont(fontBytes);   // Now works
```

---

## AP-002: Using StandardFonts for Non-ASCII Text

**Symptom:** `"WinAnsi cannot encode "X" (0xNNNN)"` error.

**Broken:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Text with special characters', { font }); // CRASHES on non-WinAnsi chars
```

**Fix:** ALWAYS use a custom font with fontkit for any non-ASCII text.
```typescript
import fontkit from '@pdf-lib/fontkit';

pdfDoc.registerFontkit(fontkit);
const font = await pdfDoc.embedFont(fontBytes); // TTF/OTF with required glyph coverage
page.drawText('Text with special characters', { font }); // Works
```

---

## AP-003: Forgetting to Await Async Methods

**Symptom:** `font` is a Promise object, not a PDFFont. Drawing operations fail silently or produce corrupt PDFs.

**Broken:**
```typescript
const font = pdfDoc.embedFont(StandardFonts.Helvetica); // Returns Promise!
page.drawText('Hello', { font }); // font is a Promise, not PDFFont
```

**Fix:** ALWAYS `await` async methods.
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font }); // font is a PDFFont
```

**All async methods that REQUIRE `await`:**
- `PDFDocument.create()`
- `PDFDocument.load()`
- `pdfDoc.embedFont()` (NOT `embedStandardFont()`)
- `pdfDoc.embedPng()`
- `pdfDoc.embedJpg()`
- `pdfDoc.copyPages()`
- `pdfDoc.embedPdf()` / `pdfDoc.embedPage()` / `pdfDoc.embedPages()`
- `pdfDoc.save()`
- `pdfDoc.saveAsBase64()`
- `pdfDoc.attach()`
- `pdfDoc.flush()`
- `pdfDoc.copy()`

**The ONE sync embed method (NEVER await):**
- `pdfDoc.embedStandardFont()` — returns `PDFFont` directly

---

## AP-004: Missing esModuleInterop in tsconfig.json

**Symptom:** `import fontkit from '@pdf-lib/fontkit'` fails with TypeScript error about default imports.

**Broken tsconfig.json:**
```json
{
  "compilerOptions": {
    "esModuleInterop": false
  }
}
```

**Fix:** ALWAYS enable `esModuleInterop`.
```json
{
  "compilerOptions": {
    "esModuleInterop": true
  }
}
```

Alternative (if `esModuleInterop` cannot be enabled):
```typescript
import * as fontkit from '@pdf-lib/fontkit';
```

---

## AP-005: Wrong Color Value Range

**Symptom:** Colors appear white or incorrect. `rgb(255, 0, 0)` does NOT produce red.

**Broken:**
```typescript
const red = rgb(255, 0, 0);   // Values overflow — NOT red
const blue = rgb(0, 0, 255);  // Values overflow — NOT blue
```

**Fix:** ALWAYS use values between 0.0 and 1.0.
```typescript
const red = rgb(1, 0, 0);
const blue = rgb(0, 0, 1);
const gray = rgb(0.5, 0.5, 0.5);
const teal = rgb(0, 0.53, 0.71);
```

---

## AP-006: Coordinate System Confusion

**Symptom:** Text appears at the bottom of the page when intended for the top. Content is positioned upside-down relative to expectations.

**Broken (assumes top-left origin like HTML):**
```typescript
page.drawText('Title', { x: 50, y: 50 }); // Appears near BOTTOM, not top
```

**Fix:** PDF origin is BOTTOM-LEFT. ALWAYS calculate from page height for top positioning.
```typescript
const { height } = page.getSize();
page.drawText('Title', { x: 50, y: height - 50 }); // Near TOP of page
page.drawText('Footer', { x: 50, y: 30 });          // Near BOTTOM of page
```

---

## AP-007: Not Pinning CDN Version in Production

**Symptom:** Application breaks when CDN serves a new version with breaking changes.

**Broken:**
```html
<script src="https://unpkg.com/pdf-lib/dist/pdf-lib.min.js"></script>
```

**Fix:** ALWAYS pin the exact version in production.
```html
<script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
```

---

## AP-008: Adding Pages Directly Between Documents

**Symptom:** Error when trying to add a page from one PDFDocument to another.

**Broken:**
```typescript
const sourcePage = sourceDoc.getPage(0);
targetDoc.addPage(sourcePage); // ERROR — page belongs to sourceDoc
```

**Fix:** ALWAYS use `copyPages()` to transfer pages between documents.
```typescript
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage); // Works — page now belongs to targetDoc
```

---

## AP-009: Forgetting to addPage() After copyPages()

**Symptom:** Copied pages do not appear in the output PDF. File has fewer pages than expected.

**Broken:**
```typescript
await targetDoc.copyPages(sourceDoc, [0, 1, 2]);
// Pages are copied but NOT added to the document
const pdfBytes = await targetDoc.save(); // Empty or missing pages
```

**Fix:** ALWAYS call `addPage()` for each copied page.
```typescript
const copiedPages = await targetDoc.copyPages(sourceDoc, [0, 1, 2]);
copiedPages.forEach((page) => targetDoc.addPage(page));
const pdfBytes = await targetDoc.save(); // All 3 pages present
```

---

## AP-010: Using fs Module in Browser Context

**Symptom:** `fs is not defined` or `Cannot find module 'fs'` in browser environment.

**Broken:**
```typescript
import * as fs from 'fs';
fs.writeFileSync('output.pdf', pdfBytes); // FAILS in browser
```

**Fix:** Use Blob + download pattern for browser, fs for Node.js only.

**Browser:**
```typescript
const blob = new Blob([pdfBytes], { type: 'application/pdf' });
const url = URL.createObjectURL(blob);
const link = document.createElement('a');
link.href = url;
link.download = 'output.pdf';
link.click();
URL.revokeObjectURL(url);
```

**Node.js:**
```typescript
import * as fs from 'fs';
fs.writeFileSync('output.pdf', pdfBytes);
```

---

## AP-011: Not Setting Font Before drawText()

**Symptom:** No text appears, or default font is used unexpectedly.

**Broken:**
```typescript
const page = pdfDoc.addPage();
page.drawText('Hello World'); // No font specified — may use default or fail
```

**Fix:** ALWAYS embed a font and pass it to `drawText()`.
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage();
page.drawText('Hello World', {
  x: 50,
  y: 700,
  size: 14,
  font,
});
```

Alternative: set font at page level.
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage();
page.setFont(font);
page.setFontSize(14);
page.drawText('Hello World', { x: 50, y: 700 }); // Uses page-level font
```

---

## AP-012: Incorrect TypeScript Target for pdf-lib

**Symptom:** TypeScript compilation errors related to `Uint8Array`, `ArrayBuffer`, or `Promise`.

**Broken tsconfig.json:**
```json
{
  "compilerOptions": {
    "target": "ES5",
    "lib": ["ES5"]
  }
}
```

**Fix:** ALWAYS target ES2020 or later for pdf-lib projects.
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"]
  }
}
```

---

## AP-013: Font Subsetting Not Enabled

**Symptom:** PDF file size is unexpectedly large (full font file embedded).

**Broken:**
```typescript
const font = await pdfDoc.embedFont(fontBytes); // Full font embedded (~500KB+)
```

**Fix:** ALWAYS enable subsetting for production builds.
```typescript
const font = await pdfDoc.embedFont(fontBytes, { subset: true }); // Only used glyphs (~20KB)
```

> Exception: disable subsetting (`subset: false`) if the PDF will be further modified and may need additional glyphs.

---

## AP-014: Guessing Form Field Names

**Symptom:** `No field with name "X" exists` error when filling forms.

**Broken:**
```typescript
form.getTextField('name');     // Guessed name — may not exist
form.getTextField('Name');     // Field names are case-sensitive
form.getTextField('fullName'); // Actual name might be "form.personal.fullName"
```

**Fix:** ALWAYS enumerate fields first to discover exact names.
```typescript
const fields = form.getFields();
fields.forEach((field) => {
  console.log(`"${field.getName()}" (${field.constructor.name})`);
});
// Then use exact names from the output
```

---

## Setup Checklist

Before considering a pdf-lib project setup complete, verify:

- [ ] `pdf-lib` is in `dependencies` (not `devDependencies`)
- [ ] `@pdf-lib/fontkit` is in `dependencies` IF custom fonts are needed
- [ ] `tsconfig.json` targets ES2020 or later
- [ ] `esModuleInterop: true` in tsconfig (required for fontkit import)
- [ ] `skipLibCheck: true` in tsconfig (prevents type conflicts)
- [ ] All `embedFont()`, `embedPng()`, `embedJpg()`, `save()` calls use `await`
- [ ] `registerFontkit()` is called BEFORE any custom `embedFont()` call
- [ ] Color values use 0.0-1.0 range (NOT 0-255)
- [ ] CDN URLs are version-pinned in browser projects
- [ ] File output uses platform-appropriate method (fs for Node.js, Blob for browser)
