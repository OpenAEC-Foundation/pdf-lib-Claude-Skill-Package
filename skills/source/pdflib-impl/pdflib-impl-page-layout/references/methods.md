# Positioning and Measurement API Methods

## Page Dimension Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `page.getSize()` | `{ width: number; height: number }` | Get page dimensions in PDF points |
| `page.getWidth()` | `number` | Get page width in PDF points |
| `page.getHeight()` | `number` | Get page height in PDF points |
| `page.setSize(w, h)` | `void` | Set page dimensions |
| `page.setWidth(w)` | `void` | Set page width |
| `page.setHeight(h)` | `void` | Set page height |

## Page Position/Cursor Methods

pdf-lib provides built-in cursor methods on PDFPage. These track an internal position used by some drawing operations (primarily `drawSvgPath` when no x/y is specified).

| Method | Returns | Description |
|--------|---------|-------------|
| `page.getPosition()` | `{ x: number; y: number }` | Get current internal cursor position |
| `page.getX()` | `number` | Get current X position |
| `page.getY()` | `number` | Get current Y position |
| `page.moveTo(x, y)` | `void` | Set internal cursor to absolute position |
| `page.moveUp(amount)` | `void` | Move cursor up (increases Y) |
| `page.moveDown(amount)` | `void` | Move cursor down (decreases Y) |
| `page.moveLeft(amount)` | `void` | Move cursor left (decreases X) |
| `page.moveRight(amount)` | `void` | Move cursor right (increases X) |
| `page.resetPosition()` | `void` | Reset cursor to (0, 0) |

**Important**: These cursor methods affect the page's internal position state. For layout purposes, ALWAYS use your own `cursorY` variable rather than relying on these methods. The internal cursor is primarily useful for sequential SVG path drawing.

## Font Measurement Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `font.widthOfTextAtSize(text, size)` | `number` | Width of text string at given font size (PDF points) |
| `font.heightAtSize(size, options?)` | `number` | Height of font at given size (PDF points) |
| `font.sizeAtHeight(height)` | `number` | Compute font size needed to achieve target height |

### heightAtSize Options

```typescript
font.heightAtSize(fontSize); // Default: includes descender
font.heightAtSize(fontSize, { descender: false }); // Excludes descender
```

The `descender` option (default `true`) controls whether the descender portion of the font (the part below the baseline, e.g., the tail of "g" or "p") is included in the height calculation.

## Image Measurement Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `image.width` | `number` | Original image width in pixels |
| `image.height` | `number` | Original image height in pixels |
| `image.scale(factor)` | `{ width: number; height: number }` | Scale dimensions by factor |
| `image.scaleToFit(w, h)` | `{ width: number; height: number }` | Scale to fit within bounds, preserving aspect ratio |
| `image.size()` | `{ width: number; height: number }` | Get original dimensions |

## drawText Layout Options

These options on `page.drawText()` directly affect layout:

| Option | Type | Effect |
|--------|------|--------|
| `x` | `number` | Horizontal position (default: 0, bottom-left origin) |
| `y` | `number` | Vertical position (default: 0, bottom-left origin) |
| `maxWidth` | `number` | Triggers automatic word wrapping at this width |
| `lineHeight` | `number` | Vertical spacing between lines (for multiline/wrapped text) |
| `wordBreaks` | `string[]` | Custom word break characters (default: `[' ']`) |

## Page-Level Defaults

Set defaults that apply to all subsequent `drawText()` calls on a page:

```typescript
page.setFont(font);           // Default font
page.setFontSize(12);         // Default font size
page.setFontColor(rgb(0,0,0)); // Default text color
page.setLineHeight(16);       // Default line height
```

Individual `drawText()` calls can still override these defaults via their options.

## Content Transformation Methods

| Method | Description |
|--------|-------------|
| `page.translateContent(x, y)` | Move all existing page content by (x, y) offset |
| `page.scale(x, y)` | Scale page size + content + annotations |
| `page.scaleContent(x, y)` | Scale content only (not page dimensions) |
| `page.scaleAnnotations(x, y)` | Scale annotations only |

## Unit Conversion Helpers

pdf-lib uses PDF points (1 point = 1/72 inch). ALWAYS convert when working with other units.

```typescript
// Conversion constants
const PT_PER_INCH = 72;
const PT_PER_MM = 72 / 25.4; // ≈ 2.8346
const PT_PER_CM = 72 / 2.54; // ≈ 28.3465

// Helper functions
function inchesToPt(inches: number): number {
  return inches * 72;
}

function mmToPt(mm: number): number {
  return mm * (72 / 25.4);
}

function cmToPt(cm: number): number {
  return cm * (72 / 2.54);
}
```

## PageSizes Enum (Common Sizes)

```typescript
import { PageSizes } from 'pdf-lib';

// Usage
const page = pdfDoc.addPage(PageSizes.A4);     // 595.28 x 841.89
const page = pdfDoc.addPage(PageSizes.Letter);  // 612 x 792
const page = pdfDoc.addPage(PageSizes.Legal);   // 612 x 1008
const page = pdfDoc.addPage(PageSizes.A3);      // 841.89 x 1190.55

// Custom size (width x height in PDF points)
const page = pdfDoc.addPage([500, 700]);
```

ALWAYS specify page size explicitly when creating documents for a specific paper format. NEVER rely on the default page size for production documents.
