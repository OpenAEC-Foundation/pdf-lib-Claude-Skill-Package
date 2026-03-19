# pdflib-syntax-fonts -- Anti-Patterns Reference

## AP-001: Using Standard Fonts with Non-WinAnsi Characters

**Severity:** CRITICAL -- Runtime crash

**Error:** `"WinAnsi cannot encode "X" (0xNNNN)"`

**Broken code:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Cyrillic/CJK/Arabic text', { font })
// CRASHES at runtime with WinAnsi encoding error
```

**Fix:**
```typescript
import fontkit from '@pdf-lib/fontkit'

pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(customFontBytes) // TTF/OTF with required glyphs
page.drawText('Cyrillic/CJK/Arabic text', { font })
```

**Why:** All 14 standard fonts (Helvetica, Times Roman, Courier, Symbol, ZapfDingbats) are hardcoded to WinAnsi encoding, which supports only ~218 characters. There is NO way to make standard fonts support unicode. This is the single most reported issue in pdf-lib with dozens of open GitHub issues.

**Related issues:** #1759, #1566, #1450, #561, #715, #1010, #746, #1297, #50, #217, #1270, #1665

---

## AP-002: Forgetting to Register fontkit

**Severity:** CRITICAL -- Runtime crash

**Error:** Runtime error when calling `embedFont()` with byte data

**Broken code:**
```typescript
const pdfDoc = await PDFDocument.create()
// MISSING: pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes) // CRASHES
```

**Fix:**
```typescript
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit) // MUST come before embedFont() with bytes
const font = await pdfDoc.embedFont(fontBytes) // Now works
```

**Why:** pdf-lib does not bundle fontkit to keep the core library lightweight. Custom font parsing requires fontkit as an explicit dependency. The `registerFontkit()` call gives pdf-lib access to the font parser.

**Rule:** ALWAYS call `registerFontkit()` before ANY call to `embedFont()` that passes font bytes (Uint8Array, ArrayBuffer, or base64 string). It is NOT needed for `StandardFonts` enum values or `embedStandardFont()`.

---

## AP-003: Forgetting to Await embedFont()

**Severity:** HIGH -- Silent failure or type error

**Broken code:**
```typescript
const font = pdfDoc.embedFont(StandardFonts.Helvetica) // Missing await!
page.drawText('Hello', { font })
// font is a Promise<PDFFont>, not a PDFFont
// May silently fail or throw a confusing type error
```

**Fix:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font })
```

**Why:** `embedFont()` ALWAYS returns `Promise<PDFFont>`. Passing a Promise object where a PDFFont is expected causes unpredictable behavior. TypeScript will catch this at compile time, but JavaScript will not.

**Alternative:** Use `embedStandardFont()` for synchronous embedding (standard fonts only):
```typescript
const font = pdfDoc.embedStandardFont(StandardFonts.Helvetica) // No await needed
```

---

## AP-004: Embedding the Same Font Multiple Times

**Severity:** MEDIUM -- File size bloat

**Broken code:**
```typescript
// Embedding the same font repeatedly wastes space
for (const pageData of pages) {
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica) // Redundant!
  page.drawText(pageData.text, { font })
}
```

**Fix:**
```typescript
// Embed once, reuse across all pages
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)

for (const pageData of pages) {
  page.drawText(pageData.text, { font }) // Reuse the same PDFFont
}
```

**Why:** Each call to `embedFont()` embeds a new copy of the font data in the PDF. For standard fonts the overhead is small, but for custom fonts (which can be several MB), this dramatically increases file size. ALWAYS embed each font once and reuse the returned `PDFFont` object.

---

## AP-005: Using Custom Font Bytes with embedStandardFont()

**Severity:** CRITICAL -- Will not work

**Broken code:**
```typescript
// embedStandardFont() ONLY accepts StandardFonts enum values
const font = pdfDoc.embedStandardFont(fontBytes) // TypeScript error / runtime crash
```

**Fix:**
```typescript
// For custom fonts, ALWAYS use embedFont() (async)
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes)
```

**Why:** `embedStandardFont()` is a synchronous method that ONLY works with the 14 built-in `StandardFonts` enum values. It cannot parse TTF/OTF font files.

---

## AP-006: Not Updating Form Field Appearances After Custom Font

**Severity:** HIGH -- Fields appear blank or with wrong font

**Broken code:**
```typescript
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)

form.getTextField('name').setText('Unicode text')
// MISSING: form.updateFieldAppearances(customFont)
const pdfBytes = await pdfDoc.save()
// Field may appear blank or show incorrect characters
```

**Fix:**
```typescript
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)

form.getTextField('name').setText('Unicode text')
form.updateFieldAppearances(customFont) // MUST call this
const pdfBytes = await pdfDoc.save()
```

**Why:** Form fields have their own appearance streams that default to Helvetica. Setting text on a field does not automatically update the visual rendering with the custom font. You MUST explicitly call `form.updateFieldAppearances(customFont)` to re-render field appearances using the custom font.

**Related issues:** #1750, #1538, #1378, #1488

---

## AP-007: Assuming Standard Fonts Support All Accented Characters

**Severity:** MEDIUM -- Unexpected encoding errors

**Broken code:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
// Assumes ALL accented characters work
page.drawText(userProvidedText, { font })
// May crash if text contains characters outside WinAnsi
```

**Fix:**
```typescript
// Option A: Check before drawing
const charSet = new Set(font.getCharacterSet())
const allSupported = [...userProvidedText].every(
  ch => charSet.has(ch.charCodeAt(0))
)

if (allSupported) {
  page.drawText(userProvidedText, { font })
} else {
  // Fall back to custom font
  pdfDoc.registerFontkit(fontkit)
  const customFont = await pdfDoc.embedFont(fontBytes)
  page.drawText(userProvidedText, { font: customFont })
}

// Option B: Always use custom font for user-provided text
pdfDoc.registerFontkit(fontkit)
const safeFont = await pdfDoc.embedFont(fontBytes)
page.drawText(userProvidedText, { font: safeFont })
```

**Why:** WinAnsi encoding covers ONLY a subset of accented Latin characters. Characters like certain Eastern European diacritics, Turkish special characters, or Vietnamese tone marks are NOT included. When processing user-provided text, NEVER assume standard fonts will handle all accented characters.

---

## AP-008: Not Subsetting Large Custom Fonts

**Severity:** LOW -- Unnecessarily large PDF files

**Broken code:**
```typescript
// Full CJK font can be 15-20 MB
const font = await pdfDoc.embedFont(largeCjkFontBytes)
// Entire font embedded even if only 10 characters are used
page.drawText('Small amount of text', { font })
// Output PDF is 15+ MB for a single page
```

**Fix:**
```typescript
const font = await pdfDoc.embedFont(largeCjkFontBytes, { subset: true })
page.drawText('Small amount of text', { font })
// Only the ~10 used glyphs are embedded -- much smaller file
```

**Why:** Without subsetting, the entire font file is embedded in the PDF. For large fonts (especially CJK fonts which contain thousands of glyphs), this adds megabytes to the output. ALWAYS use `{ subset: true }` for production PDFs unless the document will be further edited with additional text.

---

## AP-009: Using getCharacterSet() in a Hot Loop

**Severity:** LOW -- Performance issue

**Broken code:**
```typescript
for (const text of thousandsOfStrings) {
  const charSet = font.getCharacterSet() // Called thousands of times!
  const supported = [...text].every(ch => charSet.includes(ch.charCodeAt(0)))
}
```

**Fix:**
```typescript
// Cache the character set once
const charSet = new Set(font.getCharacterSet()) // Set for O(1) lookup

for (const text of thousandsOfStrings) {
  const supported = [...text].every(ch => charSet.has(ch.charCodeAt(0)))
}
```

**Why:** `getCharacterSet()` returns a new array on each call, and `Array.includes()` is O(n). For repeated checks, cache the result in a `Set` for O(1) lookups.
