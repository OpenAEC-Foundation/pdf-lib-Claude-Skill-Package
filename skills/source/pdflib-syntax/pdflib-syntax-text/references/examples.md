# Text Drawing Examples

## Example 1: Complete Document with Text at Top of Page

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.TimesRoman)
const page = pdfDoc.addPage()  // default Letter size: 612 x 792
const { width, height } = page.getSize()

// Text near the top of the page
// height = 792, so y = 792 - 50 = 742
page.drawText('Creating PDFs in JavaScript is awesome!', {
  x: 50,
  y: height - 50,
  size: 24,
  font: font,
  color: rgb(0, 0.53, 0.71),
})

const pdfBytes = await pdfDoc.save()
```

## Example 2: Horizontal Centering

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.HelveticaBold)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

const title = 'Invoice #12345'
const titleSize = 28
const titleWidth = font.widthOfTextAtSize(title, titleSize)

page.drawText(title, {
  x: (width - titleWidth) / 2,  // horizontally centered
  y: height - 80,
  size: titleSize,
  font: font,
  color: rgb(0, 0, 0),
})

// Center a subtitle too
const subtitle = 'Thank you for your purchase'
const subtitleSize = 14
const subtitleWidth = font.widthOfTextAtSize(subtitle, subtitleSize)

page.drawText(subtitle, {
  x: (width - subtitleWidth) / 2,
  y: height - 110,
  size: subtitleSize,
  font: font,
  color: grayscale(0.4),
})

const pdfBytes = await pdfDoc.save()
```

## Example 3: Multiline Text with \n

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
const { height } = page.getSize()

const address = 'John Doe\n123 Main Street\nAnytown, CA 90210\nUnited States'

page.drawText(address, {
  x: 50,
  y: height - 100,
  size: 12,
  font: font,
  lineHeight: 18,  // ALWAYS set lineHeight with multiline text
  color: rgb(0, 0, 0),
})

const pdfBytes = await pdfDoc.save()
```

**Line spacing guideline:** Set `lineHeight` to `fontSize * 1.2` for tight spacing, `fontSize * 1.5` for comfortable reading.

## Example 4: Automatic Text Wrapping with maxWidth

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

const longText = 'Lorem ipsum dolor sit amet, consectetur adipiscing elit. ' +
  'Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. ' +
  'Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris.'

page.drawText(longText, {
  x: 50,
  y: height - 100,
  size: 12,
  font: font,
  maxWidth: width - 100,  // 50pt margin on each side
  lineHeight: 16,          // ALWAYS set with maxWidth
  color: rgb(0, 0, 0),
})

const pdfBytes = await pdfDoc.save()
```

## Example 5: Custom Word Breaks

```typescript
// Break on spaces and hyphens
page.drawText('This is a long-hyphenated-word that should break', {
  x: 50,
  y: 500,
  size: 12,
  font: font,
  maxWidth: 200,
  lineHeight: 16,
  wordBreaks: [' ', '-'],  // break on spaces AND hyphens
})
```

## Example 6: Rotated Text

```typescript
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

// Diagonal watermark
page.drawText('CONFIDENTIAL', {
  x: 100,
  y: height / 2,
  size: 60,
  font: font,
  color: rgb(1, 0, 0),
  opacity: 0.15,
  rotate: degrees(45),
})

// Vertical sidebar text
page.drawText('Side Label', {
  x: 20,
  y: 400,
  size: 12,
  font: font,
  color: rgb(0.5, 0.5, 0.5),
  rotate: degrees(90),
})

const pdfBytes = await pdfDoc.save()
```

## Example 7: Semi-Transparent Text

```typescript
page.drawText('Watermark', {
  x: 100,
  y: 400,
  size: 48,
  font: font,
  color: rgb(0.8, 0.8, 0.8),
  opacity: 0.3,  // 30% visible
})
```

## Example 8: Page Defaults for Consistent Styling

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const bodyFont = await pdfDoc.embedFont(StandardFonts.Helvetica)
const boldFont = await pdfDoc.embedFont(StandardFonts.HelveticaBold)
const page = pdfDoc.addPage()
const { height } = page.getSize()

// Set page defaults once
page.setFont(bodyFont)
page.setFontSize(11)
page.setFontColor(rgb(0.1, 0.1, 0.1))
page.setLineHeight(15)

let y = height - 60

// Title — overrides font and size
page.drawText('Monthly Report', { x: 50, y, font: boldFont, size: 22 })
y -= 30

// Body text — uses page defaults (no font/size/color needed)
page.drawText('Revenue exceeded expectations this quarter.', { x: 50, y })
y -= 15
page.drawText('Customer satisfaction scores are at an all-time high.', { x: 50, y })
y -= 15
page.drawText('New product launch scheduled for next month.', { x: 50, y })

const pdfBytes = await pdfDoc.save()
```

## Example 9: Text Measurement for Layout

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
const page = pdfDoc.addPage()

const text = 'Measured Text'
const fontSize = 24

// Measure text dimensions
const textWidth = font.widthOfTextAtSize(text, fontSize)
const textHeight = font.heightAtSize(fontSize)

// Draw text with a bounding box for visualization
page.drawText(text, {
  x: 100,
  y: 400,
  size: fontSize,
  font: font,
  color: rgb(0, 0, 0),
})

// Draw bounding rectangle around the text
page.drawRectangle({
  x: 100,
  y: 400,
  width: textWidth,
  height: textHeight,
  borderColor: rgb(1, 0, 0),
  borderWidth: 1,
})

const pdfBytes = await pdfDoc.save()
```

## Example 10: Right-Aligned Text

```typescript
const text = 'Right-aligned text'
const fontSize = 12
const textWidth = font.widthOfTextAtSize(text, fontSize)
const pageWidth = page.getWidth()
const rightMargin = 50

page.drawText(text, {
  x: pageWidth - rightMargin - textWidth,
  y: 500,
  size: fontSize,
  font: font,
  color: rgb(0, 0, 0),
})
```

## Example 11: Dynamic Font Size to Fit Height

```typescript
const targetHeight = 30  // desired text height in points
const neededSize = font.sizeAtHeight(targetHeight)

page.drawText('Fits exactly', {
  x: 50,
  y: 500,
  size: neededSize,
  font: font,
  color: rgb(0, 0, 0),
})
```

## Example 12: Adding Text to Existing PDF

```typescript
import { PDFDocument, StandardFonts, rgb, degrees } from 'pdf-lib'

const existingPdfBytes = /* Uint8Array or ArrayBuffer */
const pdfDoc = await PDFDocument.load(existingPdfBytes)
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)

const pages = pdfDoc.getPages()
const firstPage = pages[0]
const { width, height } = firstPage.getSize()

// Add text overlay to existing page
firstPage.drawText('APPROVED', {
  x: width / 2 - 100,
  y: height / 2,
  size: 50,
  font: font,
  color: rgb(0, 0.6, 0),
  opacity: 0.5,
  rotate: degrees(-30),
})

const pdfBytes = await pdfDoc.save()
```

## Example 13: Multiple Color Systems

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib'

// RGB text
page.drawText('RGB Red', {
  x: 50, y: 700, size: 14, font,
  color: rgb(1, 0, 0),
})

// CMYK text (for print workflows)
page.drawText('CMYK Cyan', {
  x: 50, y: 680, size: 14, font,
  color: cmyk(1, 0, 0, 0),
})

// Grayscale text
page.drawText('Gray text', {
  x: 50, y: 660, size: 14, font,
  color: grayscale(0.5),
})
```
