# pdflib-syntax-fonts -- Methods Reference

## PDFDocument Font Methods

### embedFont()

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

Embeds a font in the document. ALWAYS returns a Promise -- use `await`.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `font` | `StandardFonts \| string \| Uint8Array \| ArrayBuffer` | Yes | Font source |
| `options` | `EmbedFontOptions` | No | Embedding options |

**Input type details:**
- `StandardFonts` -- One of the 14 standard PDF fonts. No fontkit required.
- `string` -- Base64-encoded font data or a data URI (`data:font/ttf;base64,...`). Requires fontkit.
- `Uint8Array` -- Raw font file bytes (TTF or OTF format). Requires fontkit.
- `ArrayBuffer` -- Raw font file bytes (TTF or OTF format). Requires fontkit.

**Returns:** `Promise<PDFFont>`

---

### EmbedFontOptions

```typescript
interface EmbedFontOptions {
  subset?: boolean
  customName?: string
  features?: TypeFeatures
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `subset` | `boolean \| undefined` | `undefined` | When `true`, only glyphs used in the document are embedded (smaller file). When `false`, the full font is embedded. |
| `customName` | `string \| undefined` | `undefined` | Custom name for the font within the PDF. |
| `features` | `TypeFeatures \| undefined` | `undefined` | OpenType font feature configuration object. |

---

### embedStandardFont()

```typescript
pdfDoc.embedStandardFont(font: StandardFonts): PDFFont
```

Synchronous alternative to `embedFont()` for standard fonts ONLY. This is the ONLY synchronous font embedding method in pdf-lib.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `font` | `StandardFonts` | Yes | One of the 14 standard fonts |

**Returns:** `PDFFont` (not a Promise)

**NEVER** pass custom font bytes to this method -- it ONLY accepts `StandardFonts` enum values.

---

### registerFontkit()

```typescript
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

Registers the fontkit library for custom font parsing and embedding. MUST be called before `embedFont()` when using custom font bytes (Uint8Array, ArrayBuffer, or base64 string).

**NOT needed** when using `StandardFonts` or `embedStandardFont()`.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fontkit` | `Fontkit` | Yes | The fontkit instance from `@pdf-lib/fontkit` |

**Installation:**

```bash
npm install @pdf-lib/fontkit
```

**Import:**

```typescript
import fontkit from '@pdf-lib/fontkit'
```

---

## StandardFonts Enum -- Complete List

```typescript
import { StandardFonts } from 'pdf-lib'
```

### Courier Family (Monospace)

| Enum Key | PDF Name | Style |
|----------|----------|-------|
| `StandardFonts.Courier` | `"Courier"` | Regular |
| `StandardFonts.CourierBold` | `"Courier-Bold"` | Bold |
| `StandardFonts.CourierOblique` | `"Courier-Oblique"` | Oblique (italic) |
| `StandardFonts.CourierBoldOblique` | `"Courier-BoldOblique"` | Bold oblique |

### Helvetica Family (Sans-serif)

| Enum Key | PDF Name | Style |
|----------|----------|-------|
| `StandardFonts.Helvetica` | `"Helvetica"` | Regular |
| `StandardFonts.HelveticaBold` | `"Helvetica-Bold"` | Bold |
| `StandardFonts.HelveticaOblique` | `"Helvetica-Oblique"` | Oblique (italic) |
| `StandardFonts.HelveticaBoldOblique` | `"Helvetica-BoldOblique"` | Bold oblique |

### Times Roman Family (Serif)

| Enum Key | PDF Name | Style |
|----------|----------|-------|
| `StandardFonts.TimesRoman` | `"Times-Roman"` | Regular |
| `StandardFonts.TimesRomanBold` | `"Times-Bold"` | Bold |
| `StandardFonts.TimesRomanItalic` | `"Times-Italic"` | Italic |
| `StandardFonts.TimesRomanBoldItalic` | `"Times-BoldItalic"` | Bold italic |

### Special Fonts

| Enum Key | PDF Name | Purpose |
|----------|----------|---------|
| `StandardFonts.Symbol` | `"Symbol"` | Mathematical and Greek symbols |
| `StandardFonts.ZapfDingbats` | `"ZapfDingbats"` | Decorative symbols, checkmarks, arrows |

**ALL 14 standard fonts use WinAnsi encoding.** They NEVER support Cyrillic, CJK, Arabic, Hebrew, Thai, or emoji.

---

## PDFFont Class -- Complete API

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this font belongs to |
| `name` | `string` | The name of this font |
| `ref` | `PDFRef` | Unique reference within the document |

### widthOfTextAtSize()

```typescript
font.widthOfTextAtSize(text: string, size: number): number
```

Returns the width of a text string when rendered at the given font size, measured in PDF points. ALWAYS use this for horizontal text positioning and alignment calculations.

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | `string` | The text to measure |
| `size` | `number` | Font size in points |

**Returns:** `number` -- width in PDF points

---

### heightAtSize()

```typescript
font.heightAtSize(size: number, options?: { descender?: boolean }): number
```

Returns the height of the font at the given size, measured in PDF points.

| Parameter | Type | Description |
|-----------|------|-------------|
| `size` | `number` | Font size in points |
| `options.descender` | `boolean \| undefined` | Whether to include the descender in the height calculation |

**Returns:** `number` -- height in PDF points

---

### sizeAtHeight()

```typescript
font.sizeAtHeight(height: number): number
```

Computes the font size needed to achieve a target height in PDF points. The inverse of `heightAtSize()`.

| Parameter | Type | Description |
|-----------|------|-------------|
| `height` | `number` | Target height in PDF points |

**Returns:** `number` -- the font size that produces the target height

---

### getCharacterSet()

```typescript
font.getCharacterSet(): number[]
```

Returns an array of unicode code points that this font can render. Use this to check whether a font supports specific characters before attempting to draw text.

**Returns:** `number[]` -- array of supported unicode code points

---

### encodeText()

```typescript
font.encodeText(text: string): PDFHexString
```

Encodes a text string for use with this font. NEVER call this directly -- `drawText()` handles encoding automatically. This method exists for low-level PDF content stream manipulation only.

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | `string` | The text to encode |

**Returns:** `PDFHexString`

---

### embed()

```typescript
font.embed(): Promise<void>
```

Embeds the font data in the document. NEVER call this directly -- `pdfDoc.save()` calls it automatically for all embedded fonts. This method exists for advanced use cases where explicit embedding is needed before save.

**Returns:** `Promise<void>`
