# drawText Options & Text Measurement Methods

## drawText() — Complete Signature

```typescript
page.drawText(text: string, options?: PDFPageDrawTextOptions): void
```

`drawText()` is SYNCHRONOUS — no `await` needed. The font MUST be embedded (async) before calling drawText.

## PDFPageDrawTextOptions — All 13 Properties

All properties are optional. If omitted, page-level defaults are used (set via `setFont()`, `setFontSize()`, etc.).

### Position Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `x` | `number` | 0 | Horizontal position from left edge in PDF points |
| `y` | `number` | 0 | Vertical position from bottom edge in PDF points |

Position specifies the bottom-left corner of the first character's baseline.

### Font & Size Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `font` | `PDFFont` | page default | Font to use — MUST be embedded first |
| `size` | `number` | page default | Font size in PDF points |

### Color & Opacity Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `color` | `Color` | page default | Text color — use `rgb()`, `cmyk()`, or `grayscale()` |
| `opacity` | `number` | 1.0 | Transparency: 0.0 (invisible) to 1.0 (fully opaque) |
| `blendMode` | `BlendMode` | `Normal` | Color blending: Normal, Multiply, Screen, Overlay, etc. |

### Transformation Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `rotate` | `Rotation` | none | Rotation angle — use `degrees()` or `radians()` |
| `xSkew` | `Rotation` | none | Horizontal skew — use `degrees()` or `radians()` |
| `ySkew` | `Rotation` | none | Vertical skew — use `degrees()` or `radians()` |

### Multiline & Wrapping Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `lineHeight` | `number` | page default | Vertical spacing between lines in points |
| `maxWidth` | `number` | none | Maximum text width — triggers automatic word wrapping |
| `wordBreaks` | `string[]` | `[' ']` | Characters where wrapping can occur |

## Color Functions — Complete Reference

### rgb(r, g, b)

```typescript
import { rgb } from 'pdf-lib'

// ALL values MUST be 0.0 to 1.0
const red = rgb(1, 0, 0)
const green = rgb(0, 1, 0)
const blue = rgb(0, 0, 1)
const black = rgb(0, 0, 0)
const white = rgb(1, 1, 1)
const teal = rgb(0, 0.53, 0.71)
const darkGray = rgb(0.2, 0.2, 0.2)
```

### cmyk(c, m, y, k)

```typescript
import { cmyk } from 'pdf-lib'

// ALL values MUST be 0.0 to 1.0
const cyan = cmyk(1, 0, 0, 0)
const magenta = cmyk(0, 1, 0, 0)
const yellow = cmyk(0, 0, 1, 0)
const black = cmyk(0, 0, 0, 1)
const richBlack = cmyk(0.75, 0.68, 0.67, 0.90)
```

### grayscale(value)

```typescript
import { grayscale } from 'pdf-lib'

// 0.0 = black, 1.0 = white
const black = grayscale(0)
const white = grayscale(1)
const midGray = grayscale(0.5)
const lightGray = grayscale(0.8)
```

## Rotation Helpers

```typescript
import { degrees, radians } from 'pdf-lib'

// Degrees (human-readable)
const angle45 = degrees(45)
const angle90 = degrees(90)
const angleNeg45 = degrees(-45)

// Radians (math-native)
const quarterTurn = radians(Math.PI / 2)
const halfTurn = radians(Math.PI)
```

ALWAYS use `degrees()` or `radians()` — NEVER pass raw numbers to `rotate`, `xSkew`, or `ySkew`.

## BlendMode Enum Values

```typescript
import { BlendMode } from 'pdf-lib'

// Available blend modes
BlendMode.Normal      // Default
BlendMode.Multiply
BlendMode.Screen
BlendMode.Overlay
BlendMode.Darken
BlendMode.Lighten
BlendMode.ColorDodge
BlendMode.ColorBurn
BlendMode.HardLight
BlendMode.SoftLight
BlendMode.Difference
BlendMode.Exclusion
```

## PDFFont — Text Measurement Methods

### widthOfTextAtSize(text, size)

```typescript
widthOfTextAtSize(text: string, size: number): number
```

Returns the width of `text` when rendered at `size` in PDF points. ALWAYS use this for horizontal centering, right-alignment, or checking if text fits within a boundary.

### heightAtSize(size, options?)

```typescript
heightAtSize(size: number, options?: { descender?: boolean }): number
```

Returns the height of the font at `size` in PDF points.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `descender` | `true` | Include descender height (below baseline: g, p, y) |

### sizeAtHeight(height)

```typescript
sizeAtHeight(height: number): number
```

Returns the font size needed to achieve a target height in PDF points. Useful for fitting text into a fixed-height area.

### encodeText(text)

```typescript
encodeText(text: string): PDFHexString
```

Encodes text for the font. `drawText()` calls this automatically — you NEVER need to call it directly for normal text drawing.

### getCharacterSet()

```typescript
getCharacterSet(): number[]
```

Returns an array of Unicode code points this font supports. Useful for checking character support before drawing.

```typescript
const charSet = font.getCharacterSet()
const supportsEuro = charSet.includes('€'.charCodeAt(0))
```

## Page-Level Default Methods

```typescript
page.setFont(font: PDFFont): void
page.setFontSize(fontSize: number): void
page.setFontColor(fontColor: Color): void
page.setLineHeight(lineHeight: number): void
```

These set defaults for ALL subsequent `drawText()` calls on the page. Per-call options ALWAYS override these defaults.

**Usage pattern:**
```typescript
// Set once at the top
page.setFont(helveticaFont)
page.setFontSize(12)
page.setFontColor(rgb(0, 0, 0))
page.setLineHeight(16)

// All subsequent drawText calls use these defaults
page.drawText('First paragraph', { x: 50, y: 700 })
page.drawText('Second paragraph', { x: 50, y: 680 })
page.drawText('Bold override', { x: 50, y: 660, font: boldFont })
```
