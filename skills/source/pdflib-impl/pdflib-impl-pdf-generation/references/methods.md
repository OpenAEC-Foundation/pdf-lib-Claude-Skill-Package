# Key API Methods for PDF Generation

## Document Creation

### PDFDocument.create()

```typescript
static async create(options?: CreateOptions): Promise<PDFDocument>
```

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, creation/modification dates |

```typescript
const pdfDoc = await PDFDocument.create();
```

## Page Management

### addPage()

```typescript
addPage(page?: PDFPage | [number, number]): PDFPage
```

- No args: default dimensions
- `[width, height]`: custom size in PDF points
- `PageSizes.A4`: predefined size (595.28 x 841.89 pt)
- `PageSizes.Letter`: predefined size (612 x 792 pt)

```typescript
const page = pdfDoc.addPage(PageSizes.A4);
const customPage = pdfDoc.addPage([500, 700]);
```

### getSize() / getWidth() / getHeight()

```typescript
const { width, height } = page.getSize();
const w = page.getWidth();
const h = page.getHeight();
```

### Common Page Sizes (PDF points)

| Size | Width | Height |
|------|-------|--------|
| A4 | 595.28 | 841.89 |
| A3 | 841.89 | 1190.55 |
| Letter | 612 | 792 |
| Legal | 612 | 1008 |
| Tabloid | 792 | 1224 |

## Font Embedding

### embedFont()

```typescript
async embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions,
): Promise<PDFFont>
```

| Option | Type | Purpose |
|--------|------|---------|
| `subset` | `boolean` | Include only used glyphs (smaller file) |
| `customName` | `string` | Custom name for the font |

```typescript
// Standard font (Latin characters only)
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica);
const bold = await pdfDoc.embedFont(StandardFonts.HelveticaBold);

// Custom font (requires fontkit registration)
import fontkit from '@pdf-lib/fontkit';
pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });
```

### PDFFont Measurement Methods

| Method | Signature | Returns |
|--------|-----------|---------|
| `widthOfTextAtSize` | `(text: string, size: number)` | `number` (width in pt) |
| `heightAtSize` | `(size: number, options?)` | `number` (height in pt) |
| `sizeAtHeight` | `(height: number)` | `number` (font size) |

```typescript
const textWidth = font.widthOfTextAtSize('Hello', 12);
const textHeight = font.heightAtSize(12);
```

## Drawing Methods

### drawText()

```typescript
page.drawText(text: string, options?: PDFPageDrawTextOptions): void
```

| Option | Type | Purpose |
|--------|------|---------|
| `x` | `number` | X position (default: 0) |
| `y` | `number` | Y position (default: 0) |
| `font` | `PDFFont` | Font to use |
| `size` | `number` | Font size in points |
| `color` | `Color` | Text color |
| `opacity` | `number` | 0.0 (invisible) to 1.0 (opaque) |
| `lineHeight` | `number` | Spacing between lines (for `\n` in text) |
| `maxWidth` | `number` | Wraps text at word breaks |
| `rotate` | `Rotation` | Rotation via `degrees()` |

### drawRectangle()

```typescript
page.drawRectangle(options?: PDFPageDrawRectangleOptions): void
```

| Option | Type | Purpose |
|--------|------|---------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `width` | `number` | Rectangle width |
| `height` | `number` | Rectangle height |
| `color` | `Color` | Fill color |
| `borderColor` | `Color` | Border color |
| `borderWidth` | `number` | Border thickness |
| `opacity` | `number` | Fill opacity |

### drawLine()

```typescript
page.drawLine(options: PDFPageDrawLineOptions): void
```

| Option | Type | Purpose |
|--------|------|---------|
| `start` | `{ x: number, y: number }` | Start point |
| `end` | `{ x: number, y: number }` | End point |
| `thickness` | `number` | Line width |
| `color` | `Color` | Line color |
| `dashArray` | `number[]` | Dash pattern (e.g., `[3, 3]`) |

### drawImage()

```typescript
page.drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

| Option | Type | Purpose |
|--------|------|---------|
| `x` | `number` | X position |
| `y` | `number` | Y position |
| `width` | `number` | Display width |
| `height` | `number` | Display height |
| `opacity` | `number` | Transparency |

## Image Embedding

### embedPng() / embedJpg()

```typescript
async embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
async embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

ONLY PNG and JPEG formats are supported. NEVER attempt GIF, BMP, TIFF, WebP, or SVG.

### PDFImage Scaling Methods

```typescript
const dims = image.scale(0.5);           // 50% of original
const fitted = image.scaleToFit(300, 200); // Fit in bounds, keep aspect ratio
const size = image.size();               // Original dimensions
```

## Color Functions

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib';

// RGB — values 0.0 to 1.0 (NOT 0-255)
const black = rgb(0, 0, 0);
const red = rgb(1, 0, 0);
const gray = rgb(0.5, 0.5, 0.5);
const lightGray = rgb(0.9, 0.9, 0.9);

// Grayscale — 0.0 (black) to 1.0 (white)
const grayFill = grayscale(0.5);
```

## Save Methods

### save()

```typescript
async save(options?: SaveOptions): Promise<Uint8Array>
```

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `useObjectStreams` | `boolean` | `true` | Compression (set `false` for wider compatibility) |
| `addDefaultPage` | `boolean` | `true` | Insert blank page if document is empty |

### saveAsBase64()

```typescript
async saveAsBase64(options?: Base64SaveOptions): Promise<string>
```

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `dataUri` | `boolean` | `false` | Prefix with `data:application/pdf;base64,` |

## Metadata Methods

```typescript
pdfDoc.setTitle(title: string, options?: { showInWindowTitleBar?: boolean }): void
pdfDoc.setAuthor(author: string): void
pdfDoc.setSubject(subject: string): void
pdfDoc.setKeywords(keywords: string[]): void
pdfDoc.setCreator(creator: string): void
pdfDoc.setProducer(producer: string): void
pdfDoc.setCreationDate(date: Date): void
pdfDoc.setModificationDate(date: Date): void
pdfDoc.setLanguage(language: string): void  // RFC 3066 tag, e.g., 'en-US'
```
