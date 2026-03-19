---
name: pdflib-core-architecture
description: "Guides pdf-lib architecture including PDFDocument lifecycle, async-first API design, bottom-left coordinate system, key types, installation, and project setup. Activates when creating PDFs with pdf-lib, understanding pdf-lib project structure, or setting up a new pdf-lib project."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-core-architecture

## Quick Reference

### Architecture Overview (pdf-lib 1.x)

| Aspect | Detail |
|--------|--------|
| Library | pdf-lib 1.17.1 (MIT License) |
| Language | Pure TypeScript/JavaScript — zero native dependencies |
| Platforms | Node.js, Browser, Deno, React Native |
| Module formats | ESM (`es/index.js`), CJS (`cjs/index.js`), UMD (`dist/pdf-lib.min.js`) |
| TypeScript | Built-in type definitions (`cjs/index.d.ts`) |
| Author | Andrew Dillon (hopding) |

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `@pdf-lib/standard-fonts` | 14 built-in PDF standard fonts |
| `@pdf-lib/upng` | PNG encoding/decoding |
| `pako` | zlib compression |
| `tslib` | TypeScript runtime helpers |
| `@pdf-lib/fontkit` | Custom font support (optional, separate install) |

### Core Type Hierarchy

| Type | Purpose |
|------|---------|
| `PDFDocument` | Top-level entry point for ALL operations — create, load, save |
| `PDFPage` | Individual page — drawing surface with dimensions and rotation |
| `PDFFont` | Embedded font — text measurement and encoding |
| `PDFImage` | Embedded image (PNG or JPEG only) — scaling methods |
| `PDFForm` | Container for all interactive form fields |
| `PDFField` | Base class for form fields |
| `PDFTextField` | Text input field |
| `PDFCheckBox` | Checkbox field |
| `PDFRadioGroup` | Radio button group |
| `PDFDropdown` | Dropdown/combobox field |
| `PDFOptionList` | List selection field |
| `PDFButton` | Push button (can hold image) |
| `PDFSignature` | Digital signature field (read-only) |
| `PDFEmbeddedPage` | Reference to an embedded page from another PDF |

### Critical Warnings

**NEVER** use `StandardFonts` (Helvetica, TimesRoman, Courier) with non-Latin characters -- they support WinAnsi encoding ONLY. ALWAYS use a custom font with fontkit for Unicode text. This is the #1 source of pdf-lib errors.

**NEVER** forget to `await` async methods (`create`, `load`, `save`, `embedFont`, `embedPng`, `embedJpg`, `copyPages`) -- passing a Promise instead of a resolved value causes silent failures or runtime errors.

**NEVER** add a page from one document directly to another -- ALWAYS use `copyPages()` on the destination document first, then `addPage()` the copied page.

**NEVER** embed custom font bytes without calling `pdfDoc.registerFontkit(fontkit)` first -- this ALWAYS throws an error.

**NEVER** use `rgb(255, 0, 0)` -- color values are 0.0 to 1.0, NOT 0 to 255. ALWAYS use `rgb(1, 0, 0)` for red.

**NEVER** assume y=0 is the top of the page -- PDF coordinates use bottom-left origin. ALWAYS calculate from `page.getHeight()` for top-aligned content.

---

## Async vs Sync Decision Tree

```
Is the operation async (needs await)?
+-- Creating/loading a document?
|   +-- PDFDocument.create() -> YES, async
|   +-- PDFDocument.load() -> YES, async
+-- Embedding resources?
|   +-- pdfDoc.embedFont(StandardFonts.X) -> YES, async
|   +-- pdfDoc.embedStandardFont(StandardFonts.X) -> NO, sync (the ONLY sync embed)
|   +-- pdfDoc.embedFont(fontBytes) -> YES, async
|   +-- pdfDoc.embedPng() -> YES, async
|   +-- pdfDoc.embedJpg() -> YES, async
|   +-- pdfDoc.embedPdf() -> YES, async
|   +-- pdfDoc.embedPage() -> YES, async
+-- Saving output?
|   +-- pdfDoc.save() -> YES, async
|   +-- pdfDoc.saveAsBase64() -> YES, async
+-- Copying pages?
|   +-- pdfDoc.copyPages() -> YES, async
+-- Drawing on a page?
|   +-- page.drawText() -> NO, sync
|   +-- page.drawImage() -> NO, sync
|   +-- page.drawRectangle() -> NO, sync
|   +-- ALL page.draw*() methods -> NO, sync
+-- Page/form management?
|   +-- pdfDoc.addPage() -> NO, sync
|   +-- pdfDoc.getPage() -> NO, sync
|   +-- pdfDoc.getForm() -> NO, sync
|   +-- form.getTextField() -> NO, sync
|   +-- field.setText() -> NO, sync
```

**Rule of thumb**: Embedding, loading, saving, and copying are ALWAYS async. Drawing and querying are ALWAYS sync.

---

## Coordinate System

- **Origin**: Bottom-left corner (0, 0)
- **X axis**: Increases to the RIGHT
- **Y axis**: Increases UPWARD (opposite of HTML/CSS/Canvas)
- **Units**: PDF points (1 point = 1/72 inch)

### Common Page Sizes

| Size | Width (pt) | Height (pt) |
|------|-----------|-------------|
| Letter | 612 | 792 |
| A4 | 595.28 | 841.89 |
| Legal | 612 | 1008 |
| A3 | 841.89 | 1190.55 |
| Tabloid | 792 | 1224 |

### Positioning Pattern

```typescript
import { PDFDocument, PageSizes } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage(PageSizes.A4);
const { width, height } = page.getSize();

// Top of page (subtract from height)
page.drawText('Header', { x: 50, y: height - 50 });

// Center of page
page.drawText('Center', { x: width / 2, y: height / 2 });

// Bottom of page
page.drawText('Footer', { x: 50, y: 30 });
```

---

## Installation & Setup

### npm / yarn

```bash
npm install pdf-lib
# Optional: for custom fonts (TTF/OTF)
npm install @pdf-lib/fontkit
```

### CDN (UMD — Browser)

```html
<!-- Pin version for production -->
<script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>
```

### Deno

```bash
deno run --allow-write https://pdf-lib.js.org/deno/quick_start.ts
```

### Standard Import Pattern

```typescript
// Core imports (ALWAYS needed)
import { PDFDocument, StandardFonts, rgb, degrees, PageSizes } from 'pdf-lib';

// Custom font support (ONLY when using TTF/OTF fonts)
import fontkit from '@pdf-lib/fontkit';

// Additional color functions
import { cmyk, grayscale } from 'pdf-lib';

// Additional types/enums
import { BlendMode, TextAlignment, LineCapStyle } from 'pdf-lib';
```

### fontkit Registration Pattern

ALWAYS register fontkit BEFORE embedding custom fonts:

```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);  // MUST come before embedFont() with font bytes
const customFont = await pdfDoc.embedFont(fontBytes);
```

---

## PDFDocument Lifecycle

### Create -> Configure -> Draw -> Save

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';

// 1. Create or load
const pdfDoc = await PDFDocument.create();

// 2. Add pages
const page = pdfDoc.addPage(PageSizes.A4);

// 3. Embed resources (fonts, images)
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);

// 4. Draw content (sync operations)
const { height } = page.getSize();
page.drawText('Hello World', {
  x: 50,
  y: height - 50,
  size: 24,
  font,
  color: rgb(0, 0, 0),
});

// 5. Save output
const pdfBytes = await pdfDoc.save();
```

### Load Existing -> Modify -> Save

```typescript
const pdfDoc = await PDFDocument.load(existingPdfBytes);
const pages = pdfDoc.getPages();
const firstPage = pages[0];
// ... modify content ...
const pdfBytes = await pdfDoc.save();
```

### Load Options

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `ignoreEncryption` | `boolean` | `false` | Load encrypted PDFs (no decryption) |
| `updateMetadata` | `boolean` | `true` | Auto-update producer, dates |
| `throwOnInvalidObject` | `boolean` | `false` | Strict parsing mode |

### Save Options

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `useObjectStreams` | `boolean` | `true` | Compression (disable for legacy compatibility) |
| `addDefaultPage` | `boolean` | `true` | Insert blank page if document is empty |
| `updateFieldAppearances` | `boolean` | `true` | Refresh form field visuals on save |

---

## Platform Support

| Platform | Works | Notes |
|----------|-------|-------|
| Node.js | YES | Full support, use `fs.readFileSync` / `fs.writeFileSync` for I/O |
| Browser | YES | Use `fetch()` for loading, Blob/URL for downloading |
| Deno | YES | Use `Deno.readFile` / `Deno.writeFile` for I/O |
| React Native | YES | Use `react-native-fs` or `expo-file-system` for I/O |

pdf-lib handles ONLY the in-memory PDF manipulation. File I/O is the responsibility of the host platform.

---

## Color System

ALWAYS use values between 0.0 and 1.0 for ALL color functions:

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib';

const red = rgb(1, 0, 0);           // RGB: 0.0-1.0 per channel
const cyan = cmyk(1, 0, 0, 0);      // CMYK: 0.0-1.0 per channel
const midGray = grayscale(0.5);      // Grayscale: 0.0 (black) to 1.0 (white)
```

---

## Font Decision Tree

```
What text do you need to render?
+-- ASCII / basic Latin only?
|   +-- Use StandardFonts (Helvetica, TimesRoman, Courier)
|   +-- embedStandardFont() for sync, embedFont() for async
+-- Extended Latin (accents, diacritics)?
|   +-- StandardFonts: PARTIAL support (WinAnsi subset only)
|   +-- For full support: use custom font + fontkit
+-- Non-Latin (CJK, Arabic, Cyrillic, Thai)?
|   +-- StandardFonts: NEVER works
|   +-- MUST use custom font (TTF/OTF) + fontkit
|   +-- Font file MUST contain the required glyphs
+-- Emoji?
|   +-- NEVER use StandardFonts
|   +-- Custom font support is LIMITED and unreliable
```

---

## Reference Links

- [references/methods.md](references/methods.md) -- API signatures for PDFDocument, PDFPage, PDFFont, PDFImage, PDFForm
- [references/examples.md](references/examples.md) -- Working code examples verified against pdf-lib 1.x documentation
- [references/anti-patterns.md](references/anti-patterns.md) -- What NOT to do, with explanations and correct alternatives

### Official Sources

- https://pdf-lib.js.org/ -- Official homepage and examples
- https://pdf-lib.js.org/docs/api/ -- API documentation
- https://github.com/Hopding/pdf-lib -- Source repository
- https://www.npmjs.com/package/pdf-lib -- npm registry
