# Font-Related Method Signatures (Error Context)

## Font Registration and Embedding

### registerFontkit()

```typescript
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

- MUST be called BEFORE `embedFont()` with custom font bytes
- NOT needed for `StandardFonts` enum values
- Call ONCE per `PDFDocument` instance
- Accepts the fontkit instance from `@pdf-lib/fontkit`

### embedFont()

```typescript
pdfDoc.embedFont(
  font: StandardFonts | string | Uint8Array | ArrayBuffer,
  options?: EmbedFontOptions
): Promise<PDFFont>
```

**EmbedFontOptions**:

| Property | Type | Default | Purpose |
|----------|------|---------|---------|
| `subset` | `boolean \| undefined` | `undefined` | Include only used glyphs (reduces file size) |
| `customName` | `string \| undefined` | `undefined` | Custom name for the embedded font |
| `features` | `TypeFeatures \| undefined` | `undefined` | OpenType font feature configuration |

**Error conditions**:
- Throws if `font` is `Uint8Array`/`ArrayBuffer`/`string` (base64) and fontkit is NOT registered
- Throws if font bytes are corrupt or not a valid TTF/OTF file

### embedStandardFont()

```typescript
pdfDoc.embedStandardFont(font: StandardFonts): PDFFont
```

- Synchronous (no `await` needed)
- NEVER throws for font registration issues
- WILL throw WinAnsi errors when the returned font is used with unsupported characters

## PDFFont Methods Relevant to Error Diagnosis

### encodeText()

```typescript
font.encodeText(text: string): PDFHexString
```

- Throws `"WinAnsi cannot encode"` for standard fonts with unsupported characters
- Called internally by `drawText()` — the error surfaces during drawing, not during font embedding

### getCharacterSet()

```typescript
font.getCharacterSet(): number[]
```

- Returns array of Unicode code points the font supports
- Use this to verify character support BEFORE attempting to draw
- Works for both standard and custom fonts

### widthOfTextAtSize()

```typescript
font.widthOfTextAtSize(text: string, size: number): number
```

- Also throws WinAnsi errors for standard fonts with unsupported characters
- Error occurs during measurement, not during drawing

### heightAtSize()

```typescript
font.heightAtSize(size: number, options?: { descender?: boolean }): number
```

- Does NOT throw font encoding errors (size-based, not text-based)

## Form Methods Relevant to Font Errors

### updateFieldAppearances()

```typescript
form.updateFieldAppearances(font?: PDFFont): void
```

- When called WITHOUT a font argument: uses default Helvetica
- When called WITH a custom font: uses that font for ALL field appearances
- MUST be called AFTER `setText()` calls to render non-Latin text
- Automatically called by `pdfDoc.save()` when `updateFieldAppearances: true` (default)

### PDFTextField.updateAppearances()

```typescript
field.updateAppearances(font: PDFFont, provider?: AppearanceProvider): void
```

- Updates appearance for a SINGLE field with a specific font
- Use when different fields need different fonts

### PDFTextField.defaultUpdateAppearances()

```typescript
field.defaultUpdateAppearances(font: PDFFont): void
```

- Applies default appearance with the specified font

### save() — Font-Related Options

```typescript
pdfDoc.save(options?: SaveOptions): Promise<Uint8Array>
```

**Relevant SaveOptions property**:

| Property | Type | Default | Impact |
|----------|------|---------|--------|
| `updateFieldAppearances` | `boolean` | `true` | When `true`, calls `form.updateFieldAppearances()` during save. Set to `false` to skip (use when you have already called it manually with a custom font). |

**Critical note**: If `updateFieldAppearances` is `true` (default) and form fields contain non-Latin text, `save()` will attempt to update appearances with the default Helvetica font, which triggers WinAnsi errors. To prevent this:

```typescript
// Option A: Update appearances manually with custom font, then skip during save
form.updateFieldAppearances(customFont);
const pdfBytes = await pdfDoc.save({ updateFieldAppearances: false });

// Option B: Update appearances manually, let save() run it again (harmless if custom font was set)
form.updateFieldAppearances(customFont);
const pdfBytes = await pdfDoc.save();
```

## StandardFonts Enum — Complete List

| Enum Key | WinAnsi Only |
|----------|:---:|
| `StandardFonts.Courier` | YES |
| `StandardFonts.CourierBold` | YES |
| `StandardFonts.CourierOblique` | YES |
| `StandardFonts.CourierBoldOblique` | YES |
| `StandardFonts.Helvetica` | YES |
| `StandardFonts.HelveticaBold` | YES |
| `StandardFonts.HelveticaOblique` | YES |
| `StandardFonts.HelveticaBoldOblique` | YES |
| `StandardFonts.TimesRoman` | YES |
| `StandardFonts.TimesRomanBold` | YES |
| `StandardFonts.TimesRomanItalic` | YES |
| `StandardFonts.TimesRomanBoldItalic` | YES |
| `StandardFonts.Symbol` | YES |
| `StandardFonts.ZapfDingbats` | YES |

ALL 14 standard fonts are WinAnsi-only. There are NO exceptions.
