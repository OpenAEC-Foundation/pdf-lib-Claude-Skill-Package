# Font Anti-Pattern Catalog

## Anti-Pattern 1: Assuming Standard Fonts Handle All Latin Text

**What developers do**:
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText(userInput, { font }); // userInput comes from a form/database
```

**Why it fails**: User input may contain characters outside the WinAnsi range. Even "Latin" text can include characters like smart quotes, em dashes, or diacritics that WinAnsi does not support.

**Correct approach**: ALWAYS use a custom font when text comes from user input, databases, or any external source. ONLY use standard fonts when you control the exact string content and have verified every character is within WinAnsi.

---

## Anti-Pattern 2: Registering fontkit AFTER embedFont

**What developers do**:
```typescript
const font = await pdfDoc.embedFont(fontBytes); // CRASHES — fontkit not yet registered
pdfDoc.registerFontkit(fontkit); // Too late
```

**Why it fails**: `registerFontkit()` MUST be called before `embedFont()` when passing font bytes. The order is non-negotiable.

**Correct approach**:
```typescript
pdfDoc.registerFontkit(fontkit); // FIRST
const font = await pdfDoc.embedFont(fontBytes); // THEN embed
```

---

## Anti-Pattern 3: Forgetting updateFieldAppearances for Form Fields

**What developers do**:
```typescript
pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes);
form.getTextField('name').setText('Unicode text');
// Missing: form.updateFieldAppearances(customFont)
const pdfBytes = await pdfDoc.save();
```

**Why it fails**: Embedding a custom font does NOT automatically apply it to form fields. Without `updateFieldAppearances(customFont)`, the form uses Helvetica to render field appearances. The `save()` method calls `updateFieldAppearances()` by default, but without a font argument, it uses Helvetica.

**Correct approach**:
```typescript
pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes);
form.getTextField('name').setText('Unicode text');
form.updateFieldAppearances(customFont); // MUST call with custom font
const pdfBytes = await pdfDoc.save({ updateFieldAppearances: false });
```

---

## Anti-Pattern 4: Using Multiple Standard Fonts to "Cover" Unicode

**What developers do**:
```typescript
// Trying to use Symbol or ZapfDingbats for special characters
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica);
const symbol = await pdfDoc.embedFont(StandardFonts.Symbol);
// Attempting to mix fonts for broader coverage
```

**Why it fails**: ALL 14 standard fonts use WinAnsi encoding. Symbol and ZapfDingbats have different character mappings but are still WinAnsi-limited. There is NO combination of standard fonts that provides Unicode coverage.

**GitHub issue**: #1665

**Correct approach**: Use a single custom font with broad Unicode coverage (e.g., Noto Sans) or multiple custom fonts for specific scripts.

---

## Anti-Pattern 5: Ignoring Font Subsetting for Large Fonts

**What developers do**:
```typescript
// CJK font file: 15+ MB
const cjkFont = await pdfDoc.embedFont(cjkFontBytes);
// Entire 15 MB font embedded even if only 5 characters are used
```

**Why it fails**: The PDF output contains the ENTIRE font file, causing massive file sizes. A document using 10 CJK characters from a 15 MB font produces a 15+ MB PDF.

**Correct approach**:
```typescript
const cjkFont = await pdfDoc.embedFont(cjkFontBytes, { subset: true });
// Only the 10 used glyphs are embedded — PDF is kilobytes, not megabytes
```

**Rule**: ALWAYS use `{ subset: true }` for custom fonts in production, unless the PDF will be further edited and may need additional glyphs.

---

## Anti-Pattern 6: Not Validating Font File Contents

**What developers do**:
```typescript
// Using a font file without checking glyph coverage
const font = await pdfDoc.embedFont(someRandomFont);
page.drawText('Arabic text', { font }); // Font does not contain Arabic glyphs
```

**Why it fails**: Embedding a custom font does NOT guarantee it supports all Unicode. The font file must contain glyphs for the specific characters you need. A Latin-only TTF file will fail on CJK characters even with fontkit registered.

**Correct approach**: Verify the font covers your needed character ranges before embedding. Use `font.getCharacterSet()` to check programmatically.

---

## Anti-Pattern 7: Catching WinAnsi Errors Instead of Preventing Them

**What developers do**:
```typescript
try {
  page.drawText(text, { font: helvetica });
} catch (e) {
  // Silently swallow the error or replace with '?'
  page.drawText(text.replace(/[^\x00-\x7F]/g, '?'), { font: helvetica });
}
```

**Why it fails**: This destroys data. Users lose their text content. The error is a signal that the wrong font is being used, not that the text should be corrupted.

**Correct approach**: Use a custom font that supports the characters. If you must handle unknown input, check character support proactively:

```typescript
const charSet = new Set(font.getCharacterSet());
const unsupported = [...text].filter(ch => !charSet.has(ch.codePointAt(0)!));
if (unsupported.length > 0) {
  // Switch to custom font — do NOT corrupt the text
}
```

---

## Anti-Pattern 8: Embedding the Same Font Multiple Times

**What developers do**:
```typescript
// In a loop generating pages
for (const data of pageDataArray) {
  const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
  const font = await pdfDoc.embedFont(fontBytes); // Embeds AGAIN each iteration
  const page = pdfDoc.addPage();
  page.drawText(data.text, { font });
}
```

**Why it fails**: Each `embedFont()` call adds a NEW copy of the font to the document. A 500-page document embeds the font 500 times, causing enormous file sizes.

**Correct approach**: Embed the font ONCE, reuse across all pages:

```typescript
const fontBytes = fs.readFileSync('fonts/NotoSans-Regular.ttf');
const font = await pdfDoc.embedFont(fontBytes, { subset: true });

for (const data of pageDataArray) {
  const page = pdfDoc.addPage();
  page.drawText(data.text, { font }); // Reuse the same font reference
}
```

---

## Anti-Pattern 9: Using embedFont for Standard Fonts Instead of embedStandardFont

**What developers do**:
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica); // Async, returns Promise
```

**Why it matters**: This works, but `embedStandardFont()` is synchronous and faster:

```typescript
const font = pdfDoc.embedStandardFont(StandardFonts.Helvetica); // Sync, no await
```

This is NOT an error, but using the async version unnecessarily adds complexity. Use `embedStandardFont()` when you know you want a standard font.

---

## Anti-Pattern 10: Expecting save() to Auto-Fix Font Issues

**What developers do**:
```typescript
form.getTextField('name').setText('Non-Latin text');
// Expecting save() to figure out the right font
const pdfBytes = await pdfDoc.save();
```

**Why it fails**: `save()` calls `updateFieldAppearances()` with NO font argument by default. This uses Helvetica. If the text contains non-WinAnsi characters, `save()` throws the WinAnsi encoding error.

**Correct approach**: Explicitly update appearances with a custom font before saving:

```typescript
form.updateFieldAppearances(customFont);
const pdfBytes = await pdfDoc.save({ updateFieldAppearances: false });
```

---

## Anti-Pattern 11: Forgetting to Install @pdf-lib/fontkit Package

**What developers do**:
```typescript
import fontkit from '@pdf-lib/fontkit'; // Import exists but package not installed
```

**Why it fails**: The import resolves to nothing at runtime, or throws a module-not-found error. This is an npm/build-time error, not a pdf-lib error, but it is a common stumbling block.

**Correct approach**:
```bash
npm install @pdf-lib/fontkit
```

ALWAYS verify the package is in `package.json` dependencies before using it.

---

## Anti-Pattern 12: Using fontkit from Wrong Package

**What developers do**:
```typescript
import fontkit from 'fontkit'; // WRONG package — this is the original fontkit, not pdf-lib's fork
```

**Why it fails**: pdf-lib requires its own fork: `@pdf-lib/fontkit`. The original `fontkit` package has a different API and is NOT compatible with `pdfDoc.registerFontkit()`.

**Correct approach**:
```typescript
import fontkit from '@pdf-lib/fontkit'; // CORRECT — pdf-lib's fork
```

---

## Summary: Font Error Prevention Rules

1. NEVER use standard fonts for user-supplied text or multilingual content
2. ALWAYS call `registerFontkit()` BEFORE `embedFont()` with custom font bytes
3. ALWAYS call `form.updateFieldAppearances(customFont)` when using non-Latin text in forms
4. ALWAYS use `{ subset: true }` for custom fonts in production
5. NEVER embed the same font multiple times — embed once, reuse everywhere
6. NEVER catch and suppress WinAnsi errors — fix the font choice instead
7. ALWAYS use `@pdf-lib/fontkit` (not `fontkit`) as the package
8. ALWAYS verify font files contain the glyphs you need
