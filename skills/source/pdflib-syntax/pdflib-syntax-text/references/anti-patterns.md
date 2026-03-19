# Text Drawing Anti-Patterns

## Anti-Pattern 1: Coordinate System Confusion (Top-Left vs Bottom-Left)

PDF coordinates start at the BOTTOM-LEFT. y=0 is the bottom of the page. This is the opposite of HTML/CSS/Canvas.

```typescript
// WRONG — text appears at the bottom, not the top
page.drawText('Title', { x: 50, y: 50, size: 24, font })

// CORRECT — subtract from height to position near the top
const { height } = page.getSize()
page.drawText('Title', { x: 50, y: height - 50, size: 24, font })
```

**Rule:** ALWAYS get page dimensions with `page.getSize()` and calculate y from the top using `height - offset`.

## Anti-Pattern 2: Color Values 0-255 Instead of 0-1

ALL color functions use the 0.0-1.0 range. Using 0-255 values produces incorrect colors or white.

```typescript
// WRONG — rgb() does NOT accept 0-255 values
page.drawText('Red text', {
  x: 50, y: 500, size: 14, font,
  color: rgb(255, 0, 0),  // NOT red — overflows
})

// CORRECT — use 0.0 to 1.0
page.drawText('Red text', {
  x: 50, y: 500, size: 14, font,
  color: rgb(1, 0, 0),  // red
})

// Converting from 0-255:
const r = 51, g = 102, b = 153
const color = rgb(r / 255, g / 255, b / 255)
```

This applies to ALL color functions: `rgb()`, `cmyk()`, `grayscale()`.

## Anti-Pattern 3: Missing Font Embedding

Drawing text without embedding a font first causes errors.

```typescript
// WRONG — no font embedded
const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()
page.drawText('Hello', { x: 50, y: 500, size: 14 })
// Error: no font set

// CORRECT — ALWAYS embed font first
const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
page.drawText('Hello', { x: 50, y: 500, size: 14, font })
```

**Alternative:** Set page-level default font before drawing:
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
page.setFont(font)
page.drawText('Hello', { x: 50, y: 500, size: 14 })
```

## Anti-Pattern 4: Forgetting to Await Font Embedding

`embedFont()` is async and returns a Promise. Using the Promise directly as a font causes errors.

```typescript
// WRONG — font is a Promise, not a PDFFont
const font = pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })  // TypeError or silent failure

// CORRECT — ALWAYS await embedFont()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })
```

**Exception:** `embedStandardFont()` is synchronous and does NOT need `await`:
```typescript
const font = pdfDoc.embedStandardFont(StandardFonts.Courier)  // no await
page.drawText('Hello', { font })
```

## Anti-Pattern 5: Multiline Text Without lineHeight

Using `\n` without setting `lineHeight` causes lines to overlap or have unpredictable spacing.

```typescript
// WRONG — lines may overlap
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50, y: 500, size: 14, font,
})

// CORRECT — ALWAYS set lineHeight with multiline text
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50, y: 500, size: 14, font,
  lineHeight: 20,  // fontSize * 1.2 to 1.5
})
```

## Anti-Pattern 6: maxWidth Without lineHeight

Using `maxWidth` for word wrapping without `lineHeight` produces unpredictable line spacing.

```typescript
// WRONG — wrapped lines have inconsistent spacing
page.drawText('Long text that will wrap...', {
  x: 50, y: 500, size: 12, font,
  maxWidth: 400,
})

// CORRECT — ALWAYS set lineHeight when using maxWidth
page.drawText('Long text that will wrap...', {
  x: 50, y: 500, size: 12, font,
  maxWidth: 400,
  lineHeight: 16,
})
```

## Anti-Pattern 7: Raw Number for Rotation

The `rotate` option requires a `Rotation` type created by `degrees()` or `radians()`. Raw numbers do NOT work.

```typescript
// WRONG — raw number is not a Rotation type
page.drawText('Rotated', {
  x: 50, y: 500, size: 14, font,
  rotate: 45,  // TypeError or ignored
})

// CORRECT — use degrees() or radians()
import { degrees } from 'pdf-lib'
page.drawText('Rotated', {
  x: 50, y: 500, size: 14, font,
  rotate: degrees(45),
})
```

## Anti-Pattern 8: Standard Fonts with Unicode Characters

Standard fonts (Helvetica, TimesRoman, Courier) ONLY support WinAnsi encoding (Latin-1 subset). Any non-WinAnsi character throws an error.

```typescript
// WRONG — standard fonts cannot encode Unicode beyond Latin-1
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Привет мир', { font })   // CRASHES: WinAnsi cannot encode
page.drawText('你好世界', { font })       // CRASHES: WinAnsi cannot encode
page.drawText('مرحبا', { font })         // CRASHES: WinAnsi cannot encode

// CORRECT — use custom font with fontkit for non-Latin text
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)  // TTF/OTF
page.drawText('Привет мир', { font: customFont })     // Works
```

**Error message:** `"WinAnsi cannot encode "X" (0xNNNN)"`

This is the #1 most common pdf-lib error reported on GitHub.

## Anti-Pattern 9: Centering Without Measuring

Guessing text position instead of calculating it from measured width.

```typescript
// WRONG — hardcoded x does NOT center text
page.drawText('Centered Title', {
  x: 200,  // guessed value — NOT centered
  y: 500, size: 24, font,
})

// CORRECT — measure text width and calculate center
const text = 'Centered Title'
const fontSize = 24
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()

page.drawText(text, {
  x: (pageWidth - textWidth) / 2,  // mathematically centered
  y: 500, size: fontSize, font,
})
```

## Anti-Pattern 10: Opacity Outside 0-1 Range

Opacity MUST be between 0.0 and 1.0. Values outside this range cause unpredictable rendering.

```typescript
// WRONG — opacity 100 is out of range
page.drawText('Faded', {
  x: 50, y: 500, size: 14, font,
  opacity: 100,  // should be 0.0-1.0
})

// CORRECT
page.drawText('Faded', {
  x: 50, y: 500, size: 14, font,
  opacity: 0.5,  // 50% transparent
})
```

## Summary Table

| Anti-Pattern | Root Cause | Fix |
|---|---|---|
| Text at wrong position | y=0 is bottom, not top | Use `height - offset` for top positioning |
| White/wrong colors | 0-255 values instead of 0-1 | Divide by 255 or use 0.0-1.0 directly |
| No font error | Missing embedFont call | ALWAYS embed font before drawText |
| Font is Promise | Missing await on embedFont | ALWAYS await async font embedding |
| Lines overlap | Missing lineHeight with \n | Set lineHeight to fontSize * 1.2-1.5 |
| Wrapping spacing off | maxWidth without lineHeight | ALWAYS set lineHeight with maxWidth |
| Rotation ignored | Raw number instead of degrees() | Use degrees() or radians() |
| WinAnsi encode error | Standard font with Unicode | Use custom font + fontkit |
| Text not centered | Hardcoded x position | Measure with widthOfTextAtSize() |
| Rendering glitches | Opacity > 1.0 | Keep opacity in 0.0-1.0 range |
