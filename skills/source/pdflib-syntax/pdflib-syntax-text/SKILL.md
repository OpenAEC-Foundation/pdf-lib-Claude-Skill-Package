---
name: pdflib-syntax-text
description: "Guides text drawing in pdf-lib including drawText options, text positioning, multiline text, text wrapping, text measuring, and centering. Activates when drawing text on PDF pages, positioning text, wrapping text, measuring text width, or centering text."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Text Drawing — Syntax Reference

## Quick Reference

```typescript
import { PDFDocument, StandardFonts, rgb, cmyk, grayscale, degrees, radians } from 'pdf-lib'

// Draw text on a page
page.drawText('Hello World', {
  x: 50,
  y: height - 50,  // bottom-left origin: y increases UPWARD
  size: 14,
  font: helveticaFont,
  color: rgb(0, 0, 0),
})

// Measure text width for centering
const textWidth = font.widthOfTextAtSize('Hello', 14)

// Set page-level defaults
page.setFont(font)
page.setFontSize(14)
page.setFontColor(rgb(0, 0, 0))
page.setLineHeight(20)
```

## Critical Warnings

> **COORDINATE SYSTEM**: PDF uses bottom-left origin. y=0 is the BOTTOM of the page, NOT the top. To place text near the top, use `y: height - margin`. This is the opposite of HTML/CSS/Canvas.

> **COLOR VALUES**: rgb(), cmyk(), and grayscale() ALL use 0.0-1.0 range. NEVER use 0-255. `rgb(255, 0, 0)` does NOT produce red — it overflows.

> **FONT REQUIRED**: ALWAYS embed a font before drawing text. Either use `await pdfDoc.embedFont(StandardFonts.Helvetica)` or embed a custom font. Drawing text without a font causes errors.

> **STANDARD FONT LIMITATION**: Standard fonts (Helvetica, TimesRoman, Courier) ONLY support WinAnsi (Latin-1) characters. For Unicode, CJK, Arabic, Cyrillic, or emoji — ALWAYS use a custom font with fontkit.

## Coordinate System

```
(0, height) ─────────────── (width, height)
     │                            │
     │   y increases UPWARD       │
     │                            │
     │   x increases RIGHTWARD    │
     │                            │
(0, 0) ──────────────────── (width, 0)
         BOTTOM-LEFT ORIGIN
```

Units: PDF points (1 point = 1/72 inch).
Typical sizes: Letter = 612 x 792 pt, A4 = 595.28 x 841.89 pt.

## Essential Patterns

### Pattern 1: Basic Text Drawing

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

page.drawText('Hello World', {
  x: 50,
  y: height - 50,
  size: 24,
  font: font,
  color: rgb(0, 0, 0),
})

const pdfBytes = await pdfDoc.save()
```

### Pattern 2: Horizontally Centered Text

```typescript
const text = 'Centered Title'
const fontSize = 24
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()

page.drawText(text, {
  x: (pageWidth - textWidth) / 2,
  y: 500,
  size: fontSize,
  font: font,
})
```

### Pattern 3: Multiline Text with \n

```typescript
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50,
  y: 500,
  size: 14,
  font: font,
  lineHeight: 20,
  color: rgb(0, 0, 0),
})
```

ALWAYS set `lineHeight` when using `\n`. Without it, lines may overlap or have inconsistent spacing. A good default is `fontSize * 1.2` to `fontSize * 1.5`.

### Pattern 4: Automatic Text Wrapping

```typescript
page.drawText('This is a long paragraph that will automatically wrap...', {
  x: 50,
  y: 700,
  size: 12,
  font: font,
  maxWidth: 500,        // wraps at 500 points width
  lineHeight: 16,       // ALWAYS set with maxWidth
  wordBreaks: [' '],    // default: break on spaces
  color: rgb(0, 0, 0),
})
```

`maxWidth` triggers word wrapping at characters in `wordBreaks`. Default `wordBreaks` is `[' ']` (space). ALWAYS set `lineHeight` together with `maxWidth` to control line spacing.

### Pattern 5: Rotated Text

```typescript
import { degrees } from 'pdf-lib'

page.drawText('Rotated 45 degrees', {
  x: 50,
  y: 400,
  size: 20,
  font: font,
  color: rgb(0.95, 0.1, 0.1),
  rotate: degrees(-45),
})
```

Use `degrees()` or `radians()` — NEVER pass raw numbers to `rotate`.

### Pattern 6: Page-Level Defaults

```typescript
page.setFont(font)
page.setFontSize(12)
page.setFontColor(rgb(0, 0, 0))
page.setLineHeight(18)

// Now drawText uses these defaults — individual calls can still override
page.drawText('Uses page defaults', { x: 50, y: 700 })
page.drawText('Override size', { x: 50, y: 680, size: 20 })
```

### Pattern 7: Text Measurement

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)

// Width of specific text at specific size (in PDF points)
const textWidth = font.widthOfTextAtSize('Hello World', 24)

// Height of font at specific size (in PDF points)
const textHeight = font.heightAtSize(24)

// With descender included (default: true)
const fullHeight = font.heightAtSize(24, { descender: true })
const ascenderOnly = font.heightAtSize(24, { descender: false })

// Calculate font size needed for a target height
const neededSize = font.sizeAtHeight(30)
```

### Pattern 8: Color Functions

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib'

// RGB — ALL values 0.0 to 1.0
const red = rgb(1, 0, 0)
const teal = rgb(0, 0.53, 0.71)
const black = rgb(0, 0, 0)

// CMYK — ALL values 0.0 to 1.0
const cyanCmyk = cmyk(1, 0, 0, 0)
const blackCmyk = cmyk(0, 0, 0, 1)

// Grayscale — 0.0 (black) to 1.0 (white)
const grayText = grayscale(0.4)
```

## Decision Tree: Text Positioning

```
Need to position text on the page?
├── Near the top? → y: height - margin (e.g., height - 50)
├── Near the bottom? → y: margin (e.g., 50)
├── Centered horizontally?
│   → x: (pageWidth - font.widthOfTextAtSize(text, size)) / 2
├── Multiple lines?
│   ├── Manual line breaks? → Use \n in string + set lineHeight
│   └── Auto-wrap at width? → Use maxWidth + set lineHeight
└── Rotated?
    → Use rotate: degrees(angle) or radians(angle)
```

## Decision Tree: Font & Color Selection

```
Choosing text color?
├── Standard RGB? → rgb(r, g, b) with 0.0-1.0 values
├── Print/CMYK workflow? → cmyk(c, m, y, k) with 0.0-1.0 values
└── Grayscale? → grayscale(value) with 0.0 (black) to 1.0 (white)

Choosing rotation unit?
├── Degrees (human-readable)? → degrees(45)
└── Radians (math-native)? → radians(Math.PI / 4)
```

## drawText() Options Summary

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `x` | `number` | 0 | Horizontal position |
| `y` | `number` | 0 | Vertical position (bottom-left origin) |
| `font` | `PDFFont` | page default | Embedded font |
| `size` | `number` | page default | Font size in points |
| `color` | `Color` | page default | rgb(), cmyk(), or grayscale() |
| `opacity` | `number` | 1.0 | 0.0 (invisible) to 1.0 (opaque) |
| `rotate` | `Rotation` | none | degrees() or radians() |
| `xSkew` | `Rotation` | none | Horizontal skew |
| `ySkew` | `Rotation` | none | Vertical skew |
| `lineHeight` | `number` | page default | Spacing between lines |
| `maxWidth` | `number` | none | Wrapping width constraint |
| `wordBreaks` | `string[]` | `[' ']` | Characters to break on |
| `blendMode` | `BlendMode` | Normal | Color blending method |

## Page-Level Text Defaults

| Method | Sets Default For |
|--------|-----------------|
| `page.setFont(font)` | `font` option |
| `page.setFontSize(size)` | `size` option |
| `page.setFontColor(color)` | `color` option |
| `page.setLineHeight(height)` | `lineHeight` option |

When set, `drawText()` uses these as defaults. Per-call options ALWAYS override page defaults.

## Reference Links

- [drawText Options — Full Method Reference](references/methods.md)
- [Text Positioning Examples — Multiline, Centering, Rotation](references/examples.md)
- [Anti-Patterns — Common Text Drawing Mistakes](references/anti-patterns.md)
- [pdf-lib API Docs — PDFPage](https://pdf-lib.js.org/docs/api/classes/pdfpage)
- [pdf-lib API Docs — PDFFont](https://pdf-lib.js.org/docs/api/classes/pdffont)
