# pdflib-syntax-pages — Method Signatures Reference

## PDFDocument Page Methods

### addPage

```typescript
addPage(page?: PDFPage | [number, number]): PDFPage
```

Adds a page at the END of the document. Returns the new or added `PDFPage`.

| Argument | Type | Description |
|----------|------|-------------|
| _(none)_ | — | Creates blank page with default dimensions |
| `[width, height]` | `[number, number]` | Creates page with specified dimensions in PDF points |
| `page` | `PDFPage` | Adds an existing PDFPage (MUST be from same document or via `copyPages()`) |

### insertPage

```typescript
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
```

Inserts a page at the specified `index`. Returns the new or inserted `PDFPage`. All pages at or after `index` shift up by one.

| Argument | Type | Description |
|----------|------|-------------|
| `index` | `number` | Zero-based position to insert at |
| `page` | `PDFPage \| [number, number]` | Optional — same as `addPage()` |

### removePage

```typescript
removePage(index: number): void
```

Removes the page at the specified zero-based `index`. All subsequent page indices shift down by one.

### getPage

```typescript
getPage(index: number): PDFPage
```

Returns the `PDFPage` at the specified zero-based `index`. Throws if index is out of bounds.

### getPages

```typescript
getPages(): PDFPage[]
```

Returns an array of all `PDFPage` objects in the document, in order.

### getPageCount

```typescript
getPageCount(): number
```

Returns the total number of pages in the document.

### getPageIndices

```typescript
getPageIndices(): number[]
```

Returns an array of zero-based page indices: `[0, 1, 2, ...]`.

---

## PDFPage Dimension Methods

### getSize

```typescript
getSize(): { width: number; height: number }
```

Returns the page width and height in PDF points.

### setSize

```typescript
setSize(width: number, height: number): void
```

Sets the page width and height in PDF points.

### getWidth / setWidth

```typescript
getWidth(): number
setWidth(width: number): void
```

### getHeight / setHeight

```typescript
getHeight(): number
setHeight(height: number): void
```

---

## PDFPage Box Methods

Each box method pair follows the same signature pattern. All coordinates are in PDF points, with origin at bottom-left.

### MediaBox (Physical Page Size)

```typescript
getMediaBox(): { x: number; y: number; width: number; height: number }
setMediaBox(x: number, y: number, width: number, height: number): void
```

### CropBox (Visible Area)

```typescript
getCropBox(): { x: number; y: number; width: number; height: number }
setCropBox(x: number, y: number, width: number, height: number): void
```

### BleedBox (Printing Bleed Area)

```typescript
getBleedBox(): { x: number; y: number; width: number; height: number }
setBleedBox(x: number, y: number, width: number, height: number): void
```

### TrimBox (Intended Final Size)

```typescript
getTrimBox(): { x: number; y: number; width: number; height: number }
setTrimBox(x: number, y: number, width: number, height: number): void
```

### ArtBox (Extent of Meaningful Content)

```typescript
getArtBox(): { x: number; y: number; width: number; height: number }
setArtBox(x: number, y: number, width: number, height: number): void
```

---

## PDFPage Rotation Methods

### getRotation

```typescript
getRotation(): Rotation
```

Returns the current rotation as a `Rotation` object (use `.angle` to get the numeric degrees value).

### setRotation

```typescript
setRotation(angle: Rotation): void
```

Sets the page rotation. MUST be a multiple of 90 degrees. Use `degrees()` to create the `Rotation` value.

```typescript
import { degrees } from 'pdf-lib';
page.setRotation(degrees(90));
page.setRotation(degrees(180));
page.setRotation(degrees(270));
page.setRotation(degrees(0));   // reset rotation
```

---

## PDFPage Position / Cursor Methods

### getPosition

```typescript
getPosition(): { x: number; y: number }
```

Returns the current cursor position.

### getX / getY

```typescript
getX(): number
getY(): number
```

### moveTo

```typescript
moveTo(x: number, y: number): void
```

Sets cursor to absolute position.

### moveUp / moveDown / moveLeft / moveRight

```typescript
moveUp(yIncrease: number): void
moveDown(yDecrease: number): void
moveLeft(xDecrease: number): void
moveRight(xIncrease: number): void
```

### resetPosition

```typescript
resetPosition(): void
```

Resets the cursor to `(0, 0)` — the bottom-left corner.

---

## PDFPage Content Transformation Methods

### translateContent

```typescript
translateContent(x: number, y: number): void
```

Shifts all existing page content by `(x, y)` points. Positive x = right, positive y = up.

### scale

```typescript
scale(x: number, y: number): void
```

Scales the page size, content, AND annotations by the given factors.

### scaleContent

```typescript
scaleContent(x: number, y: number): void
```

Scales ONLY the drawn content (not page dimensions or annotations).

### scaleAnnotations

```typescript
scaleAnnotations(x: number, y: number): void
```

Scales ONLY the annotations (not page dimensions or content).

---

## PDFPage Default Setters

### setFont

```typescript
setFont(font: PDFFont): void
```

Sets the default font for subsequent `drawText()` calls on this page.

### setFontSize

```typescript
setFontSize(fontSize: number): void
```

Sets the default font size for subsequent `drawText()` calls on this page.

### setFontColor

```typescript
setFontColor(fontColor: Color): void
```

Sets the default font color for subsequent `drawText()` calls on this page. Accepts `rgb()`, `cmyk()`, or `grayscale()`.

### setLineHeight

```typescript
setLineHeight(lineHeight: number): void
```

Sets the default line height for subsequent `drawText()` calls on this page.

---

## PDFPage Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this page belongs to |
| `node` | `PDFPageLeaf` | Low-level PDF page dictionary |
| `ref` | `PDFRef` | Unique reference within the document |

---

## PDFPage Low-Level Method

### pushOperators

```typescript
pushOperators(...operator: PDFOperator[]): void
```

Appends raw PDF operators to the page content stream. For advanced use only.

---

## PageSizes Enum — Complete Reference

All dimensions are `[width, height]` in PDF points (1 point = 1/72 inch).

### ISO A Series

| Size | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes['4A0']` | 4767.87 | 6740.79 |
| `PageSizes['2A0']` | 3370.39 | 4767.87 |
| `PageSizes.A0` | 2383.94 | 3370.39 |
| `PageSizes.A1` | 1683.78 | 2383.94 |
| `PageSizes.A2` | 1190.55 | 1683.78 |
| `PageSizes.A3` | 841.89 | 1190.55 |
| `PageSizes.A4` | 595.28 | 841.89 |
| `PageSizes.A5` | 419.53 | 595.28 |
| `PageSizes.A6` | 297.64 | 419.53 |
| `PageSizes.A7` | 209.76 | 297.64 |
| `PageSizes.A8` | 147.4 | 209.76 |
| `PageSizes.A9` | 104.88 | 147.4 |
| `PageSizes.A10` | 73.7 | 104.88 |

### ISO B Series

| Size | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes.B0` | 2834.65 | 4008.19 |
| `PageSizes.B1` | 2004.09 | 2834.65 |
| `PageSizes.B2` | 1417.32 | 2004.09 |
| `PageSizes.B3` | 1000.63 | 1417.32 |
| `PageSizes.B4` | 708.66 | 1000.63 |
| `PageSizes.B5` | 498.9 | 708.66 |
| `PageSizes.B6` | 354.33 | 498.9 |
| `PageSizes.B7` | 249.45 | 354.33 |
| `PageSizes.B8` | 175.75 | 249.45 |
| `PageSizes.B9` | 124.72 | 175.75 |
| `PageSizes.B10` | 87.87 | 124.72 |

### ISO C Series (Envelope Sizes)

| Size | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes.C0` | 2599.37 | 3676.54 |
| `PageSizes.C1` | 1836.85 | 2599.37 |
| `PageSizes.C2` | 1298.27 | 1836.85 |
| `PageSizes.C3` | 918.43 | 1298.27 |
| `PageSizes.C4` | 649.13 | 918.43 |
| `PageSizes.C5` | 459.21 | 649.13 |
| `PageSizes.C6` | 323.15 | 459.21 |
| `PageSizes.C7` | 229.61 | 323.15 |
| `PageSizes.C8` | 161.57 | 229.61 |
| `PageSizes.C9` | 113.39 | 161.57 |
| `PageSizes.C10` | 79.37 | 113.39 |

### ISO RA / SRA Series (Raw / Supplementary Raw)

| Size | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes.RA0` | 2437.8 | 3458.27 |
| `PageSizes.RA1` | 1729.13 | 2437.8 |
| `PageSizes.RA2` | 1218.9 | 1729.13 |
| `PageSizes.RA3` | 864.57 | 1218.9 |
| `PageSizes.RA4` | 609.45 | 864.57 |
| `PageSizes.SRA0` | 2551.18 | 3628.35 |
| `PageSizes.SRA1` | 1814.17 | 2551.18 |
| `PageSizes.SRA2` | 1275.59 | 1814.17 |
| `PageSizes.SRA3` | 907.09 | 1275.59 |
| `PageSizes.SRA4` | 637.8 | 907.09 |

### North American Sizes

| Size | Width (pt) | Height (pt) |
|------|-----------|------------|
| `PageSizes.Executive` | 521.86 | 756.0 |
| `PageSizes.Folio` | 612.0 | 936.0 |
| `PageSizes.Legal` | 612.0 | 1008.0 |
| `PageSizes.Letter` | 612.0 | 792.0 |
| `PageSizes.Tabloid` | 792.0 | 1224.0 |
