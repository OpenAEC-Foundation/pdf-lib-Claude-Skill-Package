---
name: pdflib-errors-fonts
description: "Diagnoses and fixes font-related errors in pdf-lib including WinAnsi encoding failures, missing fontkit registration, unicode character support issues, font embedding failures, and form field font problems. Activates when encountering font errors, WinAnsi encoding errors, unicode rendering issues, or fontkit problems."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Font Error Diagnosis and Resolution

## Critical Warning

Font errors are the **#1 source of issues** in pdf-lib. The root cause is almost always the same: standard fonts (Helvetica, Courier, TimesRoman) use WinAnsi encoding, which supports ONLY a subset of Latin-1 characters. Any character outside this range throws a runtime error.

## Diagnostic Decision Tree

```
Font error encountered?
|
+-- Error message contains "WinAnsi cannot encode"?
|   +-- YES --> Character is outside WinAnsi range
|   |   +-- Using StandardFonts? --> Switch to custom font with fontkit
|   |   +-- Using custom font? --> Font file lacks the required glyph
|   |
+-- Error message contains "fontkit" or "No fontkit instance"?
|   +-- YES --> fontkit not registered
|   |   +-- Did you call pdfDoc.registerFontkit(fontkit)?
|   |   +-- Did you install @pdf-lib/fontkit?
|   |   +-- Did you import fontkit correctly?
|   |
+-- Form field text not rendering (blank until clicked)?
|   +-- YES --> Appearance streams not updated
|   |   +-- Call form.updateFieldAppearances(customFont)
|   |   +-- Or pass { updateFieldAppearances: true } to save()
|   |
+-- Form field shows wrong characters or boxes?
|   +-- YES --> Default Helvetica cannot render your text
|   |   +-- Register fontkit + embed custom font
|   |   +-- Call form.updateFieldAppearances(customFont) AFTER setText()
|   |
+-- Diacritics (accented characters) fail?
    +-- Using StandardFonts?
    |   +-- Some diacritics work (WinAnsi subset: e, u, n, a, o)
    |   +-- Others fail (characters outside WinAnsi range)
    |   +-- SAFEST: Switch to custom font for ANY diacritics
    +-- Using custom font?
        +-- Verify the font file contains the required glyphs
```

## Font Compatibility Matrix

| Character Type | Standard Fonts (14) | Custom Font (TTF/OTF + fontkit) |
|---------------|:---:|:---:|
| Basic Latin (A-Z, a-z, 0-9) | YES | YES |
| Common punctuation (!@#$%&) | YES | YES |
| WinAnsi diacritics (e, u, n) | YES | YES |
| Extended diacritics (outside WinAnsi) | NO | YES* |
| Cyrillic | NO | YES* |
| Chinese/Japanese/Korean (CJK) | NO | YES* |
| Arabic | NO | YES* |
| Hebrew | NO | YES* |
| Thai | NO | YES* |
| Emoji | NO | DEPENDS* |

\* Requires a font file that contains the specific glyphs needed.

## Error 1: WinAnsi Encoding Failure

**Error message**: `"WinAnsi cannot encode "X" (0xNNNN)"`

**Cause**: StandardFonts use WinAnsi encoding. ANY character outside this encoding throws this error at runtime when calling `drawText()`, `encodeText()`, or when `save()` triggers `updateFieldAppearances`.

**Broken code**:
```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage();

// CRASHES: WinAnsi cannot encode Cyrillic characters
page.drawText('Hello World', { font, size: 12 });
```

**Fixed code**:
```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);

const fontBytes = fs.readFileSync('path/to/NotoSans-Regular.ttf');
const font = await pdfDoc.embedFont(fontBytes);
const page = pdfDoc.addPage();

// WORKS: Custom font supports Unicode
page.drawText('Hello World', { font, size: 12 });
```

**GitHub issues**: #1759, #1566, #561, #715, #1010, #746, #1297

## Error 2: Missing fontkit Registration

**Error message**: `"No fontkit instance registered"` or similar error when calling `embedFont()` with raw font bytes.

**Cause**: Calling `pdfDoc.embedFont(fontBytes)` with `Uint8Array` or `ArrayBuffer` font data WITHOUT first calling `pdfDoc.registerFontkit(fontkit)`.

**Broken code**:
```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
// MISSING: pdfDoc.registerFontkit(fontkit)
const fontBytes = fs.readFileSync('path/to/font.ttf');
const font = await pdfDoc.embedFont(fontBytes); // CRASHES
```

**Fixed code**:
```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit); // MUST call BEFORE embedFont with bytes
const fontBytes = fs.readFileSync('path/to/font.ttf');
const font = await pdfDoc.embedFont(fontBytes); // Works
```

**Rule**: ALWAYS call `pdfDoc.registerFontkit(fontkit)` before ANY call to `embedFont()` that passes font bytes (Uint8Array, ArrayBuffer, or base64 string). fontkit is NOT needed for `StandardFonts` enum values.

## Error 3: Standard Fonts Unicode Limitation

Standard fonts support ONLY the WinAnsi character set (a subset of Latin-1). There is NO workaround within standard fonts.

**What works with standard fonts**:
- ASCII letters, digits, common punctuation
- A subset of accented characters within WinAnsi (e.g., e, u, n, a, o, i)

**What ALWAYS fails with standard fonts**:
- Cyrillic, Greek (beyond WinAnsi subset), Arabic, Hebrew, Thai, CJK
- Emoji of any kind
- Unicode fraction slash, many mathematical symbols
- Characters with code points above U+00FF (with some WinAnsi exceptions)

**The ONLY fix**: Embed a custom font via fontkit that contains the required glyphs.

## Error 4: Font Not Applied to Form Fields

**Symptom**: Form field text appears as empty boxes, wrong characters, or is invisible until the field is clicked.

**Cause**: Form fields use Helvetica by default. Setting non-Latin text via `setText()` without updating field appearances with a Unicode-capable font causes rendering failures.

**Broken code**:
```typescript
const form = pdfDoc.getForm();
form.getTextField('name').setText('Non-Latin text');
// Field renders incorrectly — default Helvetica cannot display these characters
const pdfBytes = await pdfDoc.save();
```

**Fixed code**:
```typescript
import fontkit from '@pdf-lib/fontkit';

pdfDoc.registerFontkit(fontkit);
const fontBytes = fs.readFileSync('path/to/NotoSans-Regular.ttf');
const customFont = await pdfDoc.embedFont(fontBytes);

const form = pdfDoc.getForm();
form.getTextField('name').setText('Non-Latin text');
form.updateFieldAppearances(customFont); // MUST call AFTER setText()

const pdfBytes = await pdfDoc.save();
```

**Rule**: ALWAYS call `form.updateFieldAppearances(customFont)` after setting non-Latin text in form fields. This applies to ALL field types that display text.

**GitHub issues**: #1750, #1538, #1378, #1488, #1152

## Error 5: Diacritics Failures with Standard Fonts

**Symptom**: Some accented characters render correctly, others throw WinAnsi errors.

**Cause**: WinAnsi encoding includes SOME accented Latin characters but NOT all. This creates unpredictable behavior where some diacritics work and others crash.

**Rule**: NEVER rely on standard fonts for text that MAY contain diacritics. If your application handles user input, international names, or multilingual content, ALWAYS use a custom font. The WinAnsi subset is not documented clearly enough to safely predict which diacritics will work.

**Safe approach**:
```typescript
import { PDFDocument } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

// ALWAYS use custom font when diacritics are possible
const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);
const font = await pdfDoc.embedFont(fontBytes);

// Safe for ALL diacritics the font file supports
page.drawText('Resume, naive, Munchen, Malmo', { font, size: 12 });
```

## Error 6: Form Dropdown with Non-Latin Options

**Error message**: WinAnsi encoding failure when dropdown options contain non-Latin characters.

**Cause**: Dropdown option values are encoded using the default font. Non-Latin option strings trigger WinAnsi errors.

**GitHub issue**: #1152

**Fix**: Same pattern — register fontkit, embed custom font, call `form.updateFieldAppearances(customFont)`.

## Quick Fix Checklist

When encountering ANY font error in pdf-lib:

1. **Install fontkit**: `npm install @pdf-lib/fontkit`
2. **Import fontkit**: `import fontkit from '@pdf-lib/fontkit'`
3. **Register fontkit**: `pdfDoc.registerFontkit(fontkit)` — call ONCE per document, BEFORE embedding fonts
4. **Obtain font file**: Get a `.ttf` or `.otf` file that contains the glyphs you need (e.g., Noto Sans for broad Unicode coverage, or a CJK-specific font)
5. **Embed font**: `const font = await pdfDoc.embedFont(fontBytes)`
6. **Use font**: Pass `font` to `drawText()` options or call `form.updateFieldAppearances(font)` for forms
7. **Consider subsetting**: Use `{ subset: true }` to reduce file size when embedding large font files

## Font Subsetting for File Size

When using large font files (especially CJK fonts that can be 10+ MB), ALWAYS enable subsetting:

```typescript
const font = await pdfDoc.embedFont(fontBytes, { subset: true });
```

This embeds ONLY the glyphs actually used in the document, dramatically reducing output file size.

## Character Support Verification

To check if a font supports specific characters BEFORE drawing:

```typescript
const font = await pdfDoc.embedFont(fontBytes);
const charSet = font.getCharacterSet(); // number[] of supported code points

function canRender(text: string, supportedCodePoints: number[]): boolean {
  const supported = new Set(supportedCodePoints);
  for (let i = 0; i < text.length; i++) {
    if (!supported.has(text.charCodeAt(i))) return false;
  }
  return true;
}

if (!canRender(userText, charSet)) {
  // Fall back to a different font or warn the user
}
```

## Known GitHub Issues Catalog

| Issue | Category | Description |
|-------|----------|-------------|
| #1759 | WinAnsi | Standard fonts cannot encode characters outside WinAnsi |
| #1566 | CJK | Chinese character support failures |
| #1450 | BiDi | Arabic bidirectional text problems |
| #561 | Unicode | Hebrew/UTF-8 support |
| #715 | Unicode | Cyrillic/Russian symbol encoding |
| #1010 | Rendering | Thai font rendering with unwanted spaces |
| #746 | Rendering | Khmer font rendering problems |
| #1297 | Unicode | Unicode fraction slash not rendering |
| #50, #217 | Emoji | Emoji embedding issues |
| #1270 | Standard | ZapfDingbats font errors |
| #1665 | Standard | Cannot merge multiple Standard Fonts for mixed Unicode |
| #1152 | Forms | Non-Latin dropdown values trigger WinAnsi failures |
| #726 | Custom | Embedded font glyph width calculation errors |
| #1750, #1538, #1378 | Forms | Font not applied to form fields |
| #1488 | Diacritics | Diacritics in field values |
| #1584 | Forms | Textarea only shows value on focus (appearance stream issue) |

## Reference Files

- [Font-related method signatures](references/methods.md)
- [Error reproduction and fix examples](references/examples.md)
- [Font anti-pattern catalog](references/anti-patterns.md)
