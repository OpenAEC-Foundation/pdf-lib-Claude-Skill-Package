# pdf-lib Vooronderzoek — Consolidated Research Document

> **Research date**: 2026-03-19
> **Library**: pdf-lib 1.17.1 (MIT License)
> **Author**: Andrew Dillon (hopding)
> **Repository**: https://github.com/Hopding/pdf-lib
> **Homepage**: https://pdf-lib.js.org/
> **Sources**: Official site, API docs, GitHub README, GitHub issues, npm registry
> **Fragments consolidated**: core-api-research.md, text-font-image-research.md, forms-drawing-manipulation-research.md

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Installation & Setup](#2-installation--setup)
3. [PDFDocument Lifecycle](#3-pdfdocument-lifecycle)
4. [Page Operations](#4-page-operations)
5. [Text Operations](#5-text-operations)
6. [Font System](#6-font-system)
7. [Image Operations](#7-image-operations)
8. [Drawing Operations](#8-drawing-operations)
9. [Form System](#9-form-system)
10. [Document Manipulation](#10-document-manipulation)
11. [Anti-Patterns & Common Errors](#11-anti-patterns--common-errors)
12. [API Quick Reference](#12-api-quick-reference)

---

## 1. Architecture Overview

### 1.1 Package Metadata

| Field | Value |
|-------|-------|
| Version | 1.17.1 |
| License | MIT |
| Repository | https://github.com/Hopding/pdf-lib |
| Homepage | https://pdf-lib.js.org/ |
| Main (CJS) | `cjs/index.js` |
| Main (ESM) | `es/index.js` |
| UMD | `dist/pdf-lib.min.js` |
| TypeScript defs | `cjs/index.d.ts` (+ downleveled `ts3.4/` for TS <=3.5) |

### 1.2 Dependencies

| Dependency | Version |
|-----------|---------|
| @pdf-lib/standard-fonts | ^1.0.0 |
| @pdf-lib/upng | ^1.0.1 |
| pako | ^1.0.11 |
| tslib | ^1.11.1 |

### 1.3 Platform Support

Written in TypeScript, compiled to pure JavaScript with no native dependencies. Works in:
- Node.js
- Browsers
- Deno
- React Native

### 1.4 Core Classes

| Class | Purpose |
|-------|---------|
| `PDFDocument` | Top-level document. Entry point for all operations. |
| `PDFPage` | Individual page. Drawing surface with dimension/rotation control. |
| `PDFFont` | Embedded font. Provides text measurement methods. |
| `PDFImage` | Embedded image (PNG or JPEG). Provides scaling methods. |
| `PDFForm` | Container for all interactive form fields. |
| `PDFField` | Base class for form fields. |
| `PDFTextField` | Text input field. |
| `PDFCheckBox` | Checkbox field. |
| `PDFRadioGroup` | Radio button group. |
| `PDFDropdown` | Dropdown/combobox field. |
| `PDFOptionList` | List selection field. |
| `PDFButton` | Push button (can hold image). |
| `PDFSignature` | Digital signature field. |
| `PDFEmbeddedPage` | Reference to embedded page from another PDF. |

### 1.5 Async-First Design

ALL major operations are async and return Promises:
- `PDFDocument.create()` / `PDFDocument.load()` — async
- `pdfDoc.embedFont()` — async
- `pdfDoc.embedPng()` / `pdfDoc.embedJpg()` — async
- `pdfDoc.copyPages()` — async
- `pdfDoc.embedPdf()` / `pdfDoc.embedPage()` — async
- `pdfDoc.save()` / `pdfDoc.saveAsBase64()` — async
- `pdfDoc.attach()` — async

**Exception**: `pdfDoc.embedStandardFont()` is synchronous (no `await` needed).

Drawing operations on PDFPage are synchronous:
- `page.drawText()` — sync
- `page.drawImage()` — sync
- `page.drawRectangle()` — sync
- All other `page.draw*()` methods — sync

### 1.6 Coordinate System

- **Origin**: Bottom-left corner of the page (0, 0)
- **X axis**: Increases to the right
- **Y axis**: Increases upward
- **Units**: PDF points (1 point = 1/72 inch)
- **Typical dimensions**: Letter = 612 x 792 pt, A4 = 595.28 x 841.89 pt

This means to draw text at the "top" of a Letter page, use y ≈ 750 (not 0).

```typescript
const { height } = page.getSize()
// Top of page
page.drawText('Near top', { x: 50, y: height - 50 })
// Bottom of page
page.drawText('Near bottom', { x: 50, y: 50 })
```

### 1.7 PDFDocument Properties

| Property | Type | Description |
|----------|------|-------------|
| `catalog` | `PDFCatalog` | The document catalog |
| `context` | `PDFContext` | Low-level document context |
| `defaultWordBreaks` | `string[]` | Default word breaks (defaults to `[' ']`) |
| `isEncrypted` | `boolean` | Whether document is encrypted |

---

## 2. Installation & Setup

### 2.1 npm / yarn

```bash
npm install pdf-lib
# or
yarn add pdf-lib
```

### 2.2 CDN (UMD)

```
https://unpkg.com/pdf-lib/dist/pdf-lib.js
https://unpkg.com/pdf-lib/dist/pdf-lib.min.js
https://cdn.jsdelivr.net/npm/pdf-lib/dist/pdf-lib.js
https://cdn.jsdelivr.net/npm/pdf-lib/dist/pdf-lib.min.js
```
For production, pin the version: `https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js`

### 2.3 Deno

```bash
deno run --allow-write https://pdf-lib.js.org/deno/quick_start.ts
```

### 2.4 Standard Imports

```typescript
import { PDFDocument, StandardFonts, rgb, degrees, PageSizes } from 'pdf-lib';
```

### 2.5 Custom Font Support (requires fontkit)

```bash
npm install @pdf-lib/fontkit
```

```typescript
import fontkit from '@pdf-lib/fontkit';
pdfDoc.registerFontkit(fontkit);
```

### 2.6 Import Checklist

```typescript
// Core
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib'

// For custom fonts
import fontkit from '@pdf-lib/fontkit'

// Additional color functions
import { cmyk, grayscale } from 'pdf-lib'

// Additional types/enums
import { BlendMode, PageSizes, TextAlignment, LineCapStyle } from 'pdf-lib'
```

---

## 3. PDFDocument Lifecycle

### 3.1 Create a New Document

```typescript
static async create(options?: CreateOptions): Promise<PDFDocument>
```

**CreateOptions**:
| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, creation/modification dates |

```typescript
const pdfDoc = await PDFDocument.create();
// With options:
const pdfDoc = await PDFDocument.create({ updateMetadata: false });
```

### 3.2 Load an Existing Document

```typescript
static async load(
  pdf: string | Uint8Array | ArrayBuffer,
  options?: LoadOptions
): Promise<PDFDocument>
```

**Input types**:
- `string` — Base64-encoded string or data URI
- `Uint8Array` — Raw PDF bytes
- `ArrayBuffer` — Raw PDF bytes

**LoadOptions**:
| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ignoreEncryption` | `boolean` | `false` | Skip encryption validation (load encrypted PDFs) |
| `parseSpeed` | `number` | — | Parsing performance level |
| `throwOnInvalidObject` | `boolean` | `false` | Throw error on malformed PDF objects |
| `updateMetadata` | `boolean` | `true` | Auto-update producer, modification date, creator, creation date |
| `capNumbers` | `boolean` | `false` | Limit numeric precision |

```typescript
// Basic load
const pdfDoc = await PDFDocument.load(existingPdfBytes);

// Load encrypted PDF
const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  ignoreEncryption: true,
});

// Load without updating metadata
const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  updateMetadata: false,
});

// Load from base64
const pdfDoc = await PDFDocument.load(base64String);
```

### 3.3 Save Document

```typescript
save(options?: SaveOptions): Promise<Uint8Array>
```

**SaveOptions**:
| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `useObjectStreams` | `boolean` | `true` | Enable object stream compression |
| `addDefaultPage` | `boolean` | `true` | Insert blank page if document is empty |
| `objectsPerTick` | `number` | `50` | Processing batch size per event loop tick |
| `updateFieldAppearances` | `boolean` | `true` | Refresh form field visual renderings |

```typescript
const pdfBytes = await pdfDoc.save();

// Without object streams (larger file but more compatible)
const pdfBytes = await pdfDoc.save({ useObjectStreams: false });

// Skip form field appearance updates
const pdfBytes = await pdfDoc.save({ updateFieldAppearances: false });
```

### 3.4 Save as Base64

```typescript
saveAsBase64(options?: Base64SaveOptions): Promise<string>
```

**Base64SaveOptions** extends SaveOptions with:
| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dataUri` | `boolean` | `false` | Return `data:application/pdf;base64,...` format |

```typescript
const base64 = await pdfDoc.saveAsBase64();
const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });
```

### 3.5 Copy Document

```typescript
copy(): Promise<PDFDocument>
```
Creates a copy of the document. **Note**: Does NOT copy acroforms, outlines, etc.

### 3.6 Flush

```typescript
flush(): Promise<void>
```
Flushes embedded fonts, pages, and images to context. Automatically called by `save()` and `saveAsBase64()`.

### 3.7 Document Metadata

#### Getters

```typescript
getTitle(): string | undefined
getAuthor(): string | undefined
getSubject(): string | undefined
getKeywords(): string | undefined
getCreator(): string | undefined
getProducer(): string | undefined
getCreationDate(): Date | undefined
getModificationDate(): Date | undefined
```

#### Setters

```typescript
setTitle(title: string, options?: SetTitleOptions): void
setAuthor(author: string): void
setSubject(subject: string): void
setKeywords(keywords: string[]): void
setCreator(creator: string): void
setProducer(producer: string): void
setCreationDate(creationDate: Date): void
setModificationDate(modificationDate: Date): void
setLanguage(language: string): void  // RFC 3066 language tags
```

**SetTitleOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `showInWindowTitleBar` | `boolean` | Display title in PDF viewer title bar |

```typescript
pdfDoc.setTitle('My Document', { showInWindowTitleBar: true });
pdfDoc.setAuthor('John Doe');
pdfDoc.setSubject('Document Subject');
pdfDoc.setKeywords(['pdf', 'generation', 'typescript']);
pdfDoc.setCreator('My Application');
pdfDoc.setProducer('pdf-lib');
pdfDoc.setCreationDate(new Date());
pdfDoc.setModificationDate(new Date());
pdfDoc.setLanguage('en-US');
```

**Note**: When `updateMetadata` is `true` (default) during `create()` or `load()`, the library automatically sets producer to `"pdf-lib (https://github.com/Hopding/pdf-lib)"`, creator to `"pdf-lib (https://github.com/Hopding/pdf-lib)"`, and the creation/modification dates to the current time.

### 3.8 Attachments

```typescript
await pdfDoc.attach(fileBytes: string | Uint8Array | ArrayBuffer, name: string, options?: AttachmentOptions): Promise<void>
```

**AttachmentOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `mimeType` | `string` | MIME type of the attachment |
| `description` | `string` | Human-readable description |
| `creationDate` | `Date` | Creation timestamp |
| `modificationDate` | `Date` | Modification timestamp |

```typescript
await pdfDoc.attach(csvBytes, 'data.csv', {
  mimeType: 'text/csv',
  description: 'Exported data file',
  creationDate: new Date(),
  modificationDate: new Date(),
});
```

**Important**: Only some PDF readers can view attachments — Adobe Reader, Foxit Reader, and Firefox.

### 3.9 JavaScript Actions

```typescript
pdfDoc.addJavaScript(name: string, script: string): void
```

Adds named JavaScript that executes on document open. The name must be unique per document.

```typescript
pdfDoc.addJavaScript('onOpen', 'app.alert("Hello from pdf-lib!")');
```

---

## 4. Page Operations

### 4.1 Adding Pages

```typescript
addPage(page?: PDFPage | [number, number]): PDFPage
```
- No args: creates blank page with default dimensions
- `[width, height]`: creates page with specified dimensions (in PDF points)
- `PDFPage`: adds an existing page

```typescript
// Default size page
const page = pdfDoc.addPage();

// Custom size
const page = pdfDoc.addPage([550, 750]);

// Using PageSizes enum
const page = pdfDoc.addPage(PageSizes.A4);
const page = pdfDoc.addPage(PageSizes.Letter);
```

### 4.2 Inserting Pages

```typescript
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
```

```typescript
// Insert at beginning
const page = pdfDoc.insertPage(0);

// Insert at index 2 with custom size
const page = pdfDoc.insertPage(2, [612, 792]);
```

### 4.3 Removing Pages

```typescript
removePage(index: number): void
```

```typescript
pdfDoc.removePage(0); // Remove first page
```

### 4.4 Getting Pages

```typescript
getPage(index: number): PDFPage
getPages(): PDFPage[]
getPageCount(): number
getPageIndices(): number[]
```

```typescript
const pages = pdfDoc.getPages();
const firstPage = pdfDoc.getPage(0);
const count = pdfDoc.getPageCount();
const indices = pdfDoc.getPageIndices(); // [0, 1, 2, ...]
```

### 4.5 PDFPage Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this page belongs to |
| `node` | `PDFPageLeaf` | Low-level PDF dictionary |
| `ref` | `PDFRef` | Unique reference within the document |

### 4.6 Dimension Methods

```typescript
getSize(): { width: number; height: number }
setSize(width: number, height: number): void
getWidth(): number
setWidth(width: number): void
getHeight(): number
setHeight(height: number): void
```

### 4.7 Box Methods

Each returns/sets `{ x: number, y: number, width: number, height: number }`:

```typescript
// Media box (physical page size)
getMediaBox(): { x: number; y: number; width: number; height: number }
setMediaBox(x: number, y: number, width: number, height: number): void

// Crop box (visible area)
getCropBox(): { x: number; y: number; width: number; height: number }
setCropBox(x: number, y: number, width: number, height: number): void

// Bleed box (printing bleed area)
getBleedBox(): { x: number; y: number; width: number; height: number }
setBleedBox(x: number, y: number, width: number, height: number): void

// Trim box (intended final size)
getTrimBox(): { x: number; y: number; width: number; height: number }
setTrimBox(x: number, y: number, width: number, height: number): void

// Art box (extent of meaningful content)
getArtBox(): { x: number; y: number; width: number; height: number }
setArtBox(x: number, y: number, width: number, height: number): void
```

### 4.8 Rotation

```typescript
getRotation(): Rotation    // Returns angle as Rotation type (degrees)
setRotation(angle: Rotation): void  // Must be multiple of 90
```

```typescript
import { degrees } from 'pdf-lib';
page.setRotation(degrees(90));
page.setRotation(degrees(180));
page.setRotation(degrees(270));
```

### 4.9 Position / Cursor Methods

```typescript
getPosition(): { x: number; y: number }
getX(): number
getY(): number
moveTo(x: number, y: number): void
moveUp(yIncrease: number): void
moveDown(yDecrease: number): void
moveLeft(xDecrease: number): void
moveRight(xIncrease: number): void
resetPosition(): void  // Resets to (0, 0)
```

### 4.10 Content Transformation

```typescript
translateContent(x: number, y: number): void    // Move page content
scale(x: number, y: number): void                // Scale size + content + annotations
scaleContent(x: number, y: number): void         // Scale content only
scaleAnnotations(x: number, y: number): void     // Scale annotations only
```

### 4.11 Page-Level Font & Text Defaults

```typescript
setFont(font: PDFFont): void
setFontSize(fontSize: number): void
setFontColor(fontColor: Color): void
setLineHeight(lineHeight: number): void
```

When these are set, `drawText()` uses them as defaults — individual calls can still override.

### 4.12 Low-Level Operators

```typescript
pushOperators(...operator: PDFOperator[]): void
```

### 4.13 PageSizes Enum

All dimensions in PDF points (1 point = 1/72 inch).

#### ISO A Series

| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| 4A0 | 4767.87 | 6740.79 |
| 2A0 | 3370.39 | 4767.87 |
| A0 | 2383.94 | 3370.39 |
| A1 | 1683.78 | 2383.94 |
| A2 | 1190.55 | 1683.78 |
| A3 | 841.89 | 1190.55 |
| A4 | 595.28 | 841.89 |
| A5 | 419.53 | 595.28 |
| A6 | 297.64 | 419.53 |
| A7 | 209.76 | 297.64 |
| A8 | 147.4 | 209.76 |
| A9 | 104.88 | 147.4 |
| A10 | 73.7 | 104.88 |

#### ISO B Series

| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| B0 | 2834.65 | 4008.19 |
| B1 | 2004.09 | 2834.65 |
| B2 | 1417.32 | 2004.09 |
| B3 | 1000.63 | 1417.32 |
| B4 | 708.66 | 1000.63 |
| B5 | 498.9 | 708.66 |
| B6 | 354.33 | 498.9 |
| B7 | 249.45 | 354.33 |
| B8 | 175.75 | 249.45 |
| B9 | 124.72 | 175.75 |
| B10 | 87.87 | 124.72 |

#### ISO C Series (Envelope Sizes)

| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| C0 | 2599.37 | 3676.54 |
| C1 | 1836.85 | 2599.37 |
| C2 | 1298.27 | 1836.85 |
| C3 | 918.43 | 1298.27 |
| C4 | 649.13 | 918.43 |
| C5 | 459.21 | 649.13 |
| C6 | 323.15 | 459.21 |
| C7 | 229.61 | 323.15 |
| C8 | 161.57 | 229.61 |
| C9 | 113.39 | 161.57 |
| C10 | 79.37 | 113.39 |

#### ISO RA / SRA Series (Raw/Supplementary Raw)

| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| RA0 | 2437.8 | 3458.27 |
| RA1 | 1729.13 | 2437.8 |
| RA2 | 1218.9 | 1729.13 |
| RA3 | 864.57 | 1218.9 |
| RA4 | 609.45 | 864.57 |
| SRA0 | 2551.18 | 3628.35 |
| SRA1 | 1814.17 | 2551.18 |
| SRA2 | 1275.59 | 1814.17 |
| SRA3 | 907.09 | 1275.59 |
| SRA4 | 637.8 | 907.09 |

#### North American Sizes

| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| Executive | 521.86 | 756.0 |
| Folio | 612.0 | 936.0 |
| Legal | 612.0 | 1008.0 |
| Letter | 612.0 | 792.0 |
| Tabloid | 792.0 | 1224.0 |

---

## 5. Text Operations

### 5.1 drawText() — Complete API

```typescript
page.drawText(text: string, options?: PDFPageDrawTextOptions): void
```

**PDFPageDrawTextOptions** (ALL properties optional):

| Property | Type | Purpose |
|----------|------|---------|
| `x` | `number` | Horizontal position (default: 0) |
| `y` | `number` | Vertical position (default: 0) |
| `font` | `PDFFont` | Font to use (set via setFont() or here) |
| `size` | `number` | Font size in points |
| `color` | `Color` | Text color (rgb, cmyk, or grayscale) |
| `opacity` | `number` | Transparency 0.0 (invisible) to 1.0 (opaque) |
| `rotate` | `Rotation` | Rotation via degrees() or radians() |
| `xSkew` | `Rotation` | Horizontal skew transformation |
| `ySkew` | `Rotation` | Vertical skew transformation |
| `lineHeight` | `number` | Vertical spacing between lines |
| `maxWidth` | `number` | Text wrapping constraint (wraps at word breaks) |
| `wordBreaks` | `string[]` | Custom word break characters for wrapping |
| `blendMode` | `BlendMode` | Color blending method |

**Key behaviors:**
- Text is drawn from the BOTTOM-LEFT origin (PDF coordinate system — y=0 is bottom of page)
- Multiline text: embed `\n` in the string; lineHeight controls spacing
- maxWidth triggers automatic word wrapping at wordBreaks characters
- Default wordBreaks: `[' ']` (space character)

### 5.2 Complete drawText() Examples

**Basic text:**
```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const timesRomanFont = await pdfDoc.embedFont(StandardFonts.TimesRoman)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

page.drawText('Creating PDFs in JavaScript is awesome!', {
  x: 50,
  y: height - 120,
  size: 30,
  font: timesRomanFont,
  color: rgb(0, 0.53, 0.71),
})

const pdfBytes = await pdfDoc.save()
```

**Rotated text on existing PDF:**
```typescript
import { degrees, PDFDocument, rgb, StandardFonts } from 'pdf-lib'

const existingPdfBytes = ... // Uint8Array or ArrayBuffer
const pdfDoc = await PDFDocument.load(existingPdfBytes)
const helveticaFont = await pdfDoc.embedFont(StandardFonts.Helvetica)

const pages = pdfDoc.getPages()
const firstPage = pages[0]
const { width, height } = firstPage.getSize()

firstPage.drawText('This text was added with JavaScript!', {
  x: 5,
  y: height / 2 + 300,
  size: 50,
  font: helveticaFont,
  color: rgb(0.95, 0.1, 0.1),
  rotate: degrees(-45),
})

const pdfBytes = await pdfDoc.save()
```

**Multiline text with lineHeight:**
```typescript
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50,
  y: 500,
  size: 14,
  font: helveticaFont,
  lineHeight: 20,
  color: rgb(0, 0, 0),
})
```

### 5.3 Centering Text Example

```typescript
const text = 'Centered Title'
const fontSize = 24
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()

page.drawText(text, {
  x: (pageWidth - textWidth) / 2,  // horizontally centered
  y: 500,
  size: fontSize,
  font: font,
})
```

### 5.4 Text Layout Utilities

```typescript
layoutSinglelineText(text: string, options): TextLayout
layoutMultilineText(text: string, options): TextLayout
layoutCombedText(text: string, options): TextLayout
computeFontSize(lines: string[], font: PDFFont, bounds): number
computeCombedFontSize(line: string, font: PDFFont, bounds, cellCount: number): number
```

---

## 6. Font System

### 6.1 StandardFonts Enum — Complete List (14 fonts)

| Enum Key | PDF Name | Family |
|----------|----------|--------|
| `StandardFonts.Courier` | `"Courier"` | Monospace |
| `StandardFonts.CourierBold` | `"Courier-Bold"` | Monospace |
| `StandardFonts.CourierOblique` | `"Courier-Oblique"` | Monospace |
| `StandardFonts.CourierBoldOblique` | `"Courier-BoldOblique"` | Monospace |
| `StandardFonts.Helvetica` | `"Helvetica"` | Sans-serif |
| `StandardFonts.HelveticaBold` | `"Helvetica-Bold"` | Sans-serif |
| `StandardFonts.HelveticaOblique` | `"Helvetica-Oblique"` | Sans-serif |
| `StandardFonts.HelveticaBoldOblique` | `"Helvetica-BoldOblique"` | Sans-serif |
| `StandardFonts.TimesRoman` | `"Times-Roman"` | Serif |
| `StandardFonts.TimesRomanBold` | `"Times-Bold"` | Serif |
| `StandardFonts.TimesRomanItalic` | `"Times-Italic"` | Serif |
| `StandardFonts.TimesRomanBoldItalic` | `"Times-BoldItalic"` | Serif |
| `StandardFonts.Symbol` | `"Symbol"` | Symbol |
| `StandardFonts.ZapfDingbats` | `"ZapfDingbats"` | Dingbats |

### 6.2 Standard Font Embedding

```typescript
// Async method (returns Promise<PDFFont>)
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica)

// Sync alternative (no await needed)
const courier = pdfDoc.embedStandardFont(StandardFonts.Courier)
```

### 6.3 Standard Font Limitations — CRITICAL

**Standard fonts use WinAnsi encoding ONLY.** This means:

- ONLY Latin-1 characters are supported (Western European subset)
- NO Unicode support: Cyrillic, Chinese, Arabic, Hebrew, Thai, Korean, Japanese — ALL FAIL
- NO emoji support
- Error thrown: `"WinAnsi cannot encode "X" (0xNNNN)"`

**Characters that FAIL with standard fonts:**
- Cyrillic: characters like U+0422, etc.
- Chinese: any CJK character
- Arabic: any Arabic script
- Hebrew: any Hebrew character
- Thai: any Thai character
- Emoji: any emoji character
- Special Unicode: fraction slash, various mathematical symbols

**This is the #1 source of issues in pdf-lib** (dozens of open GitHub issues).

### 6.4 Font Compatibility Matrix

| Font Family | Latin (A-Z, 0-9) | Accented (e, u, n) | Cyrillic | CJK | Arabic | Emoji |
|-------------|:-:|:-:|:-:|:-:|:-:|:-:|
| Standard (all 14) | YES | PARTIAL (WinAnsi subset) | NO | NO | NO | NO |
| Custom TTF/OTF (with fontkit) | YES | YES | YES* | YES* | YES* | DEPENDS* |

*Depends on the specific font file containing the required glyphs.

### 6.5 Custom Fonts — fontkit Registration

```typescript
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)  // MUST call before embedFont() with custom font bytes
```

**@pdf-lib/fontkit package:**
- npm: `@pdf-lib/fontkit`
- Installation: `npm install @pdf-lib/fontkit`
- Purpose: Enables custom font (TTF/OTF) embedding and unicode support
- MUST be registered before calling `embedFont()` with font bytes
- NOT needed for StandardFonts

### 6.6 registerFontkit() Signature

```typescript
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

### 6.7 embedFont() — Complete Signature

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

**Input types for `font` parameter:**
- `StandardFonts` enum value — standard font (no fontkit needed)
- `string` — base64-encoded font data or data URI
- `Uint8Array` — raw font file bytes
- `ArrayBuffer` — raw font file bytes

**EmbedFontOptions:**

| Property | Type | Purpose |
|----------|------|---------|
| `subset` | `boolean \| undefined` | Enable font subsetting (reduces file size by including only used glyphs) |
| `customName` | `string \| undefined` | Custom name for the embedded font |
| `features` | `TypeFeatures \| undefined` | OpenType font feature configuration |

### 6.8 Complete Custom Font Example with Measurement

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

// Load font bytes (from file, fetch, etc.)
const fontUrl = 'https://pdf-lib.js.org/assets/ubuntu/Ubuntu-R.ttf'
const fontBytes = await fetch(fontUrl).then((res) => res.arrayBuffer())

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
const text = 'This is text in an embedded font!'
const textSize = 35
const textWidth = customFont.widthOfTextAtSize(text, textSize)
const textHeight = customFont.heightAtSize(textSize)

// Draw text
page.drawText(text, {
  x: 40,
  y: 450,
  size: textSize,
  font: customFont,
  color: rgb(0, 0.53, 0.71),
})

// Draw bounding box around text (for visualization)
page.drawRectangle({
  x: 40,
  y: 450,
  width: textWidth,
  height: textHeight,
  borderColor: rgb(1, 0, 0),
  borderWidth: 1.5,
})
```

### 6.9 Unicode Text with Custom Font

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

const fontBytes = await fetch(fontUrl).then((res) => res.arrayBuffer())

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)
const ubuntuFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
page.drawText('Some fancy Unicode text in the font', {
  font: ubuntuFont,
})
```

### 6.10 Font Subsetting

```typescript
// With subsetting — smaller file, only includes used glyphs
const font = await pdfDoc.embedFont(fontBytes, { subset: true })

// Without subsetting — full font embedded, larger file
const font = await pdfDoc.embedFont(fontBytes, { subset: false })
```

**Subsetting trade-offs:**
- `subset: true` — Smaller PDF file size. Only glyphs actually used in the document are embedded. Recommended for production.
- `subset: false` — Larger PDF file size. Full font embedded. Useful when the PDF will be further modified and may need additional glyphs later.

### 6.11 PDFFont Class — Complete API

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this font belongs to |
| `name` | `string` | The name of this font |
| `ref` | `PDFRef` | Unique reference within the document |

**Methods:**

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `widthOfTextAtSize` | `(text: string, size: number)` | `number` | Width of text string at given font size (in PDF points) |
| `heightAtSize` | `(size: number, options?: { descender?: boolean })` | `number` | Height of font at given size |
| `sizeAtHeight` | `(height: number)` | `number` | Compute font size needed to achieve target height |
| `encodeText` | `(text: string)` | `PDFHexString` | Encode text for this font. NOTE: drawText() handles this automatically |
| `getCharacterSet` | `()` | `number[]` | Returns array of unicode code points this font supports |
| `embed` | `()` | `Promise<void>` | Embed font in document. Called automatically by save() |

### 6.12 Text Layout Calculations

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)

const text = 'Hello World'
const fontSize = 24

// Measure text dimensions
const textWidth = font.widthOfTextAtSize(text, fontSize)   // width in points
const textHeight = font.heightAtSize(fontSize)              // height in points

// Calculate font size from desired height
const targetHeight = 30 // points
const neededSize = font.sizeAtHeight(targetHeight)

// Check if font supports specific characters
const charSet = font.getCharacterSet() // number[] of unicode code points
const supportsChar = charSet.includes('e'.charCodeAt(0))
```

---

## 7. Image Operations

### 7.1 Supported Formats

**ONLY two formats supported:**
- **PNG** — via `embedPng()`
- **JPEG/JPG** — via `embedJpg()`

**NOT supported:** GIF, BMP, TIFF, WebP, SVG, AVIF, HEIC — NONE of these work.

### 7.2 embedPng() Signature

```typescript
pdfDoc.embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64 encoded PNG or data URI (`data:image/png;base64,...`)
- `Uint8Array` — raw PNG bytes
- `ArrayBuffer` — raw PNG bytes

### 7.3 embedJpg() Signature

```typescript
pdfDoc.embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64 encoded JPEG or data URI (`data:image/jpeg;base64,...`)
- `Uint8Array` — raw JPEG bytes
- `ArrayBuffer` — raw JPEG bytes

### 7.4 PDFImage Class — Complete API

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this image belongs to |
| `width` | `number` | Original width in pixels |
| `height` | `number` | Original height in pixels |
| `ref` | `PDFRef` | Unique reference within the document |

**Methods:**

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `scale` | `(factor: number)` | `{ width: number, height: number }` | Scale dimensions by factor. E.g., 500x250 at 0.5 = 250x125 |
| `scaleToFit` | `(width: number, height: number)` | `{ width: number, height: number }` | Scale to fit within bounds, preserving aspect ratio |
| `size` | `()` | `{ width: number, height: number }` | Get original dimensions as object |
| `embed` | `()` | `Promise<void>` | Embed image in document. Called automatically by save() |

### 7.5 drawImage() — Complete API

```typescript
page.drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

**PDFPageDrawImageOptions** (ALL properties optional):

| Property | Type | Purpose |
|----------|------|---------|
| `x` | `number` | Horizontal position (default: 0) |
| `y` | `number` | Vertical position (default: 0) |
| `width` | `number` | Display width (default: original image width) |
| `height` | `number` | Display height (default: original image height) |
| `rotate` | `Rotation` | Rotation via degrees() or radians() |
| `xSkew` | `Rotation` | Horizontal skew |
| `ySkew` | `Rotation` | Vertical skew |
| `opacity` | `number` | Transparency 0.0-1.0 |
| `blendMode` | `BlendMode` | Color blending method |

### 7.6 Complete Image Embedding Example

```typescript
import { PDFDocument } from 'pdf-lib'

const jpgImageBytes = ... // Uint8Array, ArrayBuffer, or base64 string
const pngImageBytes = ...

const pdfDoc = await PDFDocument.create()
const jpgImage = await pdfDoc.embedJpg(jpgImageBytes)
const pngImage = await pdfDoc.embedPng(pngImageBytes)

const jpgDims = jpgImage.scale(0.25)
const pngDims = pngImage.scale(0.5)

const page = pdfDoc.addPage()

page.drawImage(jpgImage, {
  x: page.getWidth() / 2 - jpgDims.width / 2,
  y: page.getHeight() / 2 - jpgDims.height / 2,
  width: jpgDims.width,
  height: jpgDims.height,
})

page.drawImage(pngImage, {
  x: page.getWidth() / 2 - pngDims.width / 2 + 75,
  y: page.getHeight() / 2 - pngDims.height,
  width: pngDims.width,
  height: pngDims.height,
})

const pdfBytes = await pdfDoc.save()
```

### 7.7 Aspect Ratio Preservation Patterns

**Pattern 1: Uniform scale factor**
```typescript
const image = await pdfDoc.embedPng(pngBytes)
const dims = image.scale(0.5) // 50% of original
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
})
```

**Pattern 2: Fit within bounds (preserves aspect ratio)**
```typescript
const image = await pdfDoc.embedJpg(jpgBytes)
const dims = image.scaleToFit(300, 200) // max 300x200, aspect ratio preserved
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
})
```

**Pattern 3: Manual aspect ratio calculation**
```typescript
const image = await pdfDoc.embedPng(pngBytes)
const targetWidth = 300
const scale = targetWidth / image.width
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: image.height * scale,
})
```

### 7.8 Image Drawing with Rotation and Opacity

```typescript
page.drawImage(jpgImage, {
  x: 25,
  y: 25,
  width: 200,
  height: 150,
  rotate: degrees(30),
  opacity: 0.75,
})
```

---

## 8. Drawing Operations

### 8.1 drawRectangle()

```typescript
page.drawRectangle(options?: PDFPageDrawRectangleOptions): void
```

**PDFPageDrawRectangleOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `width` | `number` | Width |
| `height` | `number` | Height |
| `color` | `Color` | Fill color |
| `borderColor` | `Color` | Border color |
| `borderWidth` | `number` | Border width |
| `rotate` | `Rotation` | Rotation |
| `opacity` | `number` | Fill opacity |
| `borderOpacity` | `number` | Border opacity |
| `blendMode` | `BlendMode` | Blend mode |
| `borderDashArray` | `number[]` | Dash pattern |
| `borderDashPhase` | `number` | Dash phase |
| `borderLineCap` | `LineCapStyle` | Line cap style |

### 8.2 drawSquare()

```typescript
page.drawSquare(options?: PDFPageDrawSquareOptions): void
```
Options: `x`, `y`, `size`, `color`, `borderColor`, `borderWidth`, `rotate`, `opacity`, `borderOpacity`, `blendMode`, `borderDashArray`, `borderDashPhase`, `borderLineCap`.

### 8.3 drawCircle()

```typescript
page.drawCircle(options?: PDFPageDrawCircleOptions): void
```
Options: `x`, `y`, `size` (diameter), `color`, `borderColor`, `borderWidth`, `opacity`, `borderOpacity`, `blendMode`, `borderDashArray`, `borderDashPhase`, `borderLineCap`.

### 8.4 drawEllipse()

```typescript
page.drawEllipse(options?: PDFPageDrawEllipseOptions): void
```
Options: `x`, `y`, `xScale`, `yScale`, `color`, `borderColor`, `borderWidth`, `rotate`, `opacity`, `borderOpacity`, `blendMode`.

### 8.5 drawLine()

```typescript
page.drawLine(options: PDFPageDrawLineOptions): void
```

**PDFPageDrawLineOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `start` | `{ x: number, y: number }` | Start point |
| `end` | `{ x: number, y: number }` | End point |
| `thickness` | `number` | Line thickness |
| `color` | `Color` | Line color |
| `opacity` | `number` | Opacity |
| `dashArray` | `number[]` | Dash pattern |
| `dashPhase` | `number` | Dash phase |
| `lineCap` | `LineCapStyle` | Line cap style |
| `blendMode` | `BlendMode` | Blend mode |

### 8.6 drawSvgPath()

```typescript
page.drawSvgPath(path: string, options?: PDFPageDrawSVGOptions): void
```

**PDFPageDrawSVGOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X offset |
| `y` | `number` | Y offset |
| `color` | `Color` | Fill color |
| `borderColor` | `Color` | Stroke color |
| `borderWidth` | `number` | Stroke width |
| `opacity` | `number` | Fill opacity |
| `borderOpacity` | `number` | Stroke opacity |
| `scale` | `number` | Scale factor |
| `rotate` | `Rotation` | Rotation |
| `blendMode` | `BlendMode` | Blend mode |

**SVG Path Example:**

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const svgPath = 'M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

// Draw at current position (uses page.moveTo())
page.moveTo(100, page.getHeight() - 5)
page.moveDown(25)
page.drawSvgPath(svgPath)

// Draw with stroke only
page.moveDown(200)
page.drawSvgPath(svgPath, { borderColor: rgb(0, 1, 0), borderWidth: 5 })

// Draw with fill only
page.moveDown(200)
page.drawSvgPath(svgPath, { color: rgb(1, 0, 0) })

const pdfBytes = await pdfDoc.save()
```

### 8.7 drawPage() (Embedded Page)

```typescript
page.drawPage(embeddedPage: PDFEmbeddedPage, options?: PDFPageDrawPageOptions): void
```

**PDFPageDrawPageOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `width` | `number` | Display width |
| `height` | `number` | Display height |
| `rotate` | `Rotation` | Rotation |
| `opacity` | `number` | Opacity |
| `xSkew` | `Rotation` | Horizontal skew |
| `ySkew` | `Rotation` | Vertical skew |
| `blendMode` | `BlendMode` | Blend mode |

### 8.8 Color Functions

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib'

// RGB — values 0.0 to 1.0 (NOT 0-255)
const red = rgb(1, 0, 0)
const blue = rgb(0, 0, 1)
const teal = rgb(0, 0.53, 0.71)

// CMYK — values 0.0 to 1.0
const cyan = cmyk(1, 0, 0, 0)
const magenta = cmyk(0, 1, 0, 0)
const black = cmyk(0, 0, 0, 1)

// Grayscale — 0.0 (black) to 1.0 (white)
const gBlack = grayscale(0)
const gWhite = grayscale(1)
const gMidGray = grayscale(0.5)
```

**Color type**: `Grayscale | RGB | CMYK`

### 8.9 Rotation & Transformation Helpers

```typescript
degrees(angle: number): Rotation
radians(angle: number): Rotation
degreesToRadians(degree: number): number
radiansToDegrees(radian: number): number
```

**TransformationMatrix** = `[a, b, c, d, e, f]` (6-element number array)

### 8.10 Enums

| Enum | Values |
|------|--------|
| `TextAlignment` | `Left`, `Center`, `Right` |
| `LineCapStyle` | `Butt`, `Round`, `Square` |
| `LineJoinStyle` | `Miter`, `Round`, `Bevel` |
| `BlendMode` | `Normal`, `Multiply`, `Screen`, `Overlay`, `Darken`, `Lighten`, `ColorDodge`, `ColorBurn`, `HardLight`, `SoftLight`, `Difference`, `Exclusion` |
| `TextRenderingMode` | Text rendering options |

### 8.11 Drawing Options Reference Table

| Option | Applies To | Type | Description |
|--------|-----------|------|-------------|
| `x` | All shapes, text, image | `number` | X coordinate (lower-left origin) |
| `y` | All shapes, text, image | `number` | Y coordinate (lower-left origin) |
| `width` | Rectangle, image, page | `number` | Width in points |
| `height` | Rectangle, image, page | `number` | Height in points |
| `size` | Circle, square, text | `number` | Diameter (circle/square) or font size (text) |
| `color` | All shapes, text | `Color` | Fill color |
| `borderColor` | Rectangle, circle, ellipse, square, SVG | `Color` | Stroke color |
| `borderWidth` | Rectangle, circle, ellipse, square, SVG | `number` | Stroke thickness |
| `opacity` | All | `number` | Fill opacity (0.0-1.0) |
| `borderOpacity` | Rectangle, circle, ellipse, square, SVG | `number` | Stroke opacity (0.0-1.0) |
| `rotate` | Rectangle, square, text, image, page | `Rotation` | `degrees(45)` or `radians(Math.PI/4)` |
| `scale` | SVG path | `number` | Scale factor |
| `thickness` | Line | `number` | Line width in points |
| `start` / `end` | Line | `{ x, y }` | Start and end points |
| `xScale` / `yScale` | Ellipse, embedded page | `number` | Horizontal/vertical scale |
| `font` | Text | `PDFFont` | Font to use |
| `lineHeight` | Text | `number` | Line spacing |
| `maxWidth` | Text | `number` | Text wrapping width |

---

## 9. Form System

### 9.1 Getting the Form Object

```typescript
const form = pdfDoc.getForm()  // Returns: PDFForm
```

### 9.2 PDFForm — Complete API

```typescript
// Getters — retrieve existing fields by name (throws if not found)
form.getFields(): PDFField[]
form.getField(name: string): PDFField
form.getFieldMaybe(name: string): PDFField | undefined
form.getTextField(name: string): PDFTextField
form.getCheckBox(name: string): PDFCheckBox
form.getRadioGroup(name: string): PDFRadioGroup
form.getDropdown(name: string): PDFDropdown
form.getOptionList(name: string): PDFOptionList
form.getButton(name: string): PDFButton
form.getSignature(name: string): PDFSignature
form.getDefaultFont(): PDFFont

// Creators — create new fields
form.createTextField(name: string): PDFTextField
form.createCheckBox(name: string): PDFCheckBox
form.createRadioGroup(name: string): PDFRadioGroup
form.createDropdown(name: string): PDFDropdown
form.createOptionList(name: string): PDFOptionList
form.createButton(name: string): PDFButton

// Field management
form.removeField(field: PDFField): void
form.markFieldAsDirty(fieldRef: PDFRef): void
form.markFieldAsClean(fieldRef: PDFRef): void
form.fieldIsDirty(fieldRef: PDFRef): boolean

// Form operations
form.flatten(options?: FlattenOptions): void  // Default: { updateFieldAppearances: true }
form.updateFieldAppearances(font?: PDFFont): void

// XFA handling
form.hasXFA(): boolean
form.deleteXFA(): void
```

### 9.3 Getting All Field Names

```typescript
const fields = form.getFields()
const fieldNames = fields.map(f => f.getName())
// Returns fully qualified names, e.g. "form.section.fieldName"
```

### 9.4 PDFTextField — Complete API

```typescript
const field = form.getTextField('fieldName')

// Core text operations
field.setText(text: string | undefined): void
field.getText(): string | undefined
field.setFontSize(fontSize: number): void

// Layout & display
field.setAlignment(alignment: TextAlignment): void  // TextAlignment.Left | Center | Right
field.enableMultiline(): void
field.disableMultiline(): void
field.enableScrolling(): void
field.disableScrolling(): void

// Input constraints
field.setMaxLength(maxLength?: number | undefined): void
field.getMaxLength(): number | undefined
field.removeMaxLength(): void

// Special behaviors
field.enableCombing(): void    // Distributes chars equally across cells
field.disableCombing(): void
field.enablePassword(): void   // Masks input
field.disablePassword(): void
field.enableFileSelection(): void
field.disableFileSelection(): void

// Appearance
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(font: PDFFont, provider?: AppearanceProvider): void
field.defaultUpdateAppearances(font: PDFFont): void
field.setImage(image: PDFImage): void

// Field properties
field.enableReadOnly(): void
field.disableReadOnly(): void
field.enableRequired(): void
field.disableRequired(): void
field.enableExporting(): void
field.disableExporting(): void
field.enableRichFormatting(): void
field.disableRichFormatting(): void
field.enableSpellChecking(): void
field.disableSpellChecking(): void

// Query methods
field.getName(): string
field.isMultiline(): boolean
field.isPassword(): boolean
field.isCombed(): boolean
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isFileSelector(): boolean
field.isExported(): boolean
field.isScrollable(): boolean
field.isSpellChecked(): boolean
field.isRichFormatted(): boolean
field.needsAppearancesUpdate(): boolean
field.getAlignment(): TextAlignment
```

### 9.5 PDFCheckBox — Complete API

```typescript
const field = form.getCheckBox('fieldName')

// Core
field.check(): void
field.uncheck(): void
field.isChecked(): boolean

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(provider?: AppearanceProviderFor<PDFCheckBox>): void
field.defaultUpdateAppearances(): void

// Properties (inherited)
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
field.getName(): string
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isExported(): boolean
field.needsAppearancesUpdate(): boolean
```

### 9.6 PDFRadioGroup — Complete API

```typescript
const field = form.getRadioGroup('groupName')

// Core
field.select(option: string): void
field.getSelected(): string | undefined
field.getOptions(): string[]
field.clear(): void

// Widget management
field.addOptionToPage(option: string, page: PDFPage, options?: FieldAppearanceOptions): void

// Mutual exclusion
field.enableMutualExclusion(): void
field.disableMutualExclusion(): void
field.isMutuallyExclusive(): boolean

// Off-toggle
field.enableOffToggling(): void
field.disableOffToggling(): void
field.isOffToggleable(): boolean

// Appearance
field.updateAppearances(provider?: AppearanceProviderFor<PDFRadioGroup>): void
field.defaultUpdateAppearances(): void
field.needsAppearancesUpdate(): boolean

// Properties (inherited)
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
field.getName(): string
```

### 9.7 PDFDropdown — Complete API

```typescript
const field = form.getDropdown('fieldName')

// Core
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void  // Replaces entire list

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void

// Configuration
field.enableMultiselect(): void / field.disableMultiselect(): void
field.enableEditing(): void / field.disableEditing(): void
field.enableSorting(): void / field.disableSorting(): void
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.enableSpellChecking(): void / field.disableSpellChecking(): void
```

### 9.8 PDFOptionList — Complete API

```typescript
const field = form.getOptionList('fieldName')

// Core
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.clear(): void
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void

// Configuration
field.enableMultiselect(): void / field.disableMultiselect(): void
field.isMultiselect(): boolean
field.enableSorting(): void / field.disableSorting(): void
field.isSorted(): boolean
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.isSelectOnClick(): boolean

// Appearance
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFOptionList>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean
```

### 9.9 PDFButton — Complete API

```typescript
const field = form.getButton('fieldName')

// Core
field.setImage(image: PDFImage, alignment?: ImageAlignment): void
field.setFontSize(fontSize: number): void

// Display — NOTE: addToPage takes a text label as first argument (unlike other field types)
field.addToPage(text: string, page: PDFPage, options?: FieldAppearanceOptions): void

// Appearance
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFButton>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean

// Properties (inherited)
field.getName(): string
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
```

### 9.10 PDFSignature

```typescript
const field = form.getSignature('fieldName')
// Primarily a metadata container for digital signature fields
// No create method available — signatures are read from existing PDFs
```

### 9.11 Form Filling — Complete Workflow

**Fill Existing Form:**
```typescript
import { PDFDocument } from 'pdf-lib'

const formPdfBytes = /* Uint8Array or ArrayBuffer */
const pdfDoc = await PDFDocument.load(formPdfBytes)
const form = pdfDoc.getForm()

// Text fields
form.getTextField('CharacterName 2').setText('Mario')
form.getTextField('Age').setText('24 years')

// Checkboxes
form.getCheckBox('Check Box3').check()

// Radio groups
form.getRadioGroup('Group2').select('Choice1')

// Dropdowns
form.getDropdown('Dropdown7').select('Infinity')

// Buttons (with images)
const marioImage = await pdfDoc.embedPng(marioImageBytes)
form.getButton('CHARACTER IMAGE').setImage(marioImage)

const pdfBytes = await pdfDoc.save()
```

**Create New Form:**
```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage([550, 750])
const form = pdfDoc.getForm()

// Text field
page.drawText('Enter your favorite superhero:', { x: 50, y: 700, size: 20 })
const superheroField = form.createTextField('favorite.superhero')
superheroField.setText('One Punch Man')
superheroField.addToPage(page, { x: 55, y: 640 })

// Radio group
const rocketField = form.createRadioGroup('favorite.rocket')
rocketField.addOptionToPage('Falcon Heavy', page, { x: 55, y: 540 })
rocketField.addOptionToPage('Saturn IV', page, { x: 55, y: 480 })
rocketField.select('Saturn IV')

// Checkbox
const exiaField = form.createCheckBox('gundam.exia')
exiaField.addToPage(page, { x: 55, y: 380 })
exiaField.check()

// Dropdown
const planetsField = form.createDropdown('favorite.planet')
planetsField.addOptions(['Venus', 'Earth', 'Mars', 'Pluto'])
planetsField.select('Pluto')
planetsField.addToPage(page, { x: 55, y: 220 })

// Option list
const personField = form.createOptionList('favorite.person')
personField.addOptions(['Julius Caesar', 'Ada Lovelace', 'Cleopatra'])
personField.select('Ada Lovelace')
personField.addToPage(page, { x: 55, y: 70 })

const pdfBytes = await pdfDoc.save()
```

### 9.12 Unicode / Custom Font Support for Forms

The default font used for form fields is Helvetica, which only supports Latin characters. For non-Latin text:

```typescript
import fontkit from '@pdf-lib/fontkit'

pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes) // TTF or OTF bytes

// Fill fields first
form.getTextField('name').setText('Japanese text here')

// Then update appearances with the custom font
form.updateFieldAppearances(customFont)

const pdfBytes = await pdfDoc.save()
```

### 9.13 Form Flattening

```typescript
form.flatten(options?: {
  updateFieldAppearances?: boolean  // Default: true
})
```

**What Flattening Does:**
- Converts interactive form fields into static, non-editable content
- Field values become part of the page content stream
- Users can no longer modify values in the PDF
- Reduces file complexity and improves compatibility

**When to Flatten:**
- ALWAYS flatten when generating final output PDFs (invoices, certificates, reports)
- ALWAYS flatten before distributing PDFs to prevent modification
- NEVER flatten if the form needs to be filled again later
- NEVER flatten if you need to programmatically read field values later

**Selective Flattening Note:** As of the current API, `form.flatten()` flattens ALL fields. There are open PRs (GitHub Issues #1758, #1367) for selective flattening. Workaround: use `form.removeField(field)` on specific fields before calling `form.flatten()`.

### 9.14 XFA Handling

```typescript
form.hasXFA(): boolean   // Check if form uses XFA (XML Forms Architecture)
form.deleteXFA(): void   // Remove XFA data to work with AcroForm fields instead
```

### 9.15 Form Decision Trees

**Form Field Selection:**
```
Need to work with form fields?
+-- Reading an existing form?
|   +-- Know field names? -> form.getTextField('name') / getCheckBox() / etc.
|   +-- Don't know names? -> form.getFields().map(f => ({ name: f.getName(), type: f.constructor.name }))
+-- Creating a new form?
|   +-- Text input -> form.createTextField('name')
|   +-- Yes/No choice -> form.createCheckBox('name')
|   +-- One-of-many choice -> form.createRadioGroup('name')
|   +-- Selection from list -> form.createDropdown('name')
|   +-- Multi-select list -> form.createOptionList('name')
|   +-- Image/button -> form.createButton('name')
+-- Making form read-only?
    +-- Permanent (static PDF) -> form.flatten()
    +-- Still interactive but locked -> field.enableReadOnly() on each field
```

**Font Decision Tree for Forms:**
```
Setting text in form fields?
+-- Latin characters only (ASCII)?
|   +-- Default Helvetica works -- no extra setup needed
+-- Extended Latin (diacritics)?
|   +-- Register fontkit
|   +-- Embed a Unicode-supporting font (TTF/OTF)
|   +-- Call form.updateFieldAppearances(customFont)
+-- Non-Latin (CJK, Arabic, etc.)?
    +-- Register fontkit (REQUIRED)
    +-- Embed font that supports the script
    +-- Call form.updateFieldAppearances(customFont)
```

---

## 10. Document Manipulation

### 10.1 Copying Pages Between Documents

```typescript
copyPages(srcDoc: PDFDocument, indices: number[]): Promise<PDFPage[]>
```

```typescript
const srcDoc = await PDFDocument.load(srcPdfBytes);
const dstDoc = await PDFDocument.create();

// Copy pages 0 and 2 from source
const [page1, page3] = await dstDoc.copyPages(srcDoc, [0, 2]);
dstDoc.addPage(page1);
dstDoc.addPage(page3);
```

### 10.2 Embedding Pages (for drawing within other pages)

```typescript
embedPdf(
  pdf: string | Uint8Array | ArrayBuffer | PDFDocument,
  indices?: number[]
): Promise<PDFEmbeddedPage[]>

embedPage(
  page: PDFPage,
  boundingBox?: PageBoundingBox,
  transformationMatrix?: [number, number, number, number, number, number]
): Promise<PDFEmbeddedPage>

embedPages(
  pages: PDFPage[],
  boundingBoxes?: PageBoundingBox[],
  transformationMatrices?: [number, number, number, number, number, number][]
): Promise<PDFEmbeddedPage[]>
```

**PageBoundingBox**: `{ left: number, bottom: number, right: number, top: number }`

```typescript
// Embed a page from another PDF
const [embeddedPage] = await pdfDoc.embedPdf(otherPdfBytes);
page.drawPage(embeddedPage, { x: 50, y: 50, width: 300, height: 400 });

// Embed with clipping
const embeddedPage = await pdfDoc.embedPage(sourcePage, {
  left: 0,
  bottom: 0,
  right: 300,
  top: 400,
});
```

### 10.3 Merge Multiple Documents (Complete Pattern)

```typescript
import { PDFDocument } from 'pdf-lib'

async function mergeDocuments(pdfByteArrays: Uint8Array[]): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create()

  for (const pdfBytes of pdfByteArrays) {
    const sourcePdf = await PDFDocument.load(pdfBytes)
    const pageCount = sourcePdf.getPageCount()
    const indices = Array.from({ length: pageCount }, (_, i) => i)
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices)
    copiedPages.forEach(page => mergedPdf.addPage(page))
  }

  return await mergedPdf.save()
}
```

### 10.4 Extract Single Page

```typescript
async function extractPage(pdfBytes: Uint8Array, pageIndex: number): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const newPdf = await PDFDocument.create()

  const [extractedPage] = await newPdf.copyPages(sourcePdf, [pageIndex])
  newPdf.addPage(extractedPage)

  return await newPdf.save()
}
```

### 10.5 Extract Page Range

```typescript
async function extractPageRange(
  pdfBytes: Uint8Array,
  startPage: number,  // 0-indexed
  endPage: number     // 0-indexed, inclusive
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const newPdf = await PDFDocument.create()

  const indices = Array.from(
    { length: endPage - startPage + 1 },
    (_, i) => startPage + i
  )
  const pages = await newPdf.copyPages(sourcePdf, indices)
  pages.forEach(page => newPdf.addPage(page))

  return await newPdf.save()
}
```

### 10.6 Split Into Individual Pages

```typescript
async function splitAllPages(pdfBytes: Uint8Array): Promise<Uint8Array[]> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const pageCount = sourcePdf.getPageCount()
  const results: Uint8Array[] = []

  for (let i = 0; i < pageCount; i++) {
    const newPdf = await PDFDocument.create()
    const [page] = await newPdf.copyPages(sourcePdf, [i])
    newPdf.addPage(page)
    results.push(await newPdf.save())
  }

  return results
}
```

### 10.7 Encrypted PDFs

```typescript
const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,
})

pdfDoc.isEncrypted  // boolean — check if document is encrypted
```

**Key Notes on Encryption:**
- `ignoreEncryption: true` allows loading PDFs that have encryption markers but may not have actual content encryption
- pdf-lib does NOT support decrypting PDFs with passwords (no built-in decryption engine)
- pdf-lib does NOT support encrypting/password-protecting output PDFs
- For truly encrypted PDFs, you may need to pre-process with another tool (e.g., qpdf, mutool)

### 10.8 Document Merging Decision Tree

```
Need to combine PDFs?
+-- Merge entire documents -> Loop: copyPages(source, allIndices) + addPage()
+-- Extract specific pages -> copyPages(source, [specificIndices])
+-- Split into single pages -> Loop: create new doc per page, copyPages + addPage
+-- Reorder pages -> copyPages all, then addPage/insertPage in desired order
+-- Remove pages -> copyPages only the pages you want to KEEP

CRITICAL: ALWAYS use copyPages() -- NEVER add pages directly from another document
```

---

## 11. Anti-Patterns & Common Errors

### 11.1 Standard Fonts with Unicode — THE #1 ERROR

**Error:** `"WinAnsi cannot encode "X" (0xNNNN)"`

**Cause:** Using StandardFonts (Helvetica, TimesRoman, Courier) with non-WinAnsi characters.

**Broken code:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Non-latin text', { font }) // CRASHES with non-WinAnsi chars
```

**Fix:** Use a custom font with fontkit:
```typescript
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes) // TTF/OTF with required glyph support
page.drawText('Non-latin text', { font }) // Works!
```

### 11.2 Missing fontkit Registration

**Error:** Attempting to embed custom font bytes without registering fontkit first.

**Broken code:**
```typescript
const pdfDoc = await PDFDocument.create()
// MISSING: pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes) // CRASHES
```

**Fix:** ALWAYS register fontkit before embedding custom fonts:
```typescript
import fontkit from '@pdf-lib/fontkit'
const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)              // MUST come first
const font = await pdfDoc.embedFont(fontBytes) // Now works
```

### 11.3 Unsupported Image Formats

pdf-lib ONLY supports `embedPng()` and `embedJpg()`. There is NO `embedGif()`, `embedBmp()`, `embedSvg()`, or generic `embedImage()`.

**Workaround:** Convert images to PNG or JPG before embedding (using a canvas or image processing library).

### 11.4 Coordinate System Confusion

PDF coordinates start at BOTTOM-LEFT (y=0 is the bottom of the page). This is the opposite of HTML/CSS/Canvas where y=0 is the top.

```typescript
const { width, height } = page.getSize()

// Top of page
page.drawText('Near top', { x: 50, y: height - 50 })

// Bottom of page
page.drawText('Near bottom', { x: 50, y: 50 })
```

### 11.5 Forgetting to Await Async Methods

```typescript
// WRONG — embedFont returns a Promise
const font = pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font }) // font is a Promise, not a PDFFont!

// CORRECT
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })
```

### 11.6 Cross-Document Page Addition

**Issue pattern**: Trying to add a page from one document directly to another without `copyPages()`.

```typescript
// ANTI-PATTERN — will throw error
const sourcePage = sourceDoc.getPage(0)
targetDoc.addPage(sourcePage)

// CORRECT PATTERN
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(copiedPage)
```

### 11.7 Form Field Not Found

**Cause:** Field names are case-sensitive and use fully qualified names (dot-separated hierarchy).

```typescript
// ANTI-PATTERN — guessing field names
form.getTextField('name')  // May throw if actual name is "form.personal.name"

// CORRECT PATTERN — enumerate fields first
const fields = form.getFields()
fields.forEach(field => {
  console.log(`Type: ${field.constructor.name}, Name: "${field.getName()}"`)
})
// Then use exact names from enumeration
```

### 11.8 Font Not Applied to Form Fields

**Cause:** Default Helvetica only supports Latin characters. Custom fonts need explicit appearance update.

```typescript
// ANTI-PATTERN — setting text without font consideration
form.getTextField('name').setText('Non-Latin text')
// Result: Characters may not render

// CORRECT PATTERN — register fontkit and update appearances
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)
form.getTextField('name').setText('Non-Latin text')
form.updateFieldAppearances(customFont)
```

### 11.9 Duplicate Field Names

Creating two fields with the same name generates an error.

```typescript
// ANTI-PATTERN
form.createTextField('myField')
form.createTextField('myField')  // ERROR: duplicate name

// CORRECT PATTERN — use unique names
form.createTextField('myField_1')
form.createTextField('myField_2')
```

### 11.10 Form Fields Lost During copyPages()

Form field data (AcroForm entries) may not transfer correctly when copying pages between documents. The field widgets (visual representations) are copied but the AcroForm field definitions may not transfer, resulting in non-functional form fields. Workaround: re-create form fields after merge if needed.

### 11.11 Blank Pages in Merged PDFs

Content streams may reference resources that don't copy correctly. Particularly affects PDFs viewed in Adobe Acrobat Reader (may work in browsers). No guaranteed workaround — test merged output in multiple viewers.

### 11.12 File Size Bloat After Merge

Duplicate resources (fonts, images) from source documents are not deduplicated. 433MB output from 15MB input reported in issue #1338. No built-in deduplication — each copied page brings its own resources.

### 11.13 Textarea Only Shows Value on Focus

**Cause:** Appearance streams not updated after setting text.

```typescript
// ANTI-PATTERN
field.setText('value')
// Field appears empty until clicked

// CORRECT PATTERN — ensure appearances are updated
field.setText('value')
form.updateFieldAppearances()  // Or field.updateAppearances(font)
```

### 11.14 Image Aspect Ratio Distortion

```typescript
// BAD — image will be distorted
page.drawImage(image, {
  x: 50, y: 50,
  width: 300,  // arbitrary width
  height: 300, // arbitrary height — aspect ratio broken!
})

// CORRECT — use scale() or scaleToFit()
const dims = image.scale(0.5)
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
})
```

### 11.15 Color Values 0-255 Instead of 0-1

```typescript
// WRONG — values must be 0-1
const red = rgb(255, 0, 0)   // This creates white (values are clamped/overflow)

// CORRECT
const red = rgb(1, 0, 0)
```

### 11.16 Page Copying Workflow Mistake

When copying pages between documents, you must: (1) call `copyPages()` on the destination, (2) then `addPage()` or `insertPage()` each copied page. `copyPages()` alone does not add them.

### 11.17 Keywords Setter/Getter Asymmetry

`setKeywords(['a', 'b'])` takes `string[]`, but `getKeywords()` returns `string | undefined` (a single comma-separated string).

### 11.18 Known Open GitHub Issues

| Issue | Problem | Status |
|-------|---------|--------|
| #1759 | Standard fonts cannot encode characters outside WinAnsi | Open |
| #1566 | Chinese character support | Open |
| #1450 | Arabic bidirectional text | Open |
| #561 | Hebrew/UTF-8 support | Open |
| #715 | Cyrillic/Russian symbols | Open |
| #1010 | Thai font rendering (unwanted spaces) | Open |
| #746 | Khmer font problems | Open |
| #1297 | Unicode fraction slash not rendering | Open |
| #50, #217 | Emoji embedding issues | Open |
| #1270 | ZapfDingbats font errors | Open |
| #1665 | Cannot merge multiple Standard Fonts for mixed Unicode | Open |
| #1152 | Non-Latin dropdown values trigger WinAnsi encoding failures | Open |
| #726 | Embedded font glyph width calculation errors | PR |
| #1670, #1620, #1585 | Form field not found (name mismatch) | Open |
| #1750, #1538, #1378 | Font not applied to form fields | Open |
| #1652 | Duplicate field names | Open |
| #1205, #1587 | Form fields lost during copyPages() | Open |
| #1579, #1767 | Blank pages in merged PDFs | Open |
| #1338 | File size bloat after merge | Open |
| #1584 | Textarea only shows value on focus | Open |
| #1488 | Diacritics in field values | Open |
| #1732 | Restricted document form filling | Open |
| #1758, #1367 | Selective field flattening (open PRs) | Open |

---

## 12. API Quick Reference

### PDFDocument — Static Methods

| Method | Returns | Async |
|--------|---------|-------|
| `PDFDocument.create(options?)` | `Promise<PDFDocument>` | Yes |
| `PDFDocument.load(pdf, options?)` | `Promise<PDFDocument>` | Yes |

### PDFDocument — Instance Methods

| Method | Returns | Async |
|--------|---------|-------|
| `addPage(page?)` | `PDFPage` | No |
| `insertPage(index, page?)` | `PDFPage` | No |
| `removePage(index)` | `void` | No |
| `getPage(index)` | `PDFPage` | No |
| `getPages()` | `PDFPage[]` | No |
| `getPageCount()` | `number` | No |
| `getPageIndices()` | `number[]` | No |
| `copyPages(srcDoc, indices)` | `Promise<PDFPage[]>` | Yes |
| `embedFont(font, options?)` | `Promise<PDFFont>` | Yes |
| `embedStandardFont(font)` | `PDFFont` | No |
| `embedPng(png)` | `Promise<PDFImage>` | Yes |
| `embedJpg(jpg)` | `Promise<PDFImage>` | Yes |
| `embedPdf(pdf, indices?)` | `Promise<PDFEmbeddedPage[]>` | Yes |
| `embedPage(page, bbox?, matrix?)` | `Promise<PDFEmbeddedPage>` | Yes |
| `embedPages(pages, bboxes?)` | `Promise<PDFEmbeddedPage[]>` | Yes |
| `save(options?)` | `Promise<Uint8Array>` | Yes |
| `saveAsBase64(options?)` | `Promise<string>` | Yes |
| `copy()` | `Promise<PDFDocument>` | Yes |
| `flush()` | `Promise<void>` | Yes |
| `getForm()` | `PDFForm` | No |
| `attach(bytes, name, options?)` | `Promise<void>` | Yes |
| `addJavaScript(name, script)` | `void` | No |
| `registerFontkit(fontkit)` | `void` | No |
| `setTitle(title, options?)` | `void` | No |
| `setAuthor(author)` | `void` | No |
| `setSubject(subject)` | `void` | No |
| `setKeywords(keywords)` | `void` | No |
| `setCreator(creator)` | `void` | No |
| `setProducer(producer)` | `void` | No |
| `setCreationDate(date)` | `void` | No |
| `setModificationDate(date)` | `void` | No |
| `setLanguage(language)` | `void` | No |

### PDFPage — Drawing Methods

| Method | Returns | Key Options |
|--------|---------|-------------|
| `drawText(text, options?)` | `void` | x, y, size, font, color, opacity, rotate, lineHeight, maxWidth |
| `drawImage(image, options?)` | `void` | x, y, width, height, rotate, opacity |
| `drawRectangle(options?)` | `void` | x, y, width, height, color, borderColor, borderWidth |
| `drawSquare(options?)` | `void` | x, y, size, color, borderColor |
| `drawCircle(options?)` | `void` | x, y, size, color, borderColor |
| `drawEllipse(options?)` | `void` | x, y, xScale, yScale, color, borderColor |
| `drawLine(options)` | `void` | start, end, thickness, color |
| `drawSvgPath(path, options?)` | `void` | x, y, color, borderColor, scale |
| `drawPage(embeddedPage, options?)` | `void` | x, y, width, height, rotate, opacity |

### PDFFont — Methods

| Method | Returns |
|--------|---------|
| `widthOfTextAtSize(text, size)` | `number` |
| `heightAtSize(size, options?)` | `number` |
| `sizeAtHeight(height)` | `number` |
| `encodeText(text)` | `PDFHexString` |
| `getCharacterSet()` | `number[]` |

### PDFImage — Methods

| Method | Returns |
|--------|---------|
| `scale(factor)` | `{ width, height }` |
| `scaleToFit(width, height)` | `{ width, height }` |
| `size()` | `{ width, height }` |

### PDFForm — Key Methods

| Method | Returns |
|--------|---------|
| `getFields()` | `PDFField[]` |
| `getTextField(name)` | `PDFTextField` |
| `getCheckBox(name)` | `PDFCheckBox` |
| `getRadioGroup(name)` | `PDFRadioGroup` |
| `getDropdown(name)` | `PDFDropdown` |
| `getOptionList(name)` | `PDFOptionList` |
| `getButton(name)` | `PDFButton` |
| `getSignature(name)` | `PDFSignature` |
| `createTextField(name)` | `PDFTextField` |
| `createCheckBox(name)` | `PDFCheckBox` |
| `createRadioGroup(name)` | `PDFRadioGroup` |
| `createDropdown(name)` | `PDFDropdown` |
| `createOptionList(name)` | `PDFOptionList` |
| `createButton(name)` | `PDFButton` |
| `flatten(options?)` | `void` |
| `updateFieldAppearances(font?)` | `void` |
| `removeField(field)` | `void` |
| `hasXFA()` | `boolean` |
| `deleteXFA()` | `void` |

### Color Functions

| Function | Input Range | Returns |
|----------|-------------|---------|
| `rgb(r, g, b)` | 0.0-1.0 | `RGB` |
| `cmyk(c, m, y, k)` | 0.0-1.0 | `CMYK` |
| `grayscale(gray)` | 0.0-1.0 | `Grayscale` |

### Rotation Functions

| Function | Returns |
|----------|---------|
| `degrees(angle)` | `Rotation` |
| `radians(angle)` | `Rotation` |

---

## Verification Status

| Source | URL | Fetched |
|--------|-----|---------|
| pdf-lib official site | https://pdf-lib.js.org | 2026-03-19 |
| pdf-lib API docs (PDFDocument) | https://pdf-lib.js.org/docs/api/classes/pdfdocument | 2026-03-19 |
| pdf-lib API docs (PDFPage) | https://pdf-lib.js.org/docs/api/classes/pdfpage | 2026-03-19 |
| pdf-lib API docs (PDFForm) | https://pdf-lib.js.org/docs/api/classes/pdfform | 2026-03-19 |
| pdf-lib API docs (PDFTextField) | https://pdf-lib.js.org/docs/api/classes/pdftextfield | 2026-03-19 |
| pdf-lib API docs (PDFCheckBox) | https://pdf-lib.js.org/docs/api/classes/pdfcheckbox | 2026-03-19 |
| pdf-lib API docs (PDFRadioGroup) | https://pdf-lib.js.org/docs/api/classes/pdfradiogroup | 2026-03-19 |
| pdf-lib API docs (PDFDropdown) | https://pdf-lib.js.org/docs/api/classes/pdfdropdown | 2026-03-19 |
| pdf-lib API docs (PDFOptionList) | https://pdf-lib.js.org/docs/api/classes/pdfoptionlist | 2026-03-19 |
| pdf-lib API docs (PDFButton) | https://pdf-lib.js.org/docs/api/classes/pdfbutton | 2026-03-19 |
| GitHub README | https://github.com/Hopding/pdf-lib | 2026-03-19 |
| GitHub Issues (font+unicode) | https://github.com/Hopding/pdf-lib/issues?q=font+unicode | 2026-03-19 |
| GitHub Issues (forms) | https://github.com/Hopding/pdf-lib/issues?q=form+fill | 2026-03-19 |
| GitHub Issues (merge) | https://github.com/Hopding/pdf-lib/issues?q=merge+copy+pages | 2026-03-19 |
| npm registry | https://registry.npmjs.org/pdf-lib | 2026-03-19 |
