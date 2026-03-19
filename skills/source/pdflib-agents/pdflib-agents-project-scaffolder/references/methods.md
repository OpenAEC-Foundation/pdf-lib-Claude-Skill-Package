# Setup and Configuration API Reference

## PDFDocument — Creation and Loading

### PDFDocument.create()

```typescript
static async create(options?: CreateOptions): Promise<PDFDocument>
```

**CreateOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, creation/modification dates |

```typescript
// Standard creation
const pdfDoc = await PDFDocument.create();

// Without auto-metadata
const pdfDoc = await PDFDocument.create({ updateMetadata: false });
```

### PDFDocument.load()

```typescript
static async load(
  pdf: string | Uint8Array | ArrayBuffer,
  options?: LoadOptions
): Promise<PDFDocument>
```

**Input types:**
- `string` — Base64-encoded string or data URI
- `Uint8Array` — Raw PDF bytes
- `ArrayBuffer` — Raw PDF bytes

**LoadOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ignoreEncryption` | `boolean` | `false` | Skip encryption validation |
| `parseSpeed` | `number` | — | Parsing performance level |
| `throwOnInvalidObject` | `boolean` | `false` | Throw on malformed PDF objects |
| `updateMetadata` | `boolean` | `true` | Auto-update metadata |
| `capNumbers` | `boolean` | `false` | Limit numeric precision |

```typescript
// Load from bytes
const pdfDoc = await PDFDocument.load(existingPdfBytes);

// Load encrypted PDF
const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,
});

// Load from base64
const pdfDoc = await PDFDocument.load(base64String);

// Load without updating metadata
const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  updateMetadata: false,
});
```

---

## Font Registration

### registerFontkit()

```typescript
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

MUST be called BEFORE `embedFont()` with custom font bytes. NOT needed for StandardFonts.

```typescript
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);
```

### embedFont()

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

**EmbedFontOptions:**

| Property | Type | Description |
|----------|------|-------------|
| `subset` | `boolean` | Include only used glyphs (reduces file size) |
| `customName` | `string` | Custom name for the embedded font |
| `features` | `TypeFeatures` | OpenType font feature configuration |

```typescript
// Standard font
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica);

// Custom font from bytes
const customFont = await pdfDoc.embedFont(fontBytes, { subset: true });

// Custom font from base64
const customFont = await pdfDoc.embedFont(base64FontString);
```

### embedStandardFont() — Synchronous Alternative

```typescript
pdfDoc.embedStandardFont(font: StandardFonts): PDFFont
```

The ONLY synchronous embed method. NEVER use `await` with this method.

```typescript
const courier = pdfDoc.embedStandardFont(StandardFonts.Courier);
```

---

## Save Methods

### save()

```typescript
pdfDoc.save(options?: SaveOptions): Promise<Uint8Array>
```

**SaveOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `useObjectStreams` | `boolean` | `true` | Enable object stream compression |
| `addDefaultPage` | `boolean` | `true` | Insert blank page if document is empty |
| `objectsPerTick` | `number` | `50` | Processing batch size per event loop tick |
| `updateFieldAppearances` | `boolean` | `true` | Refresh form field visual renderings |

```typescript
const pdfBytes = await pdfDoc.save();

// Without object streams (more compatible)
const pdfBytes = await pdfDoc.save({ useObjectStreams: false });
```

### saveAsBase64()

```typescript
pdfDoc.saveAsBase64(options?: Base64SaveOptions): Promise<string>
```

Extends SaveOptions with:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dataUri` | `boolean` | `false` | Return `data:application/pdf;base64,...` format |

```typescript
const base64 = await pdfDoc.saveAsBase64();
const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });
```

---

## Page Creation

### addPage()

```typescript
pdfDoc.addPage(page?: PDFPage | [number, number]): PDFPage
```

```typescript
// Default size
const page = pdfDoc.addPage();

// Custom size in PDF points
const page = pdfDoc.addPage([612, 792]);

// Using PageSizes enum
const page = pdfDoc.addPage(PageSizes.A4);
const page = pdfDoc.addPage(PageSizes.Letter);
```

### insertPage()

```typescript
pdfDoc.insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
```

```typescript
const page = pdfDoc.insertPage(0); // Insert at beginning
const page = pdfDoc.insertPage(2, PageSizes.A4); // Insert at index 2
```

---

## Document Metadata

```typescript
pdfDoc.setTitle(title: string, options?: { showInWindowTitleBar?: boolean }): void
pdfDoc.setAuthor(author: string): void
pdfDoc.setSubject(subject: string): void
pdfDoc.setKeywords(keywords: string[]): void
pdfDoc.setCreator(creator: string): void
pdfDoc.setProducer(producer: string): void
pdfDoc.setCreationDate(date: Date): void
pdfDoc.setModificationDate(date: Date): void
pdfDoc.setLanguage(language: string): void  // RFC 3066 language tags
```

```typescript
pdfDoc.setTitle('Invoice #12345', { showInWindowTitleBar: true });
pdfDoc.setAuthor('My Application');
pdfDoc.setSubject('Monthly Invoice');
pdfDoc.setKeywords(['invoice', 'billing', 'pdf']);
pdfDoc.setCreator('My App v1.0');
pdfDoc.setLanguage('en-US');
```

---

## Standard Imports Checklist

```typescript
// Core (ALWAYS needed)
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

// Page sizes (when using named sizes like A4, Letter)
import { PageSizes } from 'pdf-lib';

// Rotation (when rotating text, images, or pages)
import { degrees, radians } from 'pdf-lib';

// Additional color functions
import { cmyk, grayscale } from 'pdf-lib';

// Enums (when needed)
import { BlendMode, TextAlignment, LineCapStyle } from 'pdf-lib';

// Custom fonts (ONLY when using non-standard fonts)
import fontkit from '@pdf-lib/fontkit';
```

---

## StandardFonts Enum — Complete List

| Enum Key | Family |
|----------|--------|
| `StandardFonts.Courier` | Monospace |
| `StandardFonts.CourierBold` | Monospace |
| `StandardFonts.CourierOblique` | Monospace |
| `StandardFonts.CourierBoldOblique` | Monospace |
| `StandardFonts.Helvetica` | Sans-serif |
| `StandardFonts.HelveticaBold` | Sans-serif |
| `StandardFonts.HelveticaOblique` | Sans-serif |
| `StandardFonts.HelveticaBoldOblique` | Sans-serif |
| `StandardFonts.TimesRoman` | Serif |
| `StandardFonts.TimesRomanBold` | Serif |
| `StandardFonts.TimesRomanItalic` | Serif |
| `StandardFonts.TimesRomanBoldItalic` | Serif |
| `StandardFonts.Symbol` | Symbol |
| `StandardFonts.ZapfDingbats` | Dingbats |

> StandardFonts ONLY support WinAnsi encoding (Latin-1 subset). For Unicode text, ALWAYS use custom fonts with fontkit.

---

## Common PageSizes

| Name | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes.A4` | 595.28 | 841.89 |
| `PageSizes.Letter` | 612 | 792 |
| `PageSizes.Legal` | 612 | 1008 |
| `PageSizes.A3` | 841.89 | 1190.55 |
| `PageSizes.A5` | 419.53 | 595.28 |
| `PageSizes.Tabloid` | 792 | 1224 |

All dimensions in PDF points (1 point = 1/72 inch).

---

## Platform-Specific Dependencies

### Node.js

```json
{
  "dependencies": {
    "pdf-lib": "^1.17.1"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.0.0"
  }
}
```

### Node.js with Custom Fonts

```json
{
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

### Browser (CDN)

```html
<!-- Production: ALWAYS pin the version -->
<script src="https://unpkg.com/pdf-lib@1.17.1/dist/pdf-lib.min.js"></script>

<!-- With fontkit -->
<script src="https://unpkg.com/@pdf-lib/fontkit@1.1.1/dist/fontkit.umd.min.js"></script>
```

### Browser (Bundler)

```json
{
  "dependencies": {
    "pdf-lib": "^1.17.1"
  }
}
```
