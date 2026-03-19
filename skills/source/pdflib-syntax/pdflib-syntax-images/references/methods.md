# pdflib-syntax-images — Method Reference

## embedPng()

```typescript
pdfDoc.embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

Embeds a PNG image into the document. Returns a `PDFImage` that can be drawn
on any page in the document.

**Input types:**
| Type | Description |
|------|-------------|
| `string` | Base64-encoded PNG data or a data URI (`data:image/png;base64,...`) |
| `Uint8Array` | Raw PNG file bytes |
| `ArrayBuffer` | Raw PNG file bytes |

**Behavior:**
- ALWAYS returns a Promise — ALWAYS use `await`
- PNG transparency (alpha channel) is preserved
- The returned `PDFImage` stores the original pixel dimensions

## embedJpg()

```typescript
pdfDoc.embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

Embeds a JPEG/JPG image into the document. Returns a `PDFImage` that can be
drawn on any page in the document.

**Input types:**
| Type | Description |
|------|-------------|
| `string` | Base64-encoded JPEG data or a data URI (`data:image/jpeg;base64,...`) |
| `Uint8Array` | Raw JPEG file bytes |
| `ArrayBuffer` | Raw JPEG file bytes |

**Behavior:**
- ALWAYS returns a Promise — ALWAYS use `await`
- JPEG does NOT support transparency
- The returned `PDFImage` stores the original pixel dimensions

## PDFImage Class

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `width` | `number` | Original image width in pixels |
| `height` | `number` | Original image height in pixels |
| `ref` | `PDFRef` | Unique reference identifier within the PDF document |
| `doc` | `PDFDocument` | The document this image belongs to |

### scale(factor)

```typescript
scale(factor: number): { width: number; height: number }
```

Scales the image dimensions by a uniform factor. ALWAYS preserves aspect ratio.

- `factor = 1.0` returns original dimensions
- `factor = 0.5` returns half-size dimensions
- `factor = 2.0` returns double-size dimensions

**Example:** A 1000x500 image with `scale(0.25)` returns `{ width: 250, height: 125 }`.

Does NOT modify the image — returns new dimension values for use in `drawImage()`.

### scaleToFit(width, height)

```typescript
scaleToFit(width: number, height: number): { width: number; height: number }
```

Scales the image to fit within the given bounds while ALWAYS preserving aspect ratio.
The returned dimensions will NEVER exceed the specified width or height.

**Example:** A 1000x500 image with `scaleToFit(300, 200)` returns `{ width: 300, height: 150 }`
because the width constraint (300/1000 = 0.3) is more restrictive than the height constraint
(200/500 = 0.4).

Does NOT modify the image — returns new dimension values for use in `drawImage()`.

### size()

```typescript
size(): { width: number; height: number }
```

Returns the original dimensions of the image as an object.
Equivalent to `{ width: image.width, height: image.height }`.

### embed()

```typescript
embed(): Promise<void>
```

Embeds the image data into the PDF document context. This is called automatically
by `pdfDoc.save()` — you NEVER need to call this manually.

## drawImage()

```typescript
page.drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

Draws an embedded image on the page. This is a synchronous operation — NEVER `await` it.

### PDFPageDrawImageOptions

All properties are optional.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate (from left edge) |
| `y` | `number` | `0` | Y coordinate (from bottom edge) |
| `width` | `number` | `image.width` | Display width in PDF points |
| `height` | `number` | `image.height` | Display height in PDF points |
| `rotate` | `Rotation` | none | Rotation angle via `degrees()` or `radians()` |
| `xSkew` | `Rotation` | none | Horizontal skew transformation |
| `ySkew` | `Rotation` | none | Vertical skew transformation |
| `opacity` | `number` | `1.0` | Opacity: 0.0 = invisible, 1.0 = fully opaque |
| `blendMode` | `BlendMode` | `Normal` | PDF blend mode for compositing |

**Coordinate system:** The position `(x, y)` specifies the bottom-left corner of the
drawn image. The origin `(0, 0)` is at the bottom-left of the page.

**Width/height units:** PDF points (1 point = 1/72 inch). When omitted, the original
pixel dimensions are used as point values (1 pixel = 1 point).

## embedPdf()

```typescript
pdfDoc.embedPdf(
  pdf: string | Uint8Array | ArrayBuffer | PDFDocument,
  indices?: number[]
): Promise<PDFEmbeddedPage[]>
```

Embeds pages from another PDF as drawable objects. If `indices` is omitted,
embeds the first page only.

**Input types:**
| Type | Description |
|------|-------------|
| `string` | Base64-encoded PDF data |
| `Uint8Array` | Raw PDF bytes |
| `ArrayBuffer` | Raw PDF bytes |
| `PDFDocument` | An already-loaded PDFDocument instance |

## embedPage()

```typescript
pdfDoc.embedPage(
  page: PDFPage,
  boundingBox?: { left: number; bottom: number; right: number; top: number },
  transformationMatrix?: [number, number, number, number, number, number]
): Promise<PDFEmbeddedPage>
```

Embeds a single page with optional clipping (boundingBox) and transformation.

## embedPages()

```typescript
pdfDoc.embedPages(
  pages: PDFPage[],
  boundingBoxes?: { left: number; bottom: number; right: number; top: number }[],
  transformationMatrices?: [number, number, number, number, number, number][]
): Promise<PDFEmbeddedPage[]>
```

Embeds multiple pages at once. Each page can have its own bounding box and
transformation matrix.

## drawPage()

```typescript
page.drawPage(embeddedPage: PDFEmbeddedPage, options?: PDFPageDrawPageOptions): void
```

Draws an embedded page on the current page. Synchronous — NEVER `await` this.

### PDFPageDrawPageOptions

All properties are optional.

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | `number` | `0` | X coordinate |
| `y` | `number` | `0` | Y coordinate |
| `width` | `number` | embedded page width | Display width |
| `height` | `number` | embedded page height | Display height |
| `rotate` | `Rotation` | none | Rotation |
| `opacity` | `number` | `1.0` | Opacity |
| `xSkew` | `Rotation` | none | Horizontal skew |
| `ySkew` | `Rotation` | none | Vertical skew |
| `blendMode` | `BlendMode` | `Normal` | Blend mode |
