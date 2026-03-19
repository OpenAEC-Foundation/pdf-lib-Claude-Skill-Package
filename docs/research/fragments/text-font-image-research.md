# pdf-lib Text, Fonts & Images — Deep Research

> **Sources**: pdf-lib.js.org API docs, GitHub Hopding/pdf-lib README, npm @pdf-lib/fontkit, GitHub issues (font+unicode)
> **Date**: 2026-03-19
> **Researcher**: Claude Opus 4.6 (automated deep research, Part 2 of 3)

---

## 1. Text Operations

### 1.1 drawText() — Complete API

```typescript
page.drawText(text: string, options?: PDFPageDrawTextOptions): void
```

**PDFPageDrawTextOptions** (ALL properties optional):

| Property | Type | Purpose |
|----------|------|---------|
| `x` | `number` | Horizontal position (default: 0) |
| `y` | `number` | Vertical position (default: 0) |
| `font` | `PDFFont` | Font to use (set via setFont() or here) |
| `size` | `number` | Font size in points |
| `color` | `Color` | Text color (rgb, cmyk, or grayscale) |
| `opacity` | `number` | Transparency 0.0 (invisible) to 1.0 (opaque) |
| `rotate` | `Rotation` | Rotation via degrees() or radians() |
| `xSkew` | `Rotation` | Horizontal skew transformation |
| `ySkew` | `Rotation` | Vertical skew transformation |
| `lineHeight` | `number` | Vertical spacing between lines |
| `maxWidth` | `number` | Text wrapping constraint (wraps at word breaks) |
| `wordBreaks` | `string[]` | Custom word break characters for wrapping |
| `blendMode` | `BlendMode` | Color blending method |

**Key behaviors:**
- Text is drawn from the BOTTOM-LEFT origin (PDF coordinate system — y=0 is bottom of page)
- Multiline text: embed `\n` in the string; lineHeight controls spacing
- maxWidth triggers automatic word wrapping at wordBreaks characters
- Default wordBreaks: `[' ']` (space character)

### 1.2 Complete drawText() Examples

**Basic text:**
```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const timesRomanFont = await pdfDoc.embedFont(StandardFonts.TimesRoman)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

page.drawText('Creating PDFs in JavaScript is awesome!', {
  x: 50,
  y: height - 120,
  size: 30,
  font: timesRomanFont,
  color: rgb(0, 0.53, 0.71),
})

const pdfBytes = await pdfDoc.save()
```

**Rotated text on existing PDF:**
```typescript
import { degrees, PDFDocument, rgb, StandardFonts } from 'pdf-lib'

const existingPdfBytes = ... // Uint8Array or ArrayBuffer
const pdfDoc = await PDFDocument.load(existingPdfBytes)
const helveticaFont = await pdfDoc.embedFont(StandardFonts.Helvetica)

const pages = pdfDoc.getPages()
const firstPage = pages[0]
const { width, height } = firstPage.getSize()

firstPage.drawText('This text was added with JavaScript!', {
  x: 5,
  y: height / 2 + 300,
  size: 50,
  font: helveticaFont,
  color: rgb(0.95, 0.1, 0.1),
  rotate: degrees(-45),
})

const pdfBytes = await pdfDoc.save()
```

**Multiline text with lineHeight:**
```typescript
page.drawText('Line 1\nLine 2\nLine 3', {
  x: 50,
  y: 500,
  size: 14,
  font: helveticaFont,
  lineHeight: 20,
  color: rgb(0, 0, 0),
})
```

### 1.3 Page-level Font Defaults

```typescript
page.setFont(helveticaFont)     // Default font for subsequent drawText
page.setFontSize(14)            // Default size
page.setFontColor(rgb(0, 0, 0)) // Default color
```

When these are set, drawText() uses them as defaults — individual calls can still override.

### 1.4 Color Functions

```typescript
rgb(red: number, green: number, blue: number): RGB         // 0.0 to 1.0
cmyk(cyan: number, magenta: number, yellow: number, key: number): CMYK  // 0.0 to 1.0
grayscale(gray: number): Grayscale                          // 0.0 to 1.0
```

### 1.5 Rotation Helpers

```typescript
degrees(degreeAngle: number): Degrees    // e.g., degrees(45), degrees(-90)
radians(radianAngle: number): Radians    // e.g., radians(Math.PI / 4)
degreesToRadians(degree: number): number
radiansToDegrees(radian: number): number
```

### 1.6 BlendMode Enum

12 blend modes available: `Normal`, `Multiply`, `Screen`, `Overlay`, `Darken`, `Lighten`, `ColorDodge`, `ColorBurn`, `HardLight`, `SoftLight`, `Difference`, `Exclusion`.

---

## 2. Standard Fonts

### 2.1 StandardFonts Enum — Complete List (14 fonts)

| Enum Key | PDF Name | Family |
|----------|----------|--------|
| `StandardFonts.Courier` | `"Courier"` | Monospace |
| `StandardFonts.CourierBold` | `"Courier-Bold"` | Monospace |
| `StandardFonts.CourierOblique` | `"Courier-Oblique"` | Monospace |
| `StandardFonts.CourierBoldOblique` | `"Courier-BoldOblique"` | Monospace |
| `StandardFonts.Helvetica` | `"Helvetica"` | Sans-serif |
| `StandardFonts.HelveticaBold` | `"Helvetica-Bold"` | Sans-serif |
| `StandardFonts.HelveticaOblique` | `"Helvetica-Oblique"` | Sans-serif |
| `StandardFonts.HelveticaBoldOblique` | `"Helvetica-BoldOblique"` | Sans-serif |
| `StandardFonts.TimesRoman` | `"Times-Roman"` | Serif |
| `StandardFonts.TimesRomanBold` | `"Times-Bold"` | Serif |
| `StandardFonts.TimesRomanItalic` | `"Times-Italic"` | Serif |
| `StandardFonts.TimesRomanBoldItalic` | `"Times-BoldItalic"` | Serif |
| `StandardFonts.Symbol` | `"Symbol"` | Symbol |
| `StandardFonts.ZapfDingbats` | `"ZapfDingbats"` | Dingbats |

### 2.2 Standard Font Embedding

```typescript
// Async method (returns Promise<PDFFont>)
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica)

// Sync alternative
const courier = pdfDoc.embedStandardFont(StandardFonts.Courier)
```

Note: `embedStandardFont()` is synchronous — no await needed. But `embedFont()` with a StandardFonts value also works (async).

### 2.3 Standard Font Limitations — CRITICAL

**Standard fonts use WinAnsi encoding ONLY.** This means:

- ONLY Latin-1 characters are supported (Western European subset)
- NO Unicode support: Cyrillic, Chinese, Arabic, Hebrew, Thai, Korean, Japanese — ALL FAIL
- NO emoji support
- Error thrown: `"WinAnsi cannot encode "X" (0xNNNN)"`

**Characters that FAIL with standard fonts:**
- Cyrillic: Т, е, с, т (U+0422, etc.)
- Chinese: any CJK character
- Arabic: any Arabic script
- Hebrew: any Hebrew character
- Thai: any Thai character
- Emoji: any emoji character
- Special Unicode: fraction slash, various mathematical symbols

**This is the #1 source of issues in pdf-lib** (dozens of open GitHub issues).

### 2.4 Font Compatibility Matrix

| Font Family | Latin (A-Z, 0-9) | Accented (é, ü, ñ) | Cyrillic | CJK | Arabic | Emoji |
|-------------|:-:|:-:|:-:|:-:|:-:|:-:|
| Standard (all 14) | YES | PARTIAL (WinAnsi subset) | NO | NO | NO | NO |
| Custom TTF/OTF (with fontkit) | YES | YES | YES* | YES* | YES* | DEPENDS* |

*Depends on the specific font file containing the required glyphs.

---

## 3. Custom Fonts

### 3.1 fontkit Registration — REQUIRED for Custom Fonts

```typescript
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)  // MUST call before embedFont() with custom font bytes
```

**@pdf-lib/fontkit package:**
- npm: `@pdf-lib/fontkit`
- Installation: `npm install @pdf-lib/fontkit`
- Purpose: Enables custom font (TTF/OTF) embedding and unicode support
- MUST be registered before calling `embedFont()` with font bytes
- NOT needed for StandardFonts

### 3.2 registerFontkit() Signature

```typescript
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

### 3.3 embedFont() — Complete Signature

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

**Input types for `font` parameter:**
- `StandardFonts` enum value — standard font (no fontkit needed)
- `string` — base64-encoded font data or data URI
- `Uint8Array` — raw font file bytes
- `ArrayBuffer` — raw font file bytes

**EmbedFontOptions:**

| Property | Type | Purpose |
|----------|------|---------|
| `subset` | `boolean \| undefined` | Enable font subsetting (reduces file size by including only used glyphs) |
| `customName` | `string \| undefined` | Custom name for the embedded font |
| `features` | `TypeFeatures \| undefined` | OpenType font feature configuration |

### 3.4 Complete Custom Font Example with Measurement

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

// Load font bytes (from file, fetch, etc.)
const fontUrl = 'https://pdf-lib.js.org/assets/ubuntu/Ubuntu-R.ttf'
const fontBytes = await fetch(fontUrl).then((res) => res.arrayBuffer())

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
const text = 'This is text in an embedded font!'
const textSize = 35
const textWidth = customFont.widthOfTextAtSize(text, textSize)
const textHeight = customFont.heightAtSize(textSize)

// Draw text
page.drawText(text, {
  x: 40,
  y: 450,
  size: textSize,
  font: customFont,
  color: rgb(0, 0.53, 0.71),
})

// Draw bounding box around text (for visualization)
page.drawRectangle({
  x: 40,
  y: 450,
  width: textWidth,
  height: textHeight,
  borderColor: rgb(1, 0, 0),
  borderWidth: 1.5,
})
```

### 3.5 Unicode Text with Custom Font

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

const fontUrl = 'https://pdf-lib.js.org/assets/ubuntu/Ubuntu-R.ttf'
const fontBytes = await fetch(fontUrl).then((res) => res.arrayBuffer())

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)
const ubuntuFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
page.drawText('Some fancy Unicode text in the ŪЬȕǹƚü font', {
  font: ubuntuFont,
})
```

### 3.6 Font Subsetting

```typescript
// With subsetting — smaller file, only includes used glyphs
const font = await pdfDoc.embedFont(fontBytes, { subset: true })

// Without subsetting — full font embedded, larger file
const font = await pdfDoc.embedFont(fontBytes, { subset: false })
```

**Subsetting trade-offs:**
- `subset: true` — Smaller PDF file size. Only glyphs actually used in the document are embedded. Recommended for production.
- `subset: false` — Larger PDF file size. Full font embedded. Useful when the PDF will be further modified and may need additional glyphs later.

---

## 4. Text Measuring — PDFFont Methods

### 4.1 PDFFont Class — Complete API

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this font belongs to |
| `name` | `string` | The name of this font |
| `ref` | `PDFRef` | Unique reference within the document |

**Methods:**

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `widthOfTextAtSize` | `(text: string, size: number)` | `number` | Width of text string at given font size (in PDF points) |
| `heightAtSize` | `(size: number, options?: { descender?: boolean })` | `number` | Height of font at given size. `descender: true` includes descender (default behavior unclear) |
| `sizeAtHeight` | `(height: number)` | `number` | Compute font size needed to achieve target height |
| `encodeText` | `(text: string)` | `PDFHexString` | Encode text for this font. NOTE: drawText() handles this automatically |
| `getCharacterSet` | `()` | `number[]` | Returns array of unicode code points this font supports |
| `embed` | `()` | `Promise<void>` | Embed font in document. Called automatically by save() |

### 4.2 Text Layout Calculations

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)

const text = 'Hello World'
const fontSize = 24

// Measure text dimensions
const textWidth = font.widthOfTextAtSize(text, fontSize)   // width in points
const textHeight = font.heightAtSize(fontSize)              // height in points

// Calculate font size from desired height
const targetHeight = 30 // points
const neededSize = font.sizeAtHeight(targetHeight)

// Check if font supports specific characters
const charSet = font.getCharacterSet() // number[] of unicode code points
const supportsChar = charSet.includes('é'.charCodeAt(0))
```

### 4.3 Centering Text Example

```typescript
const text = 'Centered Title'
const fontSize = 24
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()

page.drawText(text, {
  x: (pageWidth - textWidth) / 2,  // horizontally centered
  y: 500,
  size: fontSize,
  font: font,
})
```

---

## 5. Image Embedding

### 5.1 Supported Formats

**ONLY two formats supported:**
- **PNG** — via `embedPng()`
- **JPEG/JPG** — via `embedJpg()`

**NOT supported:** GIF, BMP, TIFF, WebP, SVG, AVIF, HEIC — NONE of these work.

### 5.2 embedPng() Signature

```typescript
pdfDoc.embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64 encoded PNG or data URI (`data:image/png;base64,...`)
- `Uint8Array` — raw PNG bytes
- `ArrayBuffer` — raw PNG bytes

### 5.3 embedJpg() Signature

```typescript
pdfDoc.embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
```

**Input types:**
- `string` — base64 encoded JPEG or data URI (`data:image/jpeg;base64,...`)
- `Uint8Array` — raw JPEG bytes
- `ArrayBuffer` — raw JPEG bytes

### 5.4 PDFImage Class — Complete API

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this image belongs to |
| `width` | `number` | Original width in pixels |
| `height` | `number` | Original height in pixels |
| `ref` | `PDFRef` | Unique reference within the document |

**Methods:**

| Method | Signature | Returns | Description |
|--------|-----------|---------|-------------|
| `scale` | `(factor: number)` | `{ width: number, height: number }` | Scale dimensions by factor. E.g., 500x250 at 0.5 = 250x125 |
| `scaleToFit` | `(width: number, height: number)` | `{ width: number, height: number }` | Scale to fit within bounds, preserving aspect ratio. E.g., 500x250 into 750x1000 = 750x375 |
| `size` | `()` | `{ width: number, height: number }` | Get original dimensions as object |
| `embed` | `()` | `Promise<void>` | Embed image in document. Called automatically by save() |

### 5.5 Complete Image Embedding Example

```typescript
import { PDFDocument } from 'pdf-lib'

const jpgImageBytes = ... // Uint8Array, ArrayBuffer, or base64 string
const pngImageBytes = ...

const pdfDoc = await PDFDocument.create()
const jpgImage = await pdfDoc.embedJpg(jpgImageBytes)
const pngImage = await pdfDoc.embedPng(pngImageBytes)

const jpgDims = jpgImage.scale(0.25)
const pngDims = pngImage.scale(0.5)

const page = pdfDoc.addPage()

page.drawImage(jpgImage, {
  x: page.getWidth() / 2 - jpgDims.width / 2,
  y: page.getHeight() / 2 - jpgDims.height / 2,
  width: jpgDims.width,
  height: jpgDims.height,
})

page.drawImage(pngImage, {
  x: page.getWidth() / 2 - pngDims.width / 2 + 75,
  y: page.getHeight() / 2 - pngDims.height,
  width: pngDims.width,
  height: pngDims.height,
})

const pdfBytes = await pdfDoc.save()
```

---

## 6. Image Drawing

### 6.1 drawImage() — Complete API

```typescript
page.drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
```

**PDFPageDrawImageOptions** (ALL properties optional):

| Property | Type | Purpose |
|----------|------|---------|
| `x` | `number` | Horizontal position (default: 0) |
| `y` | `number` | Vertical position (default: 0) |
| `width` | `number` | Display width (default: original image width) |
| `height` | `number` | Display height (default: original image height) |
| `rotate` | `Rotation` | Rotation via degrees() or radians() |
| `xSkew` | `Rotation` | Horizontal skew |
| `ySkew` | `Rotation` | Vertical skew |
| `opacity` | `number` | Transparency 0.0-1.0 |
| `blendMode` | `BlendMode` | Color blending method |

### 6.2 Aspect Ratio Preservation Patterns

**Pattern 1: Uniform scale factor**
```typescript
const image = await pdfDoc.embedPng(pngBytes)
const dims = image.scale(0.5) // 50% of original
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
})
```

**Pattern 2: Fit within bounds (preserves aspect ratio)**
```typescript
const image = await pdfDoc.embedJpg(jpgBytes)
const dims = image.scaleToFit(300, 200) // max 300x200, aspect ratio preserved
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
})
```

**Pattern 3: Manual aspect ratio calculation**
```typescript
const image = await pdfDoc.embedPng(pngBytes)
const targetWidth = 300
const scale = targetWidth / image.width
page.drawImage(image, {
  x: 50, y: 50,
  width: targetWidth,
  height: image.height * scale,
})
```

**ANTI-PATTERN: Stretching without aspect ratio**
```typescript
// BAD — image will be distorted
page.drawImage(image, {
  x: 50, y: 50,
  width: 300,  // arbitrary width
  height: 300, // arbitrary height — aspect ratio broken!
})
```

### 6.3 Image Drawing with Rotation and Opacity

```typescript
page.drawImage(jpgImage, {
  x: 25,
  y: 25,
  width: 200,
  height: 150,
  rotate: degrees(30),
  opacity: 0.75,
})
```

---

## 7. Common Errors & Anti-Patterns

### 7.1 Standard Fonts with Unicode — THE #1 ERROR

**Error:** `"WinAnsi cannot encode "Т" (0x0422)"`

**Cause:** Using StandardFonts (Helvetica, TimesRoman, Courier) with non-WinAnsi characters.

**Broken code:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Привет мир', { font }) // CRASHES — Cyrillic not in WinAnsi
```

**Fix:** Use a custom font with fontkit:
```typescript
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes) // TTF/OTF with Cyrillic support
page.drawText('Привет мир', { font }) // Works!
```

### 7.2 Missing fontkit Registration

**Error:** Attempting to embed custom font bytes without registering fontkit first.

**Broken code:**
```typescript
const pdfDoc = await PDFDocument.create()
// MISSING: pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes) // CRASHES
```

**Fix:** ALWAYS register fontkit before embedding custom fonts:
```typescript
import fontkit from '@pdf-lib/fontkit'
const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)              // MUST come first
const font = await pdfDoc.embedFont(fontBytes) // Now works
```

### 7.3 Unsupported Image Formats

**Error:** Attempting to embed GIF, BMP, TIFF, WebP, SVG, or any format other than PNG/JPG.

pdf-lib ONLY supports `embedPng()` and `embedJpg()`. There is NO `embedGif()`, `embedBmp()`, `embedSvg()`, or generic `embedImage()`.

**Workaround:** Convert images to PNG or JPG before embedding (using a canvas or image processing library).

### 7.4 Form Fields with Unicode — WinAnsi Error

**Broken code:**
```typescript
const form = pdfDoc.getForm()
const field = form.getTextField('name')
field.setText('Тест') // CRASHES — default font is Helvetica (WinAnsi)
```

**Fix:**
```typescript
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)
const field = form.getTextField('name')
field.setText('Тест')
field.updateAppearances(customFont) // Use custom font for rendering
```

### 7.5 Coordinate System Confusion

PDF coordinates start at BOTTOM-LEFT (y=0 is the bottom of the page). This is the opposite of HTML/CSS/Canvas where y=0 is the top.

```typescript
const { width, height } = page.getSize()

// Top of page
page.drawText('Near top', { x: 50, y: height - 50 })

// Bottom of page
page.drawText('Near bottom', { x: 50, y: 50 })
```

### 7.6 Forgetting to Await Async Methods

```typescript
// WRONG — embedFont returns a Promise
const font = pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font }) // font is a Promise, not a PDFFont!

// CORRECT
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })
```

### 7.7 Known Open Issues (from GitHub)

| Issue | Problem | Status |
|-------|---------|--------|
| #1759 | Standard fonts cannot encode characters outside WinAnsi | Open |
| #1566 | Chinese character support | Open |
| #1450 | Arabic bidirectional text | Open |
| #561 | Hebrew/UTF-8 support | Open |
| #715 | Cyrillic/Russian symbols | Open |
| #1010 | Thai font rendering (unwanted spaces) | Open |
| #746 | Khmer font problems | Open |
| #1297 | Unicode fraction slash not rendering | Open |
| #50, #217 | Emoji embedding issues | Open |
| #1270 | ZapfDingbats font errors | Open |
| #1665 | Cannot merge multiple Standard Fonts for mixed Unicode | Open |
| #1152 | Non-Latin dropdown values trigger WinAnsi encoding failures | Open |
| #726 | Embedded font glyph width calculation errors | PR |

Most font/unicode issues remain **open and unresolved** — this is a fundamental limitation of the WinAnsi encoding used by standard fonts. The ONLY reliable workaround is using custom fonts via fontkit.

---

## 8. Summary of Key Facts

### Must-Know Rules
1. **Standard fonts = WinAnsi only = Latin-1 only** — NEVER use for international text
2. **Custom fonts require fontkit** — ALWAYS register before embedFont() with bytes
3. **Only PNG and JPG** — no other image formats supported
4. **PDF coordinates are bottom-left origin** — y=0 is the bottom
5. **embedFont() is async** — ALWAYS await the result
6. **Font subsetting reduces file size** — use `{ subset: true }` for production
7. **scaleToFit() preserves aspect ratio** — use it instead of manual width/height
8. **14 standard fonts** in 3 families + Symbol + ZapfDingbats

### Import Checklist
```typescript
// Core
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib'

// For custom fonts
import fontkit from '@pdf-lib/fontkit'

// Additional color functions
import { cmyk, grayscale } from 'pdf-lib'

// Additional types/enums
import { BlendMode } from 'pdf-lib'
```
