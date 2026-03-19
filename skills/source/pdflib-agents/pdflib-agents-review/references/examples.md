# pdf-lib Code Review — Before/After Examples

## Example 1: Missing Async/Await [CRITICAL]

### Before (BROKEN)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

function createPdf() {
  const pdfDoc = PDFDocument.create()
  const font = pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  page.drawText('Hello World', {
    x: 50,
    y: 700,
    size: 24,
    font: font,
    color: rgb(0, 0, 0),
  })

  const pdfBytes = pdfDoc.save()
  return pdfBytes
}
```

**Issues found:**
- `PDFDocument.create()` returns Promise — missing `await`
- `pdfDoc.embedFont()` returns Promise — missing `await`
- `pdfDoc.save()` returns Promise — missing `await`
- Function is not `async`

### After (FIXED)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

async function createPdf(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  page.drawText('Hello World', {
    x: 50,
    y: 700,
    size: 24,
    font: font,
    color: rgb(0, 0, 0),
  })

  const pdfBytes = await pdfDoc.save()
  return pdfBytes
}
```

---

## Example 2: Unicode Text with Standard Font [CRITICAL]

### Before (BROKEN)

```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib'

async function createInvoice(customerName: string): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  // customerName could contain non-Latin characters
  page.drawText(customerName, {
    x: 50,
    y: 700,
    size: 16,
    font: font,
  })

  return await pdfDoc.save()
}

// CRASHES when called with: createInvoice('Dmitri Ivanov')
```

**Issues found:**
- Standard font Helvetica cannot encode Cyrillic characters
- No fontkit registered for custom font fallback

### After (FIXED)

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

async function createInvoice(
  customerName: string,
  fontBytes: Uint8Array
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  pdfDoc.registerFontkit(fontkit)
  const font = await pdfDoc.embedFont(fontBytes)
  const page = pdfDoc.addPage()

  page.drawText(customerName, {
    x: 50,
    y: 700,
    size: 16,
    font: font,
  })

  return await pdfDoc.save()
}
```

---

## Example 3: Cross-Document Page Copy [CRITICAL]

### Before (BROKEN)

```typescript
import { PDFDocument } from 'pdf-lib'

async function mergePdfs(
  pdf1Bytes: Uint8Array,
  pdf2Bytes: Uint8Array
): Promise<Uint8Array> {
  const pdf1 = await PDFDocument.load(pdf1Bytes)
  const pdf2 = await PDFDocument.load(pdf2Bytes)
  const merged = await PDFDocument.create()

  // WRONG: adding pages directly from another document
  const page1 = pdf1.getPage(0)
  merged.addPage(page1) // THROWS ERROR

  const page2 = pdf2.getPage(0)
  merged.addPage(page2) // THROWS ERROR

  return await merged.save()
}
```

**Issues found:**
- Pages from one document CANNOT be added directly to another
- MUST use `copyPages()` to transfer pages between documents

### After (FIXED)

```typescript
import { PDFDocument } from 'pdf-lib'

async function mergePdfs(
  pdf1Bytes: Uint8Array,
  pdf2Bytes: Uint8Array
): Promise<Uint8Array> {
  const pdf1 = await PDFDocument.load(pdf1Bytes)
  const pdf2 = await PDFDocument.load(pdf2Bytes)
  const merged = await PDFDocument.create()

  const [copiedPage1] = await merged.copyPages(pdf1, [0])
  merged.addPage(copiedPage1)

  const [copiedPage2] = await merged.copyPages(pdf2, [0])
  merged.addPage(copiedPage2)

  return await merged.save()
}
```

---

## Example 4: Color Values and Coordinate System [WARNING]

### Before (BROKEN)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

async function createHeader(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  // WRONG: rgb values 0-255 instead of 0-1
  const headerColor = rgb(41, 128, 185)

  // WRONG: y=50 puts text at BOTTOM, not top
  page.drawText('Company Header', {
    x: 50,
    y: 50,
    size: 24,
    font: font,
    color: headerColor,
  })

  return await pdfDoc.save()
}
```

**Issues found:**
- `rgb(41, 128, 185)` — values exceed 1.0 range; produces incorrect color
- `y: 50` — positions text near the bottom of the page, not the top

### After (FIXED)

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

async function createHeader(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()
  const { height } = page.getSize()

  // CORRECT: rgb values divided by 255
  const headerColor = rgb(41 / 255, 128 / 255, 185 / 255)

  // CORRECT: y calculated from page height for top positioning
  page.drawText('Company Header', {
    x: 50,
    y: height - 60,
    size: 24,
    font: font,
    color: headerColor,
  })

  return await pdfDoc.save()
}
```

---

## Example 5: Form Field Handling [WARNING]

### Before (BROKEN)

```typescript
import { PDFDocument } from 'pdf-lib'

async function fillForm(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const form = pdfDoc.getForm()

  // WRONG: guessing field names without enumeration
  form.getTextField('name').setText('John Doe')
  form.getTextField('email').setText('john@example.com')
  form.getCheckBox('agree').check()

  return await pdfDoc.save()
}
```

**Issues found:**
- Field names are guessed — actual names may differ (case-sensitive, dot-notation)
- No field enumeration to discover actual names
- No error handling for missing fields

### After (FIXED)

```typescript
import { PDFDocument } from 'pdf-lib'

async function fillForm(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const form = pdfDoc.getForm()

  // ALWAYS enumerate fields first to discover actual names
  const fields = form.getFields()
  const fieldInfo = fields.map(f => ({
    name: f.getName(),
    type: f.constructor.name,
  }))
  // Use fieldInfo to verify correct names before accessing

  // Use exact field names from enumeration
  form.getTextField('form.personal.name').setText('John Doe')
  form.getTextField('form.contact.email').setText('john@example.com')
  form.getCheckBox('form.agreement.terms').check()

  // Flatten for final output
  form.flatten()

  return await pdfDoc.save()
}
```

---

## Example 6: Image Handling [WARNING + INFO]

### Before (BROKEN)

```typescript
import { PDFDocument } from 'pdf-lib'

async function addLogo(
  pdfBytes: Uint8Array,
  logoBytes: Uint8Array
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const page = pdfDoc.getPages()[0]

  // WRONG: no format-specific method — using generic approach
  const logo = await pdfDoc.embedPng(logoBytes) // Assumes PNG — may be JPG

  // WRONG: arbitrary width/height distorts aspect ratio
  page.drawImage(logo, {
    x: 50,
    y: 700,
    width: 200,
    height: 50,
  })

  return await pdfDoc.save()
}
```

**Issues found:**
- Assumes image is PNG without verification
- Setting independent width (200) and height (50) distorts the image

### After (FIXED)

```typescript
import { PDFDocument } from 'pdf-lib'

async function addLogo(
  pdfBytes: Uint8Array,
  logoBytes: Uint8Array,
  format: 'png' | 'jpg'
): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const page = pdfDoc.getPages()[0]
  const { height } = page.getSize()

  // Use correct embed method based on actual format
  const logo = format === 'png'
    ? await pdfDoc.embedPng(logoBytes)
    : await pdfDoc.embedJpg(logoBytes)

  // Preserve aspect ratio with scaleToFit
  const dims = logo.scaleToFit(200, 80)

  page.drawImage(logo, {
    x: 50,
    y: height - 80,
    width: dims.width,
    height: dims.height,
  })

  return await pdfDoc.save()
}
```

---

## Example 7: Missing Imports [CRITICAL]

### Before (BROKEN)

```typescript
import { PDFDocument } from 'pdf-lib'

async function createStyledPdf(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  // ERROR: StandardFonts not imported
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  page.drawText('Rotated Text', {
    x: 200,
    y: 400,
    size: 20,
    font: font,
    // ERROR: rgb not imported
    color: rgb(0.5, 0, 0),
    // ERROR: degrees not imported
    rotate: degrees(45),
  })

  return await pdfDoc.save()
}
```

### After (FIXED)

```typescript
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib'

async function createStyledPdf(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
  const page = pdfDoc.addPage()

  page.drawText('Rotated Text', {
    x: 200,
    y: 400,
    size: 20,
    font: font,
    color: rgb(0.5, 0, 0),
    rotate: degrees(45),
  })

  return await pdfDoc.save()
}
```

---

## Example 8: Custom Font Without fontkit [CRITICAL]

### Before (BROKEN)

```typescript
import { PDFDocument } from 'pdf-lib'
import * as fs from 'fs'

async function createWithCustomFont(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const fontBytes = fs.readFileSync('./fonts/CustomFont.ttf')

  // CRASHES: fontkit not registered
  const font = await pdfDoc.embedFont(fontBytes)
  const page = pdfDoc.addPage()

  page.drawText('Custom font text', { font, size: 16 })

  return await pdfDoc.save()
}
```

### After (FIXED)

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import * as fs from 'fs'

async function createWithCustomFont(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  pdfDoc.registerFontkit(fontkit) // MUST register before embedFont with bytes
  const fontBytes = fs.readFileSync('./fonts/CustomFont.ttf')

  const font = await pdfDoc.embedFont(fontBytes)
  const page = pdfDoc.addPage()

  page.drawText('Custom font text', { font, size: 16 })

  return await pdfDoc.save()
}
```
