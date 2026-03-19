# Anti-Patterns — What NOT to Do with pdf-lib 1.x

Every entry shows the broken code, explains WHY it fails, and provides the correct alternative.

---

## AP-001: Standard Fonts with Unicode Text

**Severity**: CRITICAL — the #1 source of pdf-lib errors

```typescript
// BROKEN — StandardFonts use WinAnsi encoding ONLY
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Привет мир', { font }); // THROWS: "WinAnsi cannot encode..."
```

**Why it fails**: All 14 standard fonts (Helvetica, TimesRoman, Courier families, Symbol, ZapfDingbats) support ONLY the WinAnsi character set (basic Latin + limited Western European). Cyrillic, CJK, Arabic, Hebrew, Thai, and emoji characters are NOT in this set.

```typescript
// CORRECT — use custom font with fontkit
import fontkit from '@pdf-lib/fontkit';

pdfDoc.registerFontkit(fontkit);
const font = await pdfDoc.embedFont(fontBytes); // TTF/OTF with required glyphs
page.drawText('Привет мир', { font }); // Works
```

---

## AP-002: Missing fontkit Registration

**Severity**: CRITICAL — always throws

```typescript
// BROKEN — fontkit not registered
const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(fontBytes); // THROWS error
```

**Why it fails**: pdf-lib requires fontkit to parse custom font files (TTF/OTF). Without registration, it cannot interpret the font binary data.

```typescript
// CORRECT — register fontkit BEFORE embedding
import fontkit from '@pdf-lib/fontkit';

const pdfDoc = await PDFDocument.create();
pdfDoc.registerFontkit(fontkit);  // MUST come first
const font = await pdfDoc.embedFont(fontBytes); // Works
```

---

## AP-003: Forgetting to Await Async Methods

**Severity**: CRITICAL — causes silent failures or runtime errors

```typescript
// BROKEN — font is a Promise, not a PDFFont
const font = pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font }); // font is Promise<PDFFont>, not PDFFont
```

**Why it fails**: `embedFont()` returns a Promise. Without `await`, you pass the Promise object itself to `drawText()`, which silently fails or throws a type error.

```typescript
// CORRECT — always await async methods
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font });
```

**All async methods that ALWAYS need await**:
- `PDFDocument.create()`, `PDFDocument.load()`
- `pdfDoc.embedFont()`, `pdfDoc.embedPng()`, `pdfDoc.embedJpg()`
- `pdfDoc.embedPdf()`, `pdfDoc.embedPage()`, `pdfDoc.embedPages()`
- `pdfDoc.copyPages()`
- `pdfDoc.save()`, `pdfDoc.saveAsBase64()`
- `pdfDoc.attach()`

---

## AP-004: Cross-Document Page Addition Without copyPages()

**Severity**: CRITICAL — always throws

```typescript
// BROKEN — direct page addition from another document
const sourcePage = sourceDoc.getPage(0);
targetDoc.addPage(sourcePage); // THROWS error
```

**Why it fails**: Pages are bound to their source document's internal context. Adding a page directly references objects that do not exist in the target document.

```typescript
// CORRECT — ALWAYS use copyPages() on the destination document
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage); // Works
```

---

## AP-005: Color Values 0-255 Instead of 0.0-1.0

**Severity**: HIGH — produces wrong colors silently

```typescript
// BROKEN — rgb() takes 0.0-1.0, NOT 0-255
const red = rgb(255, 0, 0);   // Values > 1.0 produce unexpected results
```

**Why it fails**: pdf-lib color functions use the PDF specification's normalized range (0.0 to 1.0), not the CSS/HTML 0-255 range.

```typescript
// CORRECT — use 0.0 to 1.0
const red = rgb(1, 0, 0);
const halfRed = rgb(0.5, 0, 0);
const teal = rgb(0, 0.53, 0.71);
```

---

## AP-006: Coordinate System Confusion (Y-axis)

**Severity**: HIGH — content appears at wrong position

```typescript
// BROKEN — assumes y=0 is top of page (HTML/CSS convention)
page.drawText('Header', { x: 50, y: 0 }); // Renders at BOTTOM of page
```

**Why it fails**: PDF uses a bottom-left origin where Y increases upward. y=0 is the BOTTOM of the page, not the top.

```typescript
// CORRECT — calculate from page height for top-aligned content
const { height } = page.getSize();
page.drawText('Header', { x: 50, y: height - 50 }); // Near top
page.drawText('Footer', { x: 50, y: 30 });            // Near bottom
```

---

## AP-007: Image Aspect Ratio Distortion

**Severity**: MEDIUM — image appears stretched or squished

```typescript
// BROKEN — arbitrary width and height distorts image
page.drawImage(image, {
  x: 50, y: 50,
  width: 300,
  height: 300,  // Aspect ratio broken if image is not square
});
```

**Why it fails**: Setting independent width and height values ignores the image's native aspect ratio.

```typescript
// CORRECT — use scale() or scaleToFit()
const dims = image.scale(0.5); // Uniform 50% scale
page.drawImage(image, {
  x: 50, y: 50,
  width: dims.width,
  height: dims.height,
});

// Or fit within bounds
const dims2 = image.scaleToFit(300, 200);
page.drawImage(image, {
  x: 50, y: 50,
  width: dims2.width,
  height: dims2.height,
});
```

---

## AP-008: Unsupported Image Formats

**Severity**: HIGH — always throws

```typescript
// BROKEN — no embedGif, embedBmp, embedSvg, embedWebp, etc.
const image = await pdfDoc.embedImage(gifBytes);  // Method does not exist
```

**Why it fails**: pdf-lib ONLY supports PNG (`embedPng()`) and JPEG (`embedJpg()`). There is no generic `embedImage()` method and no support for GIF, BMP, TIFF, WebP, SVG, AVIF, or HEIC.

```typescript
// CORRECT — convert to PNG or JPEG first, then embed
const pngImage = await pdfDoc.embedPng(pngBytes);
const jpgImage = await pdfDoc.embedJpg(jpgBytes);
```

---

## AP-009: Form Field Name Guessing

**Severity**: HIGH — throws "field not found" error

```typescript
// BROKEN — guessing field names
form.getTextField('name'); // THROWS if actual name is "form.personal.name"
```

**Why it fails**: Form field names are case-sensitive and use fully qualified dot-separated paths. The name seen in a PDF viewer may differ from the programmatic name.

```typescript
// CORRECT — enumerate fields first, then use exact names
const fields = form.getFields();
fields.forEach((field) => {
  console.log(`Type: ${field.constructor.name}, Name: "${field.getName()}"`);
});

// Use the exact name from enumeration
form.getTextField('form.personal.name');
```

---

## AP-010: Custom Font Not Applied to Form Fields

**Severity**: HIGH — non-Latin characters do not render in form fields

```typescript
// BROKEN — setting non-Latin text without font update
form.getTextField('name').setText('日本語テキスト');
const pdfBytes = await pdfDoc.save(); // Text may not render correctly
```

**Why it fails**: Form fields default to Helvetica (WinAnsi only). Setting non-Latin text without updating field appearances with a compatible font results in missing or garbled characters.

```typescript
// CORRECT — register fontkit, embed font, update appearances
import fontkit from '@pdf-lib/fontkit';

pdfDoc.registerFontkit(fontkit);
const customFont = await pdfDoc.embedFont(fontBytes);
form.getTextField('name').setText('日本語テキスト');
form.updateFieldAppearances(customFont); // MUST call this
const pdfBytes = await pdfDoc.save();
```

---

## AP-011: Duplicate Form Field Names

**Severity**: HIGH — throws error

```typescript
// BROKEN — creating two fields with the same name
form.createTextField('myField');
form.createTextField('myField'); // THROWS: duplicate name error
```

**Why it fails**: Field names must be unique within a PDF form. The PDF specification requires unique field identifiers.

```typescript
// CORRECT — use unique names
form.createTextField('myField_1');
form.createTextField('myField_2');
```

---

## AP-012: copyPages() Without addPage()

**Severity**: HIGH — pages are not added to document

```typescript
// BROKEN — copyPages alone does NOT add pages
const copiedPages = await targetDoc.copyPages(sourceDoc, [0, 1, 2]);
// Pages exist in memory but are NOT in the document
const pdfBytes = await targetDoc.save(); // Empty or missing pages
```

**Why it fails**: `copyPages()` only copies page objects into the destination document's context. You MUST explicitly add each copied page with `addPage()` or `insertPage()`.

```typescript
// CORRECT — add each copied page
const copiedPages = await targetDoc.copyPages(sourceDoc, [0, 1, 2]);
copiedPages.forEach((page) => targetDoc.addPage(page));
const pdfBytes = await targetDoc.save(); // All 3 pages present
```

---

## AP-013: Form Field Appears Empty Until Clicked

**Severity**: MEDIUM — confusing visual behavior

```typescript
// BROKEN — text set but appearance not updated
field.setText('Some value');
// Field appears empty in PDF viewer until user clicks on it
```

**Why it fails**: Setting text programmatically does not automatically regenerate the field's visual appearance stream. The value is stored but not rendered.

```typescript
// CORRECT — ensure appearances are updated
field.setText('Some value');
form.updateFieldAppearances(); // Regenerate visual appearance
// Or rely on save() default: { updateFieldAppearances: true }
```

---

## AP-014: Form Fields Lost During Page Merge

**Severity**: MEDIUM — form fields become non-functional

```typescript
// PROBLEMATIC — form field widgets copy but AcroForm definitions may not
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);
// Form fields on copiedPage may not work in targetDoc
```

**Why it happens**: `copyPages()` copies page content and widget annotations, but the AcroForm field definitions (which live at the document level) may not transfer correctly. This is a known limitation.

```typescript
// WORKAROUND — re-create form fields after merge if needed
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);
// Re-create any form fields that were on the source page
const form = targetDoc.getForm();
const newField = form.createTextField('recreated_field');
newField.addToPage(copiedPage, { x: 50, y: 700, width: 200, height: 30 });
```

---

## AP-015: Assuming pdf-lib Can Decrypt Password-Protected PDFs

**Severity**: MEDIUM — misleading API option

```typescript
// MISLEADING — ignoreEncryption does NOT decrypt
const pdfDoc = await PDFDocument.load(encryptedBytes, {
  ignoreEncryption: true,
});
// May load the document structure, but content may be unreadable
```

**Why it's misleading**: `ignoreEncryption: true` skips encryption validation checks. It does NOT decrypt content. pdf-lib has NO built-in decryption or encryption engine. For truly encrypted PDFs, use a separate tool (qpdf, mutool) to decrypt first.

---

## AP-016: setKeywords/getKeywords Asymmetry

**Severity**: LOW — unexpected return type

```typescript
// SURPRISING — setter takes array, getter returns string
pdfDoc.setKeywords(['pdf', 'report', 'finance']);
const keywords = pdfDoc.getKeywords(); // Returns "pdf,report,finance" (string), NOT array
```

**Why it's surprising**: The setter accepts `string[]` but the getter returns `string | undefined` (a comma-separated string). ALWAYS split the result if you need an array back.

```typescript
// CORRECT — handle the asymmetry
const keywordsString = pdfDoc.getKeywords(); // "pdf,report,finance"
const keywordsArray = keywordsString?.split(',') ?? [];
```
