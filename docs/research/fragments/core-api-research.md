# pdf-lib Core API Research — Part 1 of 3

> **Research date**: 2026-03-19
> **Sources fetched**: pdf-lib.js.org, github.com/Hopding/pdf-lib, registry.npmjs.org/pdf-lib
> **Library version**: 1.17.1
> **License**: MIT
> **Author**: Andrew Dillon (hopding)

---

## 1. Package Metadata

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
| Maintainer | hopding (Andrew Dillon) |

### Dependencies
| Dependency | Version |
|-----------|---------|
| @pdf-lib/standard-fonts | ^1.0.0 |
| @pdf-lib/upng | ^1.0.1 |
| pako | ^1.0.11 |
| tslib | ^1.11.1 |

### Platform Support
Written in TypeScript, compiled to pure JavaScript with no native dependencies. Works in:
- Node.js
- Browsers
- Deno
- React Native

---

## 2. Installation & Setup

### npm / yarn
```bash
npm install pdf-lib
# or
yarn add pdf-lib
```

### CDN (UMD)
```
https://unpkg.com/pdf-lib/dist/pdf-lib.js
https://unpkg.com/pdf-lib/dist/pdf-lib.min.js
https://cdn.jsdelivr.net/npm/pdf-lib/dist/pdf-lib.js
https://cdn.jsdelivr.net/npm/pdf-lib/dist/pdf-lib.min.js
```
For production, pin the version: `https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js`

### Deno
```bash
deno run --allow-write https://pdf-lib.js.org/deno/quick_start.ts
```

### Standard Imports
```typescript
import { PDFDocument, StandardFonts, rgb, degrees, PageSizes } from 'pdf-lib';
```

### Custom Font Support (requires fontkit)
```bash
npm install @pdf-lib/fontkit
```
```typescript
import fontkit from '@pdf-lib/fontkit';
pdfDoc.registerFontkit(fontkit);
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

---

## 4. PDFDocument Properties

| Property | Type | Description |
|----------|------|-------------|
| `catalog` | `PDFCatalog` | The document catalog |
| `context` | `PDFContext` | Low-level document context |
| `defaultWordBreaks` | `string[]` | Default word breaks (defaults to `[' ']`) |
| `isEncrypted` | `boolean` | Whether document is encrypted |

---

## 5. Page Operations

### 5.1 Adding Pages

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

### 5.2 Inserting Pages

```typescript
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
```

```typescript
// Insert at beginning
const page = pdfDoc.insertPage(0);

// Insert at index 2 with custom size
const page = pdfDoc.insertPage(2, [612, 792]);
```

### 5.3 Removing Pages

```typescript
removePage(index: number): void
```

```typescript
pdfDoc.removePage(0); // Remove first page
```

### 5.4 Getting Pages

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

### 5.5 Copying Pages Between Documents

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

### 5.6 Embedding Pages (for drawing within other pages)

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

---

## 6. PageSizes Enum

All dimensions in PDF points (1 point = 1/72 inch).

### ISO A Series
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

### ISO B Series
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

### ISO C Series (Envelope Sizes)
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

### ISO RA / SRA Series (Raw/Supplementary Raw)
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

### North American Sizes
| Size | Width (pt) | Height (pt) |
|------|-----------|-----------|
| Executive | 521.86 | 756.0 |
| Folio | 612.0 | 936.0 |
| Legal | 612.0 | 1008.0 |
| Letter | 612.0 | 792.0 |
| Tabloid | 792.0 | 1224.0 |

---

## 7. PDFPage API

### 7.1 Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this page belongs to |
| `node` | `PDFPageLeaf` | Low-level PDF dictionary |
| `ref` | `PDFRef` | Unique reference within the document |

### 7.2 Dimension Methods

```typescript
getSize(): { width: number; height: number }
setSize(width: number, height: number): void
getWidth(): number
setWidth(width: number): void
getHeight(): number
setHeight(height: number): void
```

### 7.3 Box Methods

Each returns/sets `{ x: number, y: number, width: number, height: number }`:

```typescript
getMediaBox(): { x: number; y: number; width: number; height: number }
setMediaBox(x: number, y: number, width: number, height: number): void

getCropBox(): { x: number; y: number; width: number; height: number }
setCropBox(x: number, y: number, width: number, height: number): void

getBleedBox(): { x: number; y: number; width: number; height: number }
setBleedBox(x: number, y: number, width: number, height: number): void

getTrimBox(): { x: number; y: number; width: number; height: number }
setTrimBox(x: number, y: number, width: number, height: number): void

getArtBox(): { x: number; y: number; width: number; height: number }
setArtBox(x: number, y: number, width: number, height: number): void
```

### 7.4 Rotation

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

### 7.5 Position / Cursor Methods

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

### 7.6 Font & Text Styling

```typescript
setFont(font: PDFFont): void
setFontSize(fontSize: number): void
setFontColor(fontColor: Color): void
setLineHeight(lineHeight: number): void
```

### 7.7 Drawing Methods

#### drawText
```typescript
drawText(text: string, options?: PDFPageDrawTextOptions): void
```

**PDFPageDrawTextOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `size` | `number` | Font size |
| `font` | `PDFFont` | Font to use |
| `color` | `Color` | Text color |
| `opacity` | `number` | 0-1 opacity |
| `rotate` | `Rotation` | Rotation angle |
| `lineHeight` | `number` | Line height for multiline text |
| `maxWidth` | `number` | Max width for line wrapping |
| `wordBreaks` | `string[]` | Characters to break on |
| `blendMode` | `BlendMode` | Color blending mode |

```typescript
page.drawText('Hello World!', {
  x: 50,
  y: 700,
  size: 30,
  font: timesRomanFont,
  color: rgb(0, 0.53, 0.71),
  rotate: degrees(-45),
  opacity: 0.75,
});
```

#### drawImage
```typescript
drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

**PDFPageDrawImageOptions**:
| Property | Type | Description |
|----------|------|-------------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `width` | `number` | Display width |
| `height` | `number` | Display height |
| `rotate` | `Rotation` | Rotation angle |
| `opacity` | `number` | 0-1 opacity |
| `blendMode` | `BlendMode` | Color blending mode |
| `xSkew` | `Rotation` | Horizontal skew |
| `ySkew` | `Rotation` | Vertical skew |

```typescript
const image = await pdfDoc.embedPng(pngBytes);
const dims = image.scale(0.5);
page.drawImage(image, {
  x: 50,
  y: 100,
  width: dims.width,
  height: dims.height,
});
```

#### drawRectangle
```typescript
drawRectangle(options?: PDFPageDrawRectangleOptions): void
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

#### drawCircle
```typescript
drawCircle(options?: PDFPageDrawCircleOptions): void
```
Options: `x`, `y`, `size` (radius), `color`, `borderColor`, `borderWidth`, `opacity`, `borderOpacity`, `blendMode`, `borderDashArray`, `borderDashPhase`, `borderLineCap`.

#### drawEllipse
```typescript
drawEllipse(options?: PDFPageDrawEllipseOptions): void
```
Options: `x`, `y`, `xScale`, `yScale`, `color`, `borderColor`, `borderWidth`, `rotate`, `opacity`, `borderOpacity`, `blendMode`.

#### drawSquare
```typescript
drawSquare(options?: PDFPageDrawSquareOptions): void
```
Options: `x`, `y`, `size`, `color`, `borderColor`, `borderWidth`, `rotate`, `opacity`, `borderOpacity`, `blendMode`, `borderDashArray`, `borderDashPhase`, `borderLineCap`.

#### drawLine
```typescript
drawLine(options: PDFPageDrawLineOptions): void
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

#### drawSvgPath
```typescript
drawSvgPath(path: string, options?: PDFPageDrawSVGOptions): void
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

#### drawPage
```typescript
drawPage(embeddedPage: PDFEmbeddedPage, options?: PDFPageDrawPageOptions): void
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

### 7.8 Content Transformation

```typescript
translateContent(x: number, y: number): void    // Move page content
scale(x: number, y: number): void                // Scale size + content + annotations
scaleContent(x: number, y: number): void         // Scale content only
scaleAnnotations(x: number, y: number): void     // Scale annotations only
```

### 7.9 Low-Level Operators

```typescript
pushOperators(...operator: PDFOperator[]): void
```

---

## 8. Document Metadata

### 8.1 Getters

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

### 8.2 Setters

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

---

## 9. Architecture & Key Types

### 9.1 Core Classes

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

### 9.2 Async-First Design

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

### 9.3 Coordinate System

- **Origin**: Bottom-left corner of the page (0, 0)
- **X axis**: Increases to the right
- **Y axis**: Increases upward
- **Units**: PDF points (1 point = 1/72 inch)
- **Typical dimensions**: Letter = 612 x 792 pt, A4 = 595.28 x 841.89 pt

This means to draw text at the "top" of a Letter page, use y ≈ 750 (not 0).

---

## 10. Enums & Utility Functions

### 10.1 StandardFonts Enum

The 14 standard PDF fonts (no embedding required):

| Enum Value | Font Name |
|-----------|-----------|
| `StandardFonts.Courier` | Courier |
| `StandardFonts.CourierBold` | Courier-Bold |
| `StandardFonts.CourierOblique` | Courier-Oblique |
| `StandardFonts.CourierBoldOblique` | Courier-BoldOblique |
| `StandardFonts.Helvetica` | Helvetica |
| `StandardFonts.HelveticaBold` | Helvetica-Bold |
| `StandardFonts.HelveticaOblique` | Helvetica-Oblique |
| `StandardFonts.HelveticaBoldOblique` | Helvetica-BoldOblique |
| `StandardFonts.TimesRoman` | Times-Roman |
| `StandardFonts.TimesRomanBold` | Times-Bold |
| `StandardFonts.TimesRomanItalic` | Times-Italic |
| `StandardFonts.TimesRomanBoldItalic` | Times-BoldItalic |
| `StandardFonts.Symbol` | Symbol |
| `StandardFonts.ZapfDingbats` | ZapfDingbats |

### 10.2 Other Enums

| Enum | Values |
|------|--------|
| `TextAlignment` | `Left`, `Center`, `Right` |
| `LineCapStyle` | `Butt`, `Round`, `Square` |
| `LineJoinStyle` | `Miter`, `Round`, `Bevel` |
| `BlendMode` | Various color blending modes |
| `TextRenderingMode` | Text rendering options |

### 10.3 Color Functions

```typescript
rgb(red: number, green: number, blue: number): RGB
// Values 0-1, NOT 0-255

grayscale(gray: number): Grayscale
// 0 = black, 1 = white

cmyk(cyan: number, magenta: number, yellow: number, key: number): CMYK
// Values 0-1
```

**Color type**: `Grayscale | RGB | CMYK`

### 10.4 Rotation Functions

```typescript
degrees(angle: number): Rotation
radians(angle: number): Rotation
degreesToRadians(degree: number): number
radiansToDegrees(radian: number): number
```

### 10.5 Transformation Matrix

`TransformationMatrix` = `[a, b, c, d, e, f]` (6-element number array)

---

## 11. Font Embedding

### 11.1 Standard Fonts (Sync)

```typescript
const font = pdfDoc.embedStandardFont(StandardFonts.Helvetica);
// No await needed — synchronous
```

### 11.2 Standard Fonts (Async — also works)

```typescript
const font = await pdfDoc.embedFont(StandardFonts.TimesRoman);
```

### 11.3 Custom Fonts (Requires fontkit)

```typescript
import fontkit from '@pdf-lib/fontkit';

pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes);
// fontBytes: Uint8Array | ArrayBuffer | string (base64)
```

### 11.4 Font Measurement

```typescript
font.widthOfTextAtSize(text: string, size: number): number
font.heightAtSize(size: number): number
```

---

## 12. Image Embedding

```typescript
const jpgImage = await pdfDoc.embedJpg(jpgBytes);
const pngImage = await pdfDoc.embedPng(pngBytes);
// Input: string (base64) | Uint8Array | ArrayBuffer
```

### Image Scaling

```typescript
const dims = image.scale(0.5);  // Returns { width, height }
// Use dims in drawImage
page.drawImage(image, {
  x: 50,
  y: 100,
  width: dims.width,
  height: dims.height,
});
```

---

## 13. Attachments

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

**Important note from official docs**: "Only some PDF readers can view attachments. This includes Adobe Reader, Foxit Reader, and Firefox."

---

## 14. JavaScript Actions

```typescript
pdfDoc.addJavaScript(name: string, script: string): void
```

Adds named JavaScript that executes on document open. The name must be unique per document.

```typescript
pdfDoc.addJavaScript('onOpen', 'app.alert("Hello from pdf-lib!")');
```

---

## 15. Forms API (Overview)

### Getting the Form

```typescript
const form = pdfDoc.getForm(): PDFForm
```

### Creating Fields

```typescript
form.createTextField(name: string): PDFTextField
form.createCheckBox(name: string): PDFCheckBox
form.createRadioGroup(name: string): PDFRadioGroup
form.createDropdown(name: string): PDFDropdown
form.createOptionList(name: string): PDFOptionList
```

### Getting Existing Fields

```typescript
form.getTextField(name: string): PDFTextField
form.getCheckBox(name: string): PDFCheckBox
form.getRadioGroup(name: string): PDFRadioGroup
form.getDropdown(name: string): PDFDropdown
form.getOptionList(name: string): PDFOptionList
form.getButton(name: string): PDFButton
form.getFields(): PDFField[]
form.getFieldMaybe(name: string): PDFField | undefined
```

### Field Operations

```typescript
// Text fields
field.setText(text: string): void
field.addToPage(page: PDFPage, options?: { x, y }): void

// Checkboxes
checkbox.check(): void
checkbox.uncheck(): void
checkbox.addToPage(page: PDFPage, options?: { x, y }): void

// Radio groups
radioGroup.addOptionToPage(option: string, page: PDFPage, options?: { x, y }): void
radioGroup.select(option: string): void

// Dropdowns
dropdown.addOptions(options: string[]): void
dropdown.select(option: string): void
dropdown.addToPage(page: PDFPage, options?: { x, y }): void

// Option lists
list.addOptions(options: string[]): void
list.select(option: string): void
list.addToPage(page: PDFPage, options?: { x, y }): void

// Buttons
button.setImage(image: PDFImage): void
```

### Flattening

```typescript
form.flatten(options?: { updateFieldAppearances: boolean }): void
```
Converts all form fields to static content (non-editable).

---

## 16. Text Layout Utilities

```typescript
layoutSinglelineText(text: string, options): TextLayout
layoutMultilineText(text: string, options): TextLayout
layoutCombedText(text: string, options): TextLayout
computeFontSize(lines: string[], font: PDFFont, bounds): number
computeCombedFontSize(line: string, font: PDFFont, bounds, cellCount: number): number
```

---

## 17. Gotchas & Important Notes

1. **Coordinate system is bottom-left origin** — Y increases upward. This catches many developers who expect top-left origin.

2. **Color values are 0-1, NOT 0-255** — `rgb(1, 0, 0)` is red, not `rgb(255, 0, 0)`.

3. **Async-first design** — Almost all embedding and I/O operations are async. Forgetting `await` is a common bug.

4. **`embedStandardFont()` is sync** — Unlike `embedFont()`, standard font embedding does not need `await`.

5. **Custom fonts require fontkit** — You must `npm install @pdf-lib/fontkit` and call `pdfDoc.registerFontkit(fontkit)` before embedding custom fonts.

6. **`copy()` doesn't copy everything** — The `copy()` method does NOT copy acroforms, outlines, etc.

7. **`updateMetadata` defaults to `true`** — Both `create()` and `load()` automatically set producer, creator, and dates. Pass `{ updateMetadata: false }` to prevent this.

8. **Encrypted PDFs** — By default, loading encrypted PDFs throws. Use `{ ignoreEncryption: true }` to load them (but you cannot decrypt content).

9. **Page rotation** — Only multiples of 90 degrees are valid.

10. **Attachments viewer support** — Only Adobe Reader, Foxit Reader, and Firefox can view PDF attachments.

11. **`save()` auto-updates form appearances** — Pass `{ updateFieldAppearances: false }` if you don't want this.

12. **`addDefaultPage` in save** — If the document has no pages, `save()` adds a blank page by default. Pass `{ addDefaultPage: false }` to prevent this.

13. **Page copying workflow** — When copying pages between documents, you must: (1) call `copyPages()` on the destination, (2) then `addPage()` or `insertPage()` each copied page. `copyPages()` alone does not add them.

14. **Keywords setter takes an array** — `setKeywords(['a', 'b'])` takes `string[]`, not a single string. But `getKeywords()` returns `string | undefined` (a single comma-separated string).
