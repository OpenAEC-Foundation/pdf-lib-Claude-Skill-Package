# pdf-lib Anti-Pattern Checklist

## Category 1: Async/Await Violations [CRITICAL]

### AP-001: Missing await on PDFDocument.create()

```typescript
// ANTI-PATTERN
const pdfDoc = PDFDocument.create() // pdfDoc is a Promise

// CORRECT
const pdfDoc = await PDFDocument.create()
```

**Impact**: Every subsequent method call fails. Nothing works.

### AP-002: Missing await on PDFDocument.load()

```typescript
// ANTI-PATTERN
const pdfDoc = PDFDocument.load(existingBytes)

// CORRECT
const pdfDoc = await PDFDocument.load(existingBytes)
```

**Impact**: Cannot access any document methods.

### AP-003: Missing await on embedFont()

```typescript
// ANTI-PATTERN
const font = pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font }) // font is a Promise, not PDFFont

// CORRECT
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
```

**Impact**: Text renders with wrong or no font. May throw runtime error.

### AP-004: Missing await on embedPng() / embedJpg()

```typescript
// ANTI-PATTERN
const image = pdfDoc.embedPng(pngBytes)
page.drawImage(image) // image is a Promise

// CORRECT
const image = await pdfDoc.embedPng(pngBytes)
```

**Impact**: drawImage receives a Promise instead of PDFImage. Throws error.

### AP-005: Missing await on copyPages()

```typescript
// ANTI-PATTERN
const pages = targetDoc.copyPages(sourceDoc, [0, 1])
targetDoc.addPage(pages[0]) // pages[0] is undefined

// CORRECT
const pages = await targetDoc.copyPages(sourceDoc, [0, 1])
```

**Impact**: addPage receives undefined. Document is empty or corrupt.

### AP-006: Missing await on save()

```typescript
// ANTI-PATTERN
const bytes = pdfDoc.save()
fs.writeFileSync('out.pdf', bytes) // Writes "[object Promise]"

// CORRECT
const bytes = await pdfDoc.save()
```

**Impact**: Output file is not a valid PDF. Contains string representation of Promise.

### AP-007: Awaiting sync methods (harmless but misleading)

```typescript
// UNNECESSARY (works but misleading)
const page = await pdfDoc.addPage()
await page.drawText('Hello')

// CORRECT (clear intent)
const page = pdfDoc.addPage()
page.drawText('Hello')
```

**Impact**: None (harmless). But suggests the developer does not understand the API.

---

## Category 2: Font Handling Violations [CRITICAL]

### AP-010: Standard fonts with non-WinAnsi characters

```typescript
// ANTI-PATTERN
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Text with Cyrillic/CJK/Arabic', { font })
// THROWS: "WinAnsi cannot encode..."
```

**Impact**: Runtime crash. The #1 reported issue in pdf-lib.

**Fix**: Use a custom TTF/OTF font with fontkit that supports the required character set.

### AP-011: Custom font bytes without fontkit registration

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(fontBytes) // CRASHES

// CORRECT
pdfDoc.registerFontkit(fontkit) // MUST come first
const font = await pdfDoc.embedFont(fontBytes)
```

**Impact**: Runtime crash when embedding custom font bytes.

### AP-012: fontkit registered AFTER embedFont call

```typescript
// ANTI-PATTERN
const font = await pdfDoc.embedFont(fontBytes) // CRASHES — fontkit not yet registered
pdfDoc.registerFontkit(fontkit) // Too late

// CORRECT — register BEFORE embed
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(fontBytes)
```

**Impact**: Runtime crash. Registration order matters.

### AP-013: Form fields with non-Latin text but no custom font

```typescript
// ANTI-PATTERN
form.getTextField('name').setText('Non-Latin text')
// Default Helvetica cannot render non-Latin characters

// CORRECT
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(unicodeFontBytes)
form.getTextField('name').setText('Non-Latin text')
form.updateFieldAppearances(customFont)
```

**Impact**: Characters appear as blank/missing in the PDF.

---

## Category 3: Image Handling Violations [CRITICAL/WARNING]

### AP-020: Unsupported image format [CRITICAL]

```typescript
// ANTI-PATTERN — NO such methods exist
const gif = await pdfDoc.embedGif(gifBytes)    // NOT A METHOD
const bmp = await pdfDoc.embedBmp(bmpBytes)    // NOT A METHOD
const svg = await pdfDoc.embedSvg(svgString)   // NOT A METHOD
const webp = await pdfDoc.embedWebP(webpBytes) // NOT A METHOD
const img = await pdfDoc.embedImage(bytes)     // NOT A METHOD
```

**Impact**: Compile error (TypeScript) or runtime error. ONLY `embedPng()` and `embedJpg()` exist.

### AP-021: Image aspect ratio distortion [WARNING]

```typescript
// ANTI-PATTERN — arbitrary width/height
page.drawImage(image, { x: 50, y: 50, width: 500, height: 100 })

// CORRECT — use scale methods
const dims = image.scaleToFit(500, 100)
page.drawImage(image, { x: 50, y: 50, width: dims.width, height: dims.height })
```

**Impact**: Image appears stretched or squashed.

---

## Category 4: Coordinate System Violations [WARNING]

### AP-030: Top-left origin assumption

```typescript
// ANTI-PATTERN — assumes y=0 is top (HTML/CSS convention)
page.drawText('Title', { x: 50, y: 20 }) // Text appears at BOTTOM

// CORRECT — PDF uses bottom-left origin
const { height } = page.getSize()
page.drawText('Title', { x: 50, y: height - 40 }) // Text appears at TOP
```

**Impact**: Content positioned at wrong location on page.

### AP-031: Hardcoded coordinates without page size reference

```typescript
// ANTI-PATTERN — assumes Letter size (612x792)
page.drawText('Title', { x: 50, y: 750 })

// CORRECT — calculate from actual page size
const { width, height } = page.getSize()
page.drawText('Title', { x: 50, y: height - 42 })
```

**Impact**: Content may be off-page or mispositioned on non-Letter pages.

---

## Category 5: Cross-Document Violations [CRITICAL]

### AP-040: Direct page addition from another document

```typescript
// ANTI-PATTERN — THROWS ERROR
const sourcePage = sourceDoc.getPage(0)
targetDoc.addPage(sourcePage)

// CORRECT
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(copiedPage)
```

**Impact**: Runtime error. Pages cannot cross document boundaries without copying.

### AP-041: copyPages without addPage

```typescript
// ANTI-PATTERN — pages copied but never added
const copiedPages = await targetDoc.copyPages(sourceDoc, [0, 1, 2])
// Missing: copiedPages.forEach(page => targetDoc.addPage(page))

// CORRECT
const copiedPages = await targetDoc.copyPages(sourceDoc, [0, 1, 2])
copiedPages.forEach(page => targetDoc.addPage(page))
```

**Impact**: Document has no pages from the source. Output is empty or missing content.

---

## Category 6: Form Field Violations [WARNING]

### AP-050: Guessing field names without enumeration

```typescript
// ANTI-PATTERN — field names are case-sensitive and hierarchical
form.getTextField('Name')      // Actual: "form.personal.name"
form.getTextField('firstName') // Actual: "form.personal.firstName"
```

**Impact**: Throws "No field with name..." error.

**Fix**: ALWAYS enumerate first:
```typescript
const fields = form.getFields()
fields.forEach(f => console.log(f.getName(), f.constructor.name))
```

### AP-051: Duplicate field names

```typescript
// ANTI-PATTERN
form.createTextField('address')
form.createTextField('address') // THROWS: duplicate name
```

**Impact**: Runtime error on second creation.

### AP-052: Not flattening final output forms

```typescript
// ANTI-PATTERN — form fields remain editable in output PDF
form.getTextField('name').setText('John Doe')
const pdfBytes = await pdfDoc.save()
// User can modify 'John Doe' in any PDF reader

// CORRECT — flatten for final output
form.getTextField('name').setText('John Doe')
form.flatten()
const pdfBytes = await pdfDoc.save()
```

**Impact**: End users can modify supposedly finalized form values.

### AP-053: Appearance streams not updated

```typescript
// ANTI-PATTERN — field appears empty until clicked
field.setText('value')
await pdfDoc.save({ updateFieldAppearances: false })

// CORRECT — either use default save or update explicitly
field.setText('value')
await pdfDoc.save() // updateFieldAppearances defaults to true
```

**Impact**: Form fields appear blank in some PDF viewers until clicked/focused.

### AP-054: Not handling XFA forms

```typescript
// ANTI-PATTERN — XFA data conflicts with AcroForm manipulation
const form = pdfDoc.getForm()
form.getTextField('name').setText('value')
// May not work if PDF uses XFA

// CORRECT — remove XFA first
const form = pdfDoc.getForm()
if (form.hasXFA()) {
  form.deleteXFA()
}
form.getTextField('name').setText('value')
```

**Impact**: Field modifications may be ignored in XFA-based PDFs.

---

## Category 7: Color Value Violations [WARNING]

### AP-060: RGB values in 0-255 range

```typescript
// ANTI-PATTERN
const red = rgb(255, 0, 0)     // Produces unexpected color
const blue = rgb(0, 0, 255)    // Produces unexpected color

// CORRECT
const red = rgb(1, 0, 0)
const blue = rgb(0, 0, 1)
```

**Impact**: Colors are completely wrong. Values above 1.0 are clamped or overflow.

### AP-061: Opacity values in 0-255 range

```typescript
// ANTI-PATTERN
page.drawText('Semi-transparent', { opacity: 128 })

// CORRECT
page.drawText('Semi-transparent', { opacity: 0.5 })
```

**Impact**: Opacity is clamped. Element may appear fully opaque.

---

## Category 8: Save Option Violations [INFO]

### AP-070: Empty document saved without addDefaultPage: false

```typescript
// ANTI-PATTERN — creates unwanted blank page
const pdfDoc = await PDFDocument.create()
// No pages added intentionally
const bytes = await pdfDoc.save() // Contains one blank page

// CORRECT — if you want zero pages
const bytes = await pdfDoc.save({ addDefaultPage: false })
```

**Impact**: Unwanted blank page in output.

### AP-071: Form appearance disabled without manual update

```typescript
// ANTI-PATTERN — fields appear empty
form.getTextField('name').setText('John')
await pdfDoc.save({ updateFieldAppearances: false })

// CORRECT — either keep default or update manually
form.getTextField('name').setText('John')
form.updateFieldAppearances()
await pdfDoc.save({ updateFieldAppearances: false })
```

**Impact**: Form fields appear blank in most PDF viewers.

---

## Category 9: Known Limitations (Not Bugs) [INFO]

### AP-080: Form fields lost during copyPages

Form field widgets (visual representations) copy with pages, but AcroForm field definitions may not transfer correctly. Workaround: re-create form fields in the target document after merge.

### AP-081: File size bloat after merge

Duplicate resources (fonts, images) from source documents are NOT deduplicated. Each copied page brings its own resources. No built-in solution — use external tools for post-processing if size is critical.

### AP-082: Keywords getter/setter asymmetry

```typescript
pdfDoc.setKeywords(['pdf', 'generation']) // Takes string[]
const keywords = pdfDoc.getKeywords()      // Returns string | undefined (comma-separated)
```

**Impact**: Developer expects array back but gets a single string.

### AP-083: copy() does not copy forms/outlines

```typescript
const copy = await pdfDoc.copy()
// AcroForms, outlines, and other metadata are NOT copied
```

**Impact**: Copied document is missing interactive elements.
