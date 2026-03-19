# pdflib-syntax-fonts -- Examples Reference

## Standard Font Examples

### Embed All Three Standard Font Families

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()
const { height } = page.getSize()

// Serif
const timesRoman = await pdfDoc.embedFont(StandardFonts.TimesRoman)
const timesBold = await pdfDoc.embedFont(StandardFonts.TimesRomanBold)
const timesItalic = await pdfDoc.embedFont(StandardFonts.TimesRomanItalic)

// Sans-serif
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica)
const helveticaBold = await pdfDoc.embedFont(StandardFonts.HelveticaBold)

// Monospace
const courier = await pdfDoc.embedFont(StandardFonts.Courier)

page.drawText('Times Roman Regular', { x: 50, y: height - 50, size: 16, font: timesRoman })
page.drawText('Times Roman Bold', { x: 50, y: height - 80, size: 16, font: timesBold })
page.drawText('Times Roman Italic', { x: 50, y: height - 110, size: 16, font: timesItalic })
page.drawText('Helvetica Regular', { x: 50, y: height - 150, size: 16, font: helvetica })
page.drawText('Helvetica Bold', { x: 50, y: height - 180, size: 16, font: helveticaBold })
page.drawText('Courier (monospace)', { x: 50, y: height - 220, size: 16, font: courier })

const pdfBytes = await pdfDoc.save()
```

### Synchronous Standard Font Embedding

```typescript
import { PDFDocument, StandardFonts } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()

// embedStandardFont() is synchronous -- no await needed
const helvetica = pdfDoc.embedStandardFont(StandardFonts.Helvetica)
const courier = pdfDoc.embedStandardFont(StandardFonts.Courier)

const page = pdfDoc.addPage()
page.drawText('No await needed for standard fonts', {
  x: 50, y: 500,
  size: 14,
  font: helvetica,
})
```

---

## Custom Font Examples

### Embed Custom TTF Font (Node.js)

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

const fontBytes = fs.readFileSync('./fonts/Roboto-Regular.ttf')
const roboto = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
page.drawText('Custom Roboto font!', {
  x: 50, y: 500,
  size: 24,
  font: roboto,
  color: rgb(0, 0.2, 0.6),
})

const pdfBytes = await pdfDoc.save()
```

### Embed Custom Font from URL (Browser)

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

const fontUrl = 'https://pdf-lib.js.org/assets/ubuntu/Ubuntu-R.ttf'
const fontBytes = await fetch(fontUrl).then(res => res.arrayBuffer())
const ubuntuFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()
page.drawText('Ubuntu font from URL', {
  x: 50, y: 500,
  size: 20,
  font: ubuntuFont,
})

const pdfBytes = await pdfDoc.save()
```

### Embed Font with Subsetting

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

const fontBytes = fs.readFileSync('./fonts/NotoSans-Regular.ttf')

// Subset: only used glyphs embedded -- smaller file size
const font = await pdfDoc.embedFont(fontBytes, { subset: true })

const page = pdfDoc.addPage()
page.drawText('Only glyphs in this text are embedded', {
  x: 50, y: 500,
  size: 14,
  font,
})

const pdfBytes = await pdfDoc.save()
// File will be significantly smaller than without subsetting
```

---

## Unicode Examples

### Cyrillic Text with Custom Font

```typescript
import { PDFDocument, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

// Use a font that contains Cyrillic glyphs (e.g., Noto Sans, Roboto, DejaVu Sans)
const fontBytes = fs.readFileSync('./fonts/NotoSans-Regular.ttf')
const font = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()

// Russian text -- ONLY works with custom font + fontkit
page.drawText('Russian text here', {
  x: 50, y: 500,
  size: 18,
  font,
  color: rgb(0, 0, 0),
})

const pdfBytes = await pdfDoc.save()
```

### CJK Text (Chinese/Japanese/Korean)

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

// MUST use a CJK-supporting font (e.g., Noto Sans CJK, Source Han Sans)
const fontBytes = fs.readFileSync('./fonts/NotoSansCJK-Regular.ttc')
const cjkFont = await pdfDoc.embedFont(fontBytes)

const page = pdfDoc.addPage()

page.drawText('CJK text here', {
  x: 50, y: 500,
  size: 18,
  font: cjkFont,
})

const pdfBytes = await pdfDoc.save()
```

### Multiple Fonts in One Document

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfDoc = await PDFDocument.create()
pdfDoc.registerFontkit(fontkit)

// Standard font for ASCII headings
const helveticaBold = await pdfDoc.embedFont(StandardFonts.HelveticaBold)

// Custom font for unicode body text
const fontBytes = fs.readFileSync('./fonts/NotoSans-Regular.ttf')
const notoSans = await pdfDoc.embedFont(fontBytes, { subset: true })

const page = pdfDoc.addPage()
const { height } = page.getSize()

// Heading in standard font
page.drawText('Document Title', {
  x: 50, y: height - 50,
  size: 28,
  font: helveticaBold,
  color: rgb(0, 0, 0),
})

// Body in custom font (supports unicode)
page.drawText('Body text with accented characters and more', {
  x: 50, y: height - 100,
  size: 14,
  font: notoSans,
  color: rgb(0.2, 0.2, 0.2),
})

const pdfBytes = await pdfDoc.save()
```

---

## Text Measurement Examples

### Center Text Horizontally

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

const title = 'Centered Title'
const fontSize = 30
const textWidth = font.widthOfTextAtSize(title, fontSize)

page.drawText(title, {
  x: (width - textWidth) / 2,
  y: height - 80,
  size: fontSize,
  font,
  color: rgb(0, 0, 0),
})
```

### Right-Align Text

```typescript
const text = 'Right-aligned text'
const fontSize = 14
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()
const rightMargin = 50

page.drawText(text, {
  x: pageWidth - textWidth - rightMargin,
  y: 500,
  size: fontSize,
  font,
})
```

### Draw Bounding Box Around Text

```typescript
const text = 'Measured text'
const fontSize = 24

const textWidth = font.widthOfTextAtSize(text, fontSize)
const textHeight = font.heightAtSize(fontSize)

// Draw text
page.drawText(text, {
  x: 100, y: 400,
  size: fontSize,
  font,
  color: rgb(0, 0, 0),
})

// Draw bounding box (for debugging/visualization)
page.drawRectangle({
  x: 100,
  y: 400,
  width: textWidth,
  height: textHeight,
  borderColor: rgb(1, 0, 0),
  borderWidth: 1,
})
```

### Calculate Font Size for Target Height

```typescript
const font = await pdfDoc.embedFont(StandardFonts.TimesRoman)

// Need text exactly 40 points tall
const targetHeight = 40
const fontSize = font.sizeAtHeight(targetHeight)

page.drawText('Sized to exact height', {
  x: 50, y: 500,
  size: fontSize,
  font,
})

// Verify
const actualHeight = font.heightAtSize(fontSize)
// actualHeight will be approximately 40
```

### Check Character Support Before Drawing

```typescript
import { PDFFont, StandardFonts } from 'pdf-lib'

function canFontRender(font: PDFFont, text: string): boolean {
  const supportedChars = new Set(font.getCharacterSet())
  for (let i = 0; i < text.length; i++) {
    if (!supportedChars.has(text.charCodeAt(i))) {
      return false
    }
  }
  return true
}

function findUnsupportedChars(font: PDFFont, text: string): string[] {
  const supportedChars = new Set(font.getCharacterSet())
  const unsupported: string[] = []
  for (let i = 0; i < text.length; i++) {
    if (!supportedChars.has(text.charCodeAt(i))) {
      unsupported.push(text[i])
    }
  }
  return unsupported
}

// Usage
const helvetica = await pdfDoc.embedFont(StandardFonts.Helvetica)

if (canFontRender(helvetica, userInput)) {
  page.drawText(userInput, { font: helvetica, x: 50, y: 500, size: 14 })
} else {
  const bad = findUnsupportedChars(helvetica, userInput)
  console.error('Unsupported characters:', bad)
  // Fall back to custom font
}
```

### Column Layout with Text Width Calculations

```typescript
const font = await pdfDoc.embedFont(StandardFonts.Courier)
const fontSize = 12

const headers = ['Name', 'Amount', 'Status']
const columnX = [50, 200, 350]

// Draw headers
headers.forEach((header, i) => {
  page.drawText(header, {
    x: columnX[i],
    y: 700,
    size: fontSize,
    font,
  })
})

// Measure widest header for underline
headers.forEach((header, i) => {
  const headerWidth = font.widthOfTextAtSize(header, fontSize)
  page.drawLine({
    start: { x: columnX[i], y: 697 },
    end: { x: columnX[i] + headerWidth, y: 697 },
    thickness: 1,
    color: rgb(0, 0, 0),
  })
})
```

---

## Form Field Font Examples

### Custom Font for Form Fields

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

const pdfBytes = fs.readFileSync('./form-template.pdf')
const pdfDoc = await PDFDocument.load(pdfBytes)
pdfDoc.registerFontkit(fontkit)

const fontBytes = fs.readFileSync('./fonts/NotoSans-Regular.ttf')
const customFont = await pdfDoc.embedFont(fontBytes)

const form = pdfDoc.getForm()

// Set field values
form.getTextField('name').setText('Unicode name value')
form.getTextField('address').setText('Unicode address value')

// CRITICAL: Update field appearances with the custom font
// Without this call, fields may display incorrectly or show blank
form.updateFieldAppearances(customFont)

const savedBytes = await pdfDoc.save()
```
