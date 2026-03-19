---
name: pdflib-syntax-fonts
description: "Guides font embedding in pdf-lib including standard fonts, custom TTF/OTF fonts, fontkit registration, unicode support, font subsetting, and text measuring. Activates when embedding fonts, using custom fonts, handling unicode text, or measuring text dimensions in PDFs."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-syntax-fonts

## Quick Reference

### Font Embedding Methods

| Method | Async | Input | Requires fontkit |
|--------|-------|-------|-----------------|
| `pdfDoc.embedFont(StandardFonts.X)` | Yes | `StandardFonts` enum | No |
| `pdfDoc.embedStandardFont(StandardFonts.X)` | **No** | `StandardFonts` enum | No |
| `pdfDoc.embedFont(fontBytes)` | Yes | `Uint8Array \| ArrayBuffer \| string` | **Yes** |
| `pdfDoc.embedFont(fontBytes, { subset: true })` | Yes | `Uint8Array \| ArrayBuffer \| string` | **Yes** |

### StandardFonts Enum (14 Fonts)

| Family | Regular | Bold | Italic/Oblique | Bold Italic/Oblique |
|--------|---------|------|----------------|---------------------|
| Serif | `TimesRoman` | `TimesRomanBold` | `TimesRomanItalic` | `TimesRomanBoldItalic` |
| Sans-serif | `Helvetica` | `HelveticaBold` | `HelveticaOblique` | `HelveticaBoldOblique` |
| Monospace | `Courier` | `CourierBold` | `CourierOblique` | `CourierBoldOblique` |
| Special | `Symbol` | -- | -- | -- |
| Special | `ZapfDingbats` | -- | -- | -- |

### PDFFont Measurement Methods

| Method | Signature | Returns |
|--------|-----------|---------|
| `widthOfTextAtSize` | `(text: string, size: number)` | `number` (points) |
| `heightAtSize` | `(size: number, options?: { descender?: boolean })` | `number` (points) |
| `sizeAtHeight` | `(height: number)` | `number` (font size) |
| `getCharacterSet` | `()` | `number[]` (unicode code points) |
| `encodeText` | `(text: string)` | `PDFHexString` |

### Essential Imports

```typescript
// Standard fonts only
import { PDFDocument, StandardFonts } from 'pdf-lib'

// Custom fonts (unicode, TTF/OTF)
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
```

---

## Critical Warnings

### WinAnsi Encoding Limitation -- THE #1 SOURCE OF ISSUES

**ALL 14 standard fonts use WinAnsi encoding ONLY.** This encoding supports approximately 218 characters: basic Latin, some Western European accented characters, and a handful of symbols. Everything else FAILS with a runtime error.

**Error message:** `"WinAnsi cannot encode "X" (0xNNNN)"`

**Characters that ALWAYS FAIL with standard fonts:**
- Cyrillic (Russian, Ukrainian, Bulgarian, Serbian)
- CJK (Chinese, Japanese, Korean)
- Arabic, Hebrew, Thai, Hindi, Korean
- Emoji of any kind
- Many mathematical symbols
- Some typographic characters (e.g., fraction slash U+2044)

**Characters that WORK with standard fonts:**
- ASCII A-Z, a-z, 0-9
- Common punctuation: `.` `,` `;` `:` `!` `?` `'` `"` `(` `)` `-` `/`
- A PARTIAL set of accented Latin: e.g., `a`, `e`, `u`, `n`, `o`, `c`
- Currency symbols: `$`, `EUR`, `GBP`, `YEN`
- Basic math: `+`, `-`, `x`, `=`, `<`, `>`, `%`

> This limitation has dozens of open GitHub issues (#1759, #1566, #1450, #561, #715, #1010, #746, #1297, #50, #217, #1270, #1665). There is NO workaround using standard fonts -- you MUST use a custom font with fontkit for any non-WinAnsi character.

### Missing fontkit Registration

**NEVER** call `embedFont()` with font bytes (Uint8Array/ArrayBuffer/string) without first calling `registerFontkit()`. The call will throw a runtime error.

```typescript
// WRONG -- crashes
const font = await pdfDoc.embedFont(fontBytes)

// CORRECT
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes)
```

### Forgetting to Await embedFont()

`embedFont()` returns `Promise<PDFFont>`. Passing a Promise to `drawText()` instead of a resolved `PDFFont` causes silent failures or type errors.

```typescript
// WRONG
const font = pdfDoc.embedFont(StandardFonts.Helvetica) // Missing await!
page.drawText('Hello', { font }) // font is a Promise, not PDFFont

// CORRECT
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })
```

---

## Font Compatibility Matrix

| Character Set | Standard Fonts (all 14) | Custom TTF/OTF (with fontkit) |
|---------------|:-----------------------:|:-----------------------------:|
| Latin (A-Z, 0-9) | YES | YES |
| Accented Latin (e, u, n) | PARTIAL (WinAnsi subset) | YES |
| Cyrillic | NO | YES* |
| CJK (Chinese/Japanese/Korean) | NO | YES* |
| Arabic | NO | YES* |
| Hebrew | NO | YES* |
| Thai | NO | YES* |
| Emoji | NO | DEPENDS* |

*Requires a font file that contains the needed glyphs. The font itself must support the script -- fontkit only enables embedding, it does not add missing glyphs.

---

## Decision Trees

### Standard vs Custom Font

```
Need to embed a font?
+-- Text contains ONLY ASCII + basic Western European?
|   +-- YES --> Use StandardFonts (no extra dependencies)
|   |           embedFont(StandardFonts.Helvetica) or embedStandardFont()
|   +-- NO  --> MUST use custom font with fontkit
|               1. npm install @pdf-lib/fontkit
|               2. pdfDoc.registerFontkit(fontkit)
|               3. pdfDoc.embedFont(fontBytes)
```

### Which Standard Font Family

```
Choosing a standard font?
+-- Body text, reports, documents --> TimesRoman family (serif)
+-- UI labels, headings, modern docs --> Helvetica family (sans-serif)
+-- Code, fixed-width tables, logs --> Courier family (monospace)
+-- Special symbols/bullets --> Symbol or ZapfDingbats
```

### Font Subsetting Decision

```
Embedding a custom font?
+-- Final output PDF (no further editing)? --> subset: true (smaller file)
+-- PDF will be modified later with new text? --> subset: false (full font)
+-- File size is critical? --> subset: true
+-- Default (no option specified) --> subset is undefined (library default)
```

---

## Essential Patterns

### Pattern 1: Standard Font (Simplest)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()

page.drawText('Hello World', {
  x: 50, y: 500,
  size: 24,
  font,
  color: rgb(0, 0, 0),
})
```

### Pattern 2: Synchronous Standard Font

```typescript
// embedStandardFont() is the ONLY synchronous font method
const font = pdfDoc.embedStandardFont(StandardFonts.Courier)
// No await needed -- returns PDFFont directly
```

### Pattern 3: Custom Font with Unicode

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)  // MUST call before embedFont with bytes

const fontBytes = fs.readFileSync('path/to/font.ttf')
const font = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
page.drawText('Unicode text here', {
  x: 50, y: 500,
  size: 18,
  font,
})
```

### Pattern 4: Font Subsetting

```typescript
// Subset: only include glyphs used in the document (smaller file)
const font = await pdfDoc.embedFont(fontBytes, { subset: true })

// Full font: all glyphs embedded (larger file, editable later)
const fontFull = await pdfDoc.embedFont(fontBytes, { subset: false })
```

### Pattern 5: Text Measurement and Centering

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const text = 'Centered Title'
const fontSize = 24

const textWidth = font.widthOfTextAtSize(text, fontSize)
const textHeight = font.heightAtSize(fontSize)
const pageWidth = page.getWidth()

// Center horizontally
page.drawText(text, {
  x: (pageWidth - textWidth) / 2,
  y: 500,
  size: fontSize,
  font,
})
```

### Pattern 6: Character Support Check

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const charSet = font.getCharacterSet() // number[] of supported code points

function canRender(text: string, font: PDFFont): boolean {
  const charSet = new Set(font.getCharacterSet())
  for (let i = 0; i < text.length; i++) {
    if (!charSet.has(text.charCodeAt(i))) return false
  }
  return true
}
```

### Pattern 7: Calculate Font Size from Target Height

```typescript
const font = await pdfDoc.embedFont(StandardFonts.TimesRoman)

// Need text to be exactly 30 points tall
const targetHeight = 30
const fontSize = font.sizeAtHeight(targetHeight)

page.drawText('Sized to height', {
  x: 50, y: 500,
  size: fontSize,
  font,
})
```

---

## embedFont() Complete Signature

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

**EmbedFontOptions:**

| Property | Type | Purpose |
|----------|------|---------|
| `subset` | `boolean \| undefined` | Include only used glyphs (reduces file size) |
| `customName` | `string \| undefined` | Custom name for the embedded font |
| `features` | `TypeFeatures \| undefined` | OpenType font feature configuration |

**Input types for `font` parameter:**
- `StandardFonts` enum value -- no fontkit needed
- `string` -- base64-encoded font data or data URI
- `Uint8Array` -- raw font file bytes (TTF or OTF)
- `ArrayBuffer` -- raw font file bytes (TTF or OTF)

---

## PDFFont Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this font belongs to |
| `name` | `string` | The name of this font |
| `ref` | `PDFRef` | Unique reference within the document |

---

## Reference Links

- [references/methods.md](references/methods.md) -- Complete PDFFont API, embedFont, embedStandardFont, StandardFonts enum
- [references/examples.md](references/examples.md) -- Standard fonts, custom fonts, unicode, measuring
- [references/anti-patterns.md](references/anti-patterns.md) -- WinAnsi errors, missing fontkit, unicode failures

### Official Sources

- https://pdf-lib.js.org/docs/api/classes/pdfdocument (embedFont, embedStandardFont, registerFontkit)
- https://pdf-lib.js.org/docs/api/classes/pdffont (PDFFont methods)
- https://pdf-lib.js.org/docs/api/enums/standardfonts (StandardFonts enum)
- https://github.com/Hopding/pdf-lib (README examples)
- https://www.npmjs.com/package/@pdf-lib/fontkit (fontkit package)
