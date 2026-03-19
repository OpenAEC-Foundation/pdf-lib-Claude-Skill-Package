# Drawing Examples

## Basic Shapes

### Filled Rectangle with Border

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawRectangle({
  x: 50,
  y: 400,
  width: 200,
  height: 100,
  color: rgb(0.85, 0.92, 1),       // light blue fill
  borderColor: rgb(0.2, 0.4, 0.8), // darker blue border
  borderWidth: 2,
})

const pdfBytes = await pdfDoc.save()
```

### Square with Rotation

```typescript
import { PDFDocument, rgb, degrees } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawSquare({
  x: 200,
  y: 400,
  size: 100,
  color: rgb(1, 0.8, 0),
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
  rotate: degrees(45),
})

const pdfBytes = await pdfDoc.save()
```

### Circle

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

// size is DIAMETER, x/y is CENTER
page.drawCircle({
  x: 300,
  y: 500,
  size: 120,                        // 120pt diameter = 60pt radius
  color: rgb(0.9, 0.2, 0.2),
  borderColor: rgb(0.5, 0, 0),
  borderWidth: 3,
})

const pdfBytes = await pdfDoc.save()
```

### Ellipse

```typescript
import { PDFDocument, rgb, degrees } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

// xScale/yScale are RADII, x/y is CENTER
page.drawEllipse({
  x: 300,
  y: 400,
  xScale: 150,                     // 150pt horizontal radius → 300pt wide
  yScale: 80,                      // 80pt vertical radius → 160pt tall
  color: rgb(0.3, 0.7, 0.3),
  borderColor: rgb(0, 0.4, 0),
  borderWidth: 2,
  rotate: degrees(20),
})

const pdfBytes = await pdfDoc.save()
```

## Lines

### Simple Line

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawLine({
  start: { x: 50, y: 600 },
  end: { x: 500, y: 600 },
  thickness: 2,
  color: rgb(0, 0, 0),
})

const pdfBytes = await pdfDoc.save()
```

### Dashed Line

```typescript
import { PDFDocument, rgb, LineCapStyle } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawLine({
  start: { x: 50, y: 500 },
  end: { x: 500, y: 500 },
  thickness: 1.5,
  color: rgb(0.5, 0.5, 0.5),
  dashArray: [8, 4],               // 8pt dash, 4pt gap
  lineCap: LineCapStyle.Round,
})

const pdfBytes = await pdfDoc.save()
```

### Dotted Line

```typescript
page.drawLine({
  start: { x: 50, y: 400 },
  end: { x: 500, y: 400 },
  thickness: 2,
  color: rgb(0, 0, 0),
  dashArray: [1, 6],               // 1pt dot, 6pt gap
  lineCap: LineCapStyle.Round,      // round caps make dots circular
})
```

## Colors

### RGB Colors (0.0-1.0 Range)

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib'

// Primary colors
const red    = rgb(1, 0, 0)
const green  = rgb(0, 1, 0)
const blue   = rgb(0, 0, 1)

// Common colors
const black  = rgb(0, 0, 0)
const white  = rgb(1, 1, 1)
const yellow = rgb(1, 1, 0)
const teal   = rgb(0, 0.53, 0.71)

// Converting from 0-255 to 0-1: divide by 255
// Example: CSS rgb(51, 102, 204) → pdf-lib rgb(0.2, 0.4, 0.8)
const cssBlue = rgb(51 / 255, 102 / 255, 204 / 255)
```

### CMYK Colors

```typescript
import { cmyk } from 'pdf-lib'

const cyan    = cmyk(1, 0, 0, 0)
const magenta = cmyk(0, 1, 0, 0)
const yellow  = cmyk(0, 0, 1, 0)
const black   = cmyk(0, 0, 0, 1)
const richBlack = cmyk(0.4, 0.3, 0.3, 1)
```

### Grayscale

```typescript
import { grayscale } from 'pdf-lib'

const black    = grayscale(0)       // 0.0 = black
const darkGray = grayscale(0.25)
const midGray  = grayscale(0.5)
const lightGray = grayscale(0.75)
const white    = grayscale(1)       // 1.0 = white
```

## SVG Paths

### Basic Triangle

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawSvgPath('M 0,0 L 100,0 L 50,-86.6 Z', {
  x: 200,
  y: 300,
  color: rgb(0.2, 0.6, 0.9),
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
})

const pdfBytes = await pdfDoc.save()
```

### Star Shape

```typescript
// Five-pointed star using SVG path
const star = 'M 50,0 L 61,35 L 98,35 L 68,57 L 79,91 L 50,70 L 21,91 L 32,57 L 2,35 L 39,35 Z'

page.drawSvgPath(star, {
  x: 100,
  y: 600,
  color: rgb(1, 0.84, 0),          // gold fill
  borderColor: rgb(0.8, 0.6, 0),
  borderWidth: 1.5,
  scale: 2,                        // double the size
})
```

### Curved Path with Bezier

```typescript
const curvePath = 'M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90'

page.drawSvgPath(curvePath, {
  x: 50,
  y: 400,
  borderColor: rgb(0, 0.5, 0),
  borderWidth: 3,
  scale: 0.5,
})
```

### Stroke-Only SVG Path (No Fill)

```typescript
// To draw stroke without fill, set color to undefined or omit it,
// and provide borderColor + borderWidth
page.drawSvgPath('M 0,0 L 200,0 L 200,100 L 0,100 Z', {
  x: 50,
  y: 300,
  borderColor: rgb(1, 0, 0),
  borderWidth: 2,
  // no color property → no fill
})
```

## Styled Drawing Compositions

### Dashed Border Rectangle

```typescript
import { PDFDocument, rgb, LineCapStyle } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

page.drawRectangle({
  x: 50,
  y: 600,
  width: 300,
  height: 150,
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
  borderDashArray: [10, 5, 2, 5],  // complex dash: 10pt dash, 5pt gap, 2pt dot, 5pt gap
  borderLineCap: LineCapStyle.Round,
})

const pdfBytes = await pdfDoc.save()
```

### Semi-Transparent Overlapping Shapes

```typescript
import { PDFDocument, rgb, BlendMode } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

// Red circle
page.drawCircle({
  x: 250, y: 400,
  size: 150,
  color: rgb(1, 0, 0),
  opacity: 0.5,
})

// Green circle overlapping
page.drawCircle({
  x: 320, y: 400,
  size: 150,
  color: rgb(0, 1, 0),
  opacity: 0.5,
})

// Blue circle overlapping
page.drawCircle({
  x: 285, y: 340,
  size: 150,
  color: rgb(0, 0, 1),
  opacity: 0.5,
})

const pdfBytes = await pdfDoc.save()
```

### Horizontal Rule / Separator Line

```typescript
const { width } = page.getSize()

// Full-width horizontal rule with padding
const padding = 50
page.drawLine({
  start: { x: padding, y: 400 },
  end: { x: width - padding, y: 400 },
  thickness: 0.5,
  color: grayscale(0.7),
})
```

### Drawing at Page Position (Using moveTo)

```typescript
const page = pdfDoc.addPage()
const { height } = page.getSize()

// Position cursor, then draw SVG path at current position
page.moveTo(100, height - 25)
page.moveDown(25)
page.drawSvgPath('M 0,0 L 100,0 L 50,-80 Z')

page.moveDown(200)
page.drawSvgPath('M 0,0 L 100,0 L 50,-80 Z', {
  color: rgb(1, 0, 0),
})
```

### Complete Drawing Composition

```typescript
import { PDFDocument, rgb, grayscale, degrees, LineCapStyle } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()
const { width, height } = page.getSize()

// Background
page.drawRectangle({
  x: 0, y: 0,
  width, height,
  color: grayscale(0.95),
})

// Header bar
page.drawRectangle({
  x: 0, y: height - 60,
  width, height: 60,
  color: rgb(0.2, 0.4, 0.8),
})

// Separator line
page.drawLine({
  start: { x: 50, y: height - 80 },
  end: { x: width - 50, y: height - 80 },
  thickness: 1,
  color: grayscale(0.7),
})

// Content area border
page.drawRectangle({
  x: 40, y: 40,
  width: width - 80,
  height: height - 140,
  borderColor: grayscale(0.8),
  borderWidth: 0.5,
  borderDashArray: [4, 2],
})

// Decorative circle
page.drawCircle({
  x: width - 80, y: height - 30,
  size: 30,
  color: rgb(1, 1, 1),
  opacity: 0.3,
})

const pdfBytes = await pdfDoc.save()
```
