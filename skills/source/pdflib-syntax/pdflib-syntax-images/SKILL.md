---
name: pdflib-syntax-images
description: >
  Use when embedding or drawing images in pdf-lib PDFs.
  Prevents the common mistake of using unsupported image formats (only PNG and JPG
  are supported — no SVG, GIF, or WebP). Covers embedPng, embedJpg, drawImage,
  image scaling, aspect ratio preservation.
  Keywords: embedPng, embedJpg, drawImage, PDFImage, scale, scaleToFit, image, PNG, JPG.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-syntax-images — Image Embedding & Drawing

## Quick Reference

```typescript
import { PDFDocument, degrees } from 'pdf-lib';

// Embed images (ONLY PNG and JPG supported)
const pngImage: PDFImage = await pdfDoc.embedPng(pngBytes);
const jpgImage: PDFImage = await pdfDoc.embedJpg(jpgBytes);

// Get dimensions
const { width, height } = pngImage.size();       // original pixels
const dims = pngImage.scale(0.5);                 // uniform scale
const fitted = pngImage.scaleToFit(300, 200);     // fit within bounds

// Draw on page
page.drawImage(pngImage, {
  x: 50, y: 50,
  width: dims.width, height: dims.height,
});

// Embed and draw pages from other PDFs
const [embeddedPage] = await pdfDoc.embedPdf(otherPdfBytes);
page.drawPage(embeddedPage, { x: 50, y: 50, width: 300, height: 400 });
```

## Critical Warnings

> **ONLY PNG and JPG are supported.** There is NO `embedGif()`, `embedBmp()`,
> `embedTiff()`, `embedWebP()`, `embedSvg()`, or generic `embedImage()` method.
> ALWAYS convert other formats to PNG or JPG before embedding.

> **All embed methods are async.** ALWAYS `await` the result of `embedPng()`,
> `embedJpg()`, `embedPdf()`, `embedPage()`, and `embedPages()`.

> **drawImage() and drawPage() are synchronous.** NEVER `await` these calls.

> **Coordinate origin is bottom-left.** `y: 0` is the bottom of the page,
> NOT the top. To place an image near the top, use `y: pageHeight - imageHeight - margin`.

## Format Support Matrix

| Format | Method | Supported |
|--------|--------|-----------|
| PNG | `embedPng()` | YES |
| JPEG/JPG | `embedJpg()` | YES |
| GIF | — | NO |
| BMP | — | NO |
| TIFF | — | NO |
| WebP | — | NO |
| SVG | — | NO |
| AVIF | — | NO |
| HEIC | — | NO |

## Embedding Methods

### embedPng()

```typescript
pdfDoc.embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64-encoded PNG or data URI (`data:image/png;base64,...`)
- `Uint8Array` — raw PNG bytes
- `ArrayBuffer` — raw PNG bytes

### embedJpg()

```typescript
pdfDoc.embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64-encoded JPEG or data URI (`data:image/jpeg;base64,...`)
- `Uint8Array` — raw JPEG bytes
- `ArrayBuffer` — raw JPEG bytes

## PDFImage API

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `width` | `number` | Original width in pixels |
| `height` | `number` | Original height in pixels |
| `ref` | `PDFRef` | Unique reference within the document |
| `doc` | `PDFDocument` | The document this image belongs to |

### Methods

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `scale` | `(factor: number)` | `{ width: number, height: number }` | Scale by uniform factor. Preserves aspect ratio. |
| `scaleToFit` | `(width: number, height: number)` | `{ width: number, height: number }` | Scale to fit within bounds. Preserves aspect ratio. |
| `size` | `()` | `{ width: number, height: number }` | Returns original dimensions. |
| `embed` | `()` | `Promise<void>` | Embed in document. Called automatically by `save()`. |

## drawImage() Options

```typescript
page.drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `x` | `number` | `0` | Horizontal position (bottom-left origin) |
| `y` | `number` | `0` | Vertical position (bottom-left origin) |
| `width` | `number` | original width | Display width in PDF points |
| `height` | `number` | original height | Display height in PDF points |
| `rotate` | `Rotation` | — | Rotation via `degrees()` or `radians()` |
| `xSkew` | `Rotation` | — | Horizontal skew |
| `ySkew` | `Rotation` | — | Vertical skew |
| `opacity` | `number` | `1.0` | Transparency 0.0 (invisible) to 1.0 (opaque) |
| `blendMode` | `BlendMode` | `Normal` | Color blending method |

## Aspect Ratio Preservation — Three Patterns

### Pattern 1: Uniform Scale Factor

ALWAYS use `image.scale()` when you want a percentage of the original size.

```typescript
const image = await pdfDoc.embedPng(pngBytes);
const dims = image.scale(0.5); // 50% of original
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});
```

### Pattern 2: Fit Within Bounds

ALWAYS use `image.scaleToFit()` when you have a maximum area and want the
image to fill it without distortion.

```typescript
const image = await pdfDoc.embedJpg(jpgBytes);
const dims = image.scaleToFit(300, 200); // max 300x200, aspect ratio preserved
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});
```

### Pattern 3: Manual Calculation (Target Width)

Use when you need a specific width and want the height to follow.

```typescript
const image = await pdfDoc.embedPng(pngBytes);
const targetWidth = 300;
const scaleFactor = targetWidth / image.width;
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: image.height * scaleFactor,
});
```

## Embedding Pages as Images

Use `embedPdf()`, `embedPage()`, or `embedPages()` to embed pages from
other PDFs, then draw them with `drawPage()`.

### embedPdf()

```typescript
pdfDoc.embedPdf(
  pdf: string | Uint8Array | ArrayBuffer | PDFDocument,
  indices?: number[]
): Promise<PDFEmbeddedPage[]>
```

### embedPage()

```typescript
pdfDoc.embedPage(
  page: PDFPage,
  boundingBox?: { left: number; bottom: number; right: number; top: number },
  transformationMatrix?: [number, number, number, number, number, number]
): Promise<PDFEmbeddedPage>
```

### embedPages()

```typescript
pdfDoc.embedPages(
  pages: PDFPage[],
  boundingBoxes?: { left: number; bottom: number; right: number; top: number }[],
  transformationMatrices?: [number, number, number, number, number, number][]
): Promise<PDFEmbeddedPage[]>
```

### drawPage()

```typescript
page.drawPage(embeddedPage: PDFEmbeddedPage, options?: PDFPageDrawPageOptions): void
```

Options: `x`, `y`, `width`, `height`, `rotate`, `opacity`, `xSkew`, `ySkew`, `blendMode`.

## Decision Tree

```
Need to add an image to a PDF?
+-- What format is the image?
|   +-- PNG -> embedPng()
|   +-- JPEG/JPG -> embedJpg()
|   +-- Other format -> Convert to PNG or JPG first, then embed
|
+-- How should the image be sized?
|   +-- Percentage of original -> image.scale(factor)
|   +-- Fit within max bounds -> image.scaleToFit(maxW, maxH)
|   +-- Exact target width -> manual: height = image.height * (targetW / image.width)
|   +-- Exact target height -> manual: width = image.width * (targetH / image.height)
|   +-- NEVER set arbitrary width AND height -> causes distortion
|
+-- Need to embed a page from another PDF?
    +-- From raw PDF bytes -> embedPdf(bytes, [pageIndex])
    +-- From PDFPage object -> embedPage(page)
    +-- Multiple pages -> embedPages(pages)
    +-- Then draw with -> drawPage(embeddedPage, options)
```

## Reference Links

- [Method signatures and PDFImage API](references/methods.md)
- [Complete code examples](references/examples.md)
- [Anti-patterns and common mistakes](references/anti-patterns.md)
- [pdf-lib API docs — PDFDocument](https://pdf-lib.js.org/docs/api/classes/pdfdocument)
- [pdf-lib API docs — PDFPage](https://pdf-lib.js.org/docs/api/classes/pdfpage)
- [pdf-lib GitHub README](https://github.com/Hopding/pdf-lib)
