# pdf-lib Forms, Drawing & Document Manipulation — Deep Research

> **Sources**: pdf-lib.js.org (official site + API docs), github.com/Hopding/pdf-lib (README + issues)
> **Date**: 2026-03-19
> **Scope**: Form fields, form filling, form flattening, drawing shapes, colors, document merging/splitting, encrypted PDFs, file attachments, common errors

---

## 1. Form Fields — Accessing and Reading

### Getting the Form Object

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.load(existingPdfBytes)
const form = pdfDoc.getForm()  // Returns: PDFForm
```

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfform

### PDFForm — Complete API

```typescript
// Getters — retrieve existing fields by name (throws if not found)
form.getFields(): PDFField[]
form.getField(name: string): PDFField
form.getFieldMaybe(name: string): PDFField | undefined
form.getTextField(name: string): PDFTextField
form.getCheckBox(name: string): PDFCheckBox
form.getRadioGroup(name: string): PDFRadioGroup
form.getDropdown(name: string): PDFDropdown
form.getOptionList(name: string): PDFOptionList
form.getButton(name: string): PDFButton
form.getSignature(name: string): PDFSignature
form.getDefaultFont(): PDFFont

// Creators — create new fields
form.createTextField(name: string): PDFTextField
form.createCheckBox(name: string): PDFCheckBox
form.createRadioGroup(name: string): PDFRadioGroup
form.createDropdown(name: string): PDFDropdown
form.createOptionList(name: string): PDFOptionList
form.createButton(name: string): PDFButton

// Field management
form.removeField(field: PDFField): void
form.markFieldAsDirty(fieldRef: PDFRef): void
form.markFieldAsClean(fieldRef: PDFRef): void
form.fieldIsDirty(fieldRef: PDFRef): boolean

// Form operations
form.flatten(options?: FlattenOptions): void  // Default: { updateFieldAppearances: true }
form.updateFieldAppearances(font?: PDFFont): void

// XFA handling
form.hasXFA(): boolean
form.deleteXFA(): void
```

### Getting All Field Names

```typescript
const fields = form.getFields()
const fieldNames = fields.map(f => f.getName())
// Returns fully qualified names, e.g. "form.section.fieldName"
```

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfform

---

## 2. Field Types — Complete API Reference

### PDFTextField

**Source**: https://pdf-lib.js.org/docs/api/classes/pdftextfield

```typescript
const field = form.getTextField('fieldName')

// Core text operations
field.setText(text: string | undefined): void
field.getText(): string | undefined
field.setFontSize(fontSize: number): void

// Layout & display
field.setAlignment(alignment: TextAlignment): void  // TextAlignment.Left | Center | Right
field.enableMultiline(): void
field.disableMultiline(): void
field.enableScrolling(): void
field.disableScrolling(): void

// Input constraints
field.setMaxLength(maxLength?: number | undefined): void
field.getMaxLength(): number | undefined
field.removeMaxLength(): void

// Special behaviors
field.enableCombing(): void    // Distributes chars equally across cells
field.disableCombing(): void
field.enablePassword(): void   // Masks input
field.disablePassword(): void
field.enableFileSelection(): void
field.disableFileSelection(): void

// Appearance
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(font: PDFFont, provider?: AppearanceProvider): void
field.defaultUpdateAppearances(font: PDFFont): void
field.setImage(image: PDFImage): void

// Field properties
field.enableReadOnly(): void
field.disableReadOnly(): void
field.enableRequired(): void
field.disableRequired(): void
field.enableExporting(): void
field.disableExporting(): void
field.enableRichFormatting(): void
field.disableRichFormatting(): void
field.enableSpellChecking(): void
field.disableSpellChecking(): void

// Query methods
field.getName(): string
field.isMultiline(): boolean
field.isPassword(): boolean
field.isCombed(): boolean
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isFileSelector(): boolean
field.isExported(): boolean
field.isScrollable(): boolean
field.isSpellChecked(): boolean
field.isRichFormatted(): boolean
field.needsAppearancesUpdate(): boolean
field.getAlignment(): TextAlignment
```

### PDFCheckBox

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfcheckbox

```typescript
const field = form.getCheckBox('fieldName')

// Core
field.check(): void
field.uncheck(): void
field.isChecked(): boolean

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(provider?: AppearanceProviderFor<PDFCheckBox>): void
field.defaultUpdateAppearances(): void

// Properties (inherited)
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
field.getName(): string
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isExported(): boolean
field.needsAppearancesUpdate(): boolean
```

### PDFRadioGroup

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfradiogroup

```typescript
const field = form.getRadioGroup('groupName')

// Core
field.select(option: string): void
field.getSelected(): string | undefined
field.getOptions(): string[]
field.clear(): void

// Widget management
field.addOptionToPage(option: string, page: PDFPage, options?: FieldAppearanceOptions): void

// Mutual exclusion
field.enableMutualExclusion(): void   // Multiple buttons with same value selected together
field.disableMutualExclusion(): void  // Only one button per selection
field.isMutuallyExclusive(): boolean

// Off-toggle
field.enableOffToggling(): void   // Allow deselect by clicking selected button
field.disableOffToggling(): void
field.isOffToggleable(): boolean

// Appearance
field.updateAppearances(provider?: AppearanceProviderFor<PDFRadioGroup>): void
field.defaultUpdateAppearances(): void
field.needsAppearancesUpdate(): boolean

// Properties (inherited)
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
field.getName(): string
```

### PDFDropdown

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfdropdown

```typescript
const field = form.getDropdown('fieldName')

// Core
field.select(options: string | string[], merge?: boolean): void  // merge preserves existing selections
field.getSelected(): string[]
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void  // Replaces entire list

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void

// Configuration
field.enableMultiselect(): void / field.disableMultiselect(): void
field.enableEditing(): void / field.disableEditing(): void      // Allow keyboard entry
field.enableSorting(): void / field.disableSorting(): void      // Alphabetical ordering
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.enableSpellChecking(): void / field.disableSpellChecking(): void

// Appearance & properties (inherited pattern same as above)
```

### PDFOptionList

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfoptionlist

```typescript
const field = form.getOptionList('fieldName')

// Core
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.clear(): void
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void

// Display
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void

// Configuration
field.enableMultiselect(): void / field.disableMultiselect(): void
field.isMultiselect(): boolean
field.enableSorting(): void / field.disableSorting(): void
field.isSorted(): boolean
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.isSelectOnClick(): boolean

// Appearance
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFOptionList>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean
```

### PDFButton

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfbutton

```typescript
const field = form.getButton('fieldName')

// Core
field.setImage(image: PDFImage, alignment?: ImageAlignment): void
field.setFontSize(fontSize: number): void

// Display
field.addToPage(text: string, page: PDFPage, options?: FieldAppearanceOptions): void
// NOTE: addToPage takes a text label as first argument (unlike other field types)

// Appearance
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFButton>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean

// Properties (inherited)
field.getName(): string
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
```

### PDFSignature

```typescript
const field = form.getSignature('fieldName')
// Primarily a metadata container for digital signature fields
// No create method available — signatures are read from existing PDFs
```

---

## 3. Form Filling — Complete Workflow

### Fill Existing Form

**Source**: github.com/Hopding/pdf-lib README

```typescript
import { PDFDocument } from 'pdf-lib'

const formPdfBytes = /* Uint8Array or ArrayBuffer */
const pdfDoc = await PDFDocument.load(formPdfBytes)
const form = pdfDoc.getForm()

// Text fields
form.getTextField('CharacterName 2').setText('Mario')
form.getTextField('Age').setText('24 years')

// Checkboxes
form.getCheckBox('Check Box3').check()

// Radio groups
form.getRadioGroup('Group2').select('Choice1')

// Dropdowns
form.getDropdown('Dropdown7').select('Infinity')

// Buttons (with images)
const marioImage = await pdfDoc.embedPng(marioImageBytes)
form.getButton('CHARACTER IMAGE').setImage(marioImage)

const pdfBytes = await pdfDoc.save()
```

### Create New Form

**Source**: github.com/Hopding/pdf-lib README

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage([550, 750])
const form = pdfDoc.getForm()

// Text field
page.drawText('Enter your favorite superhero:', { x: 50, y: 700, size: 20 })
const superheroField = form.createTextField('favorite.superhero')
superheroField.setText('One Punch Man')
superheroField.addToPage(page, { x: 55, y: 640 })

// Radio group
page.drawText('Select your favorite rocket:', { x: 50, y: 600, size: 20 })
page.drawText('Falcon Heavy', { x: 120, y: 560, size: 18 })
page.drawText('Saturn IV', { x: 120, y: 500, size: 18 })
const rocketField = form.createRadioGroup('favorite.rocket')
rocketField.addOptionToPage('Falcon Heavy', page, { x: 55, y: 540 })
rocketField.addOptionToPage('Saturn IV', page, { x: 55, y: 480 })
rocketField.select('Saturn IV')

// Checkbox
page.drawText('Select your favorite gundams:', { x: 50, y: 440, size: 20 })
const exiaField = form.createCheckBox('gundam.exia')
exiaField.addToPage(page, { x: 55, y: 380 })
exiaField.check()

// Dropdown
page.drawText('Select your favorite planet*:', { x: 50, y: 280, size: 20 })
const planetsField = form.createDropdown('favorite.planet')
planetsField.addOptions(['Venus', 'Earth', 'Mars', 'Pluto'])
planetsField.select('Pluto')
planetsField.addToPage(page, { x: 55, y: 220 })

// Option list
const personField = form.createOptionList('favorite.person')
personField.addOptions(['Julius Caesar', 'Ada Lovelace', 'Cleopatra'])
personField.select('Ada Lovelace')
personField.addToPage(page, { x: 55, y: 70 })

const pdfBytes = await pdfDoc.save()
```

### Unicode / Custom Font Support for Forms

**Source**: github.com/Hopding/pdf-lib README — important note

> The default font used to display text in buttons, dropdowns, option lists, and text fields is the standard Helvetica font, which only supports Latin characters. For non-Latin text, register `@pdf-lib/fontkit` and call `form.updateFieldAppearances(ubuntuFont)` after filling fields.

```typescript
import fontkit from '@pdf-lib/fontkit'

pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes) // TTF or OTF bytes

// Fill fields first
form.getTextField('name').setText('日本語テキスト')

// Then update appearances with the custom font
form.updateFieldAppearances(customFont)

const pdfBytes = await pdfDoc.save()
```

---

## 4. Form Flattening

### Basic Flattening

**Source**: github.com/Hopding/pdf-lib README

```typescript
import { PDFDocument } from 'pdf-lib'

const formPdfBytes = /* existing form PDF */
const pdfDoc = await PDFDocument.load(formPdfBytes)
const form = pdfDoc.getForm()

// Fill fields BEFORE flattening
form.getTextField('Text1').setText('Some Text')
form.getRadioGroup('Group2').select('Choice1')
form.getCheckBox('Check Box3').check()
form.getDropdown('Dropdown7').select('Infinity')

// Flatten — converts all fields to static content
form.flatten()
// Default options: { updateFieldAppearances: true }

const pdfBytes = await pdfDoc.save()
```

### Flatten Options

```typescript
form.flatten(options?: {
  updateFieldAppearances?: boolean  // Default: true
})
```

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfform

### Selective Flattening (from GitHub PRs)

**Source**: GitHub Issues #1758, #1367

There are open PRs for selective flattening (flatten only specific fields). As of the current API, `form.flatten()` flattens ALL fields. Workaround: remove fields you do NOT want to flatten before calling `form.flatten()`, or use `form.removeField(field)` on specific fields.

### What Flattening Does
- Converts interactive form fields into static, non-editable content
- Field values become part of the page content stream
- Users can no longer modify values in the PDF
- Reduces file complexity and improves compatibility

### When to Flatten
- ALWAYS flatten when generating final output PDFs (invoices, certificates, reports)
- ALWAYS flatten before distributing PDFs to prevent modification
- NEVER flatten if the form needs to be filled again later
- NEVER flatten if you need to programmatically read field values later

---

## 5. Drawing Shapes

### PDFPage Drawing Methods — Complete Signatures

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfpage

#### drawRectangle()

```typescript
page.drawRectangle(options?: {
  x?: number,           // X coordinate (lower-left origin)
  y?: number,           // Y coordinate (lower-left origin)
  width?: number,       // Rectangle width in points
  height?: number,      // Rectangle height in points
  color?: Color,        // Fill color (rgb/cmyk/grayscale)
  borderColor?: Color,  // Stroke color
  borderWidth?: number, // Stroke thickness in points
  opacity?: number,     // Fill opacity (0.0 to 1.0)
  borderOpacity?: number, // Stroke opacity (0.0 to 1.0)
  rotate?: Rotation,    // Rotation via degrees() or radians()
}): void
```

#### drawSquare()

```typescript
page.drawSquare(options?: {
  x?: number,
  y?: number,
  size?: number,         // Side length (unlike rectangle, single dimension)
  color?: Color,
  borderColor?: Color,
  borderWidth?: number,
  opacity?: number,
  borderOpacity?: number,
  rotate?: Rotation,
}): void
```

#### drawCircle()

```typescript
page.drawCircle(options?: {
  x?: number,            // Center X
  y?: number,            // Center Y
  size?: number,         // Diameter
  color?: Color,
  borderColor?: Color,
  borderWidth?: number,
  opacity?: number,
  borderOpacity?: number,
}): void
```

#### drawEllipse()

```typescript
page.drawEllipse(options?: {
  x?: number,            // Center X
  y?: number,            // Center Y
  xScale?: number,       // Horizontal radius
  yScale?: number,       // Vertical radius
  color?: Color,
  borderColor?: Color,
  borderWidth?: number,
  opacity?: number,
  borderOpacity?: number,
}): void
```

#### drawLine()

```typescript
page.drawLine(options: {
  start: { x: number, y: number },
  end: { x: number, y: number },
  thickness?: number,    // Line thickness in points
  color?: Color,
  opacity?: number,
}): void
```

#### drawSvgPath()

```typescript
page.drawSvgPath(path: string, options?: {
  x?: number,
  y?: number,
  color?: Color,          // Fill color
  borderColor?: Color,    // Stroke color
  borderWidth?: number,
  opacity?: number,
  borderOpacity?: number,
  scale?: number,         // Scale factor
}): void
```

**SVG Path Example** (from README):

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

const svgPath = 'M 0,20 L 100,160 Q 130,200 150,120 C 190,-40 200,200 300,150 L 400,90'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage()

// Draw at current position (uses page.moveTo())
page.moveTo(100, page.getHeight() - 5)
page.moveDown(25)
page.drawSvgPath(svgPath)

// Draw with stroke only
page.moveDown(200)
page.drawSvgPath(svgPath, { borderColor: rgb(0, 1, 0), borderWidth: 5 })

// Draw with fill only
page.moveDown(200)
page.drawSvgPath(svgPath, { color: rgb(1, 0, 0) })

const pdfBytes = await pdfDoc.save()
```

#### drawText()

```typescript
page.drawText(text: string, options?: {
  x?: number,
  y?: number,
  size?: number,          // Font size in points
  font?: PDFFont,
  color?: Color,
  opacity?: number,
  lineHeight?: number,
  maxWidth?: number,      // Text wrapping width
  rotate?: Rotation,
}): void
```

#### drawImage()

```typescript
page.drawImage(image: PDFImage, options?: {
  x?: number,
  y?: number,
  width?: number,
  height?: number,
  rotate?: Rotation,
  opacity?: number,
}): void
```

#### drawPage() (embedded page)

```typescript
page.drawPage(embeddedPage: PDFEmbeddedPage, options?: {
  x?: number,
  y?: number,
  xScale?: number,
  yScale?: number,
  width?: number,
  height?: number,
  rotate?: Rotation,
  opacity?: number,
}): void
```

---

## 6. Colors

**Source**: https://pdf-lib.js.org

### Color Constructors

```typescript
import { rgb, cmyk, grayscale } from 'pdf-lib'

// RGB — values 0.0 to 1.0 (NOT 0-255)
const red = rgb(1, 0, 0)
const blue = rgb(0, 0, 1)
const teal = rgb(0, 0.53, 0.71)
const customRed = rgb(0.95, 0.1, 0.1)

// CMYK — values 0.0 to 1.0
const cyan = cmyk(1, 0, 0, 0)
const magenta = cmyk(0, 1, 0, 0)
const yellow = cmyk(0, 0, 1, 0)
const black = cmyk(0, 0, 0, 1)

// Grayscale — 0.0 (black) to 1.0 (white)
// NOTE: In grayscale, 0 = black, 1 = white (opposite of what some expect)
const gBlack = grayscale(0)
const gWhite = grayscale(1)
const gMidGray = grayscale(0.5)
```

### Color Type

```typescript
type Color = RGB | CMYK | Grayscale

// All drawing methods accept Color type for color, borderColor, fontColor
```

### Usage in Drawing

```typescript
page.drawRectangle({
  x: 50, y: 50,
  width: 200, height: 100,
  color: rgb(0.2, 0.4, 0.8),          // Blue fill
  borderColor: rgb(0, 0, 0),          // Black border
})

page.drawText('Colored text', {
  x: 50, y: 300,
  color: rgb(0.95, 0.1, 0.1),         // Red text
})
```

---

## 7. Drawing Options — Reference Table

| Option | Applies To | Type | Description |
|--------|-----------|------|-------------|
| `x` | All shapes, text, image | `number` | X coordinate (lower-left origin) |
| `y` | All shapes, text, image | `number` | Y coordinate (lower-left origin) |
| `width` | Rectangle, image, page | `number` | Width in points |
| `height` | Rectangle, image, page | `number` | Height in points |
| `size` | Circle, square, text | `number` | Diameter (circle/square) or font size (text) |
| `color` | All shapes, text | `Color` | Fill color |
| `borderColor` | Rectangle, circle, ellipse, square, SVG | `Color` | Stroke color |
| `borderWidth` | Rectangle, circle, ellipse, square, SVG | `number` | Stroke thickness |
| `opacity` | All | `number` | Fill opacity (0.0-1.0) |
| `borderOpacity` | Rectangle, circle, ellipse, square, SVG | `number` | Stroke opacity (0.0-1.0) |
| `rotate` | Rectangle, square, text, image, page | `Rotation` | `degrees(45)` or `radians(Math.PI/4)` |
| `scale` | SVG path | `number` | Scale factor |
| `thickness` | Line | `number` | Line width in points |
| `start` / `end` | Line | `{ x, y }` | Start and end points |
| `xScale` / `yScale` | Ellipse, embedded page | `number` | Horizontal/vertical scale |
| `font` | Text | `PDFFont` | Font to use |
| `lineHeight` | Text | `number` | Line spacing |
| `maxWidth` | Text | `number` | Text wrapping width |

### Rotation Helper

```typescript
import { degrees, radians } from 'pdf-lib'

page.drawRectangle({
  x: 100, y: 100,
  width: 200, height: 50,
  rotate: degrees(45),       // 45 degree rotation
  // or: rotate: radians(Math.PI / 4)
})
```

---

## 8. Document Merging — copyPages() Workflow

### Basic Merge

**Source**: github.com/Hopding/pdf-lib README

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()

const firstDonorPdfBytes = /* Uint8Array or ArrayBuffer */
const secondDonorPdfBytes = /* Uint8Array or ArrayBuffer */

const firstDonorPdfDoc = await PDFDocument.load(firstDonorPdfBytes)
const secondDonorPdfDoc = await PDFDocument.load(secondDonorPdfBytes)

// copyPages returns an array of PDFPage objects
const [firstDonorPage] = await pdfDoc.copyPages(firstDonorPdfDoc, [0])
const [secondDonorPage] = await pdfDoc.copyPages(secondDonorPdfDoc, [742])

// Add pages in desired order
pdfDoc.addPage(firstDonorPage)
pdfDoc.insertPage(0, secondDonorPage)  // Insert at specific position

const pdfBytes = await pdfDoc.save()
```

### copyPages() Signature

```typescript
pdfDoc.copyPages(
  srcDoc: PDFDocument,    // Source document to copy FROM
  indices: number[]       // 0-based page indices to copy
): Promise<PDFPage[]>     // Returns array of copied pages
```

### Merge Multiple Documents (Complete Pattern)

```typescript
import { PDFDocument } from 'pdf-lib'

async function mergeDocuments(pdfByteArrays: Uint8Array[]): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create()

  for (const pdfBytes of pdfByteArrays) {
    const sourcePdf = await PDFDocument.load(pdfBytes)
    const pageCount = sourcePdf.getPageCount()
    const indices = Array.from({ length: pageCount }, (_, i) => i)
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices)
    copiedPages.forEach(page => mergedPdf.addPage(page))
  }

  return await mergedPdf.save()
}
```

### Page Insertion Methods

```typescript
// Append page at end
pdfDoc.addPage(page: PDFPage): PDFPage

// Add blank page (optional dimensions)
pdfDoc.addPage([width, height]?: [number, number]): PDFPage

// Insert at specific index
pdfDoc.insertPage(index: number, page: PDFPage): PDFPage

// Insert blank page at index
pdfDoc.insertPage(index: number, [width, height]?: [number, number]): PDFPage

// Remove page at index
pdfDoc.removePage(index: number): void
```

### Critical Rule: ALWAYS Use copyPages() for Cross-Document Operations

```typescript
// CORRECT — copy pages first, then add
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(copiedPage)

// WRONG — adding page directly from another document
// This will throw an error:
// "You tried to add a page that belongs to another document"
const sourcePage = sourceDoc.getPage(0)
targetDoc.addPage(sourcePage)  // ERROR!
```

---

## 9. Document Splitting

### Extract Single Page

```typescript
import { PDFDocument } from 'pdf-lib'

async function extractPage(pdfBytes: Uint8Array, pageIndex: number): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const newPdf = await PDFDocument.create()

  const [extractedPage] = await newPdf.copyPages(sourcePdf, [pageIndex])
  newPdf.addPage(extractedPage)

  return await newPdf.save()
}
```

### Extract Page Range

```typescript
async function extractPageRange(
  pdfBytes: Uint8Array,
  startPage: number,  // 0-indexed
  endPage: number     // 0-indexed, inclusive
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const newPdf = await PDFDocument.create()

  const indices = Array.from(
    { length: endPage - startPage + 1 },
    (_, i) => startPage + i
  )
  const pages = await newPdf.copyPages(sourcePdf, indices)
  pages.forEach(page => newPdf.addPage(page))

  return await newPdf.save()
}
```

### Split Into Individual Pages

```typescript
async function splitAllPages(pdfBytes: Uint8Array): Promise<Uint8Array[]> {
  const sourcePdf = await PDFDocument.load(pdfBytes)
  const pageCount = sourcePdf.getPageCount()
  const results: Uint8Array[] = []

  for (let i = 0; i < pageCount; i++) {
    const newPdf = await PDFDocument.create()
    const [page] = await newPdf.copyPages(sourcePdf, [i])
    newPdf.addPage(page)
    results.push(await newPdf.save())
  }

  return results
}
```

---

## 10. Encrypted PDFs

### Load Options

**Source**: https://pdf-lib.js.org/docs/api/classes/pdfdocument

```typescript
const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,     // Bypass encryption entirely
  // password: 'user-password', // Not a standard option — see note below
})
```

### PDFDocument Properties

```typescript
pdfDoc.isEncrypted  // boolean — check if document is encrypted
```

### Key Notes on Encryption
- `ignoreEncryption: true` allows loading PDFs that have encryption markers but may not have actual content encryption
- pdf-lib does NOT support decrypting PDFs with passwords (no built-in decryption engine)
- pdf-lib does NOT support encrypting/password-protecting output PDFs
- For truly encrypted PDFs, you may need to pre-process with another tool (e.g., qpdf, mutool)

---

## 11. File Attachments

### Attach Files to PDF

**Source**: github.com/Hopding/pdf-lib README

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()

// Attach a JPEG image
const jpgAttachmentBytes = /* Uint8Array or ArrayBuffer */
await pdfDoc.attach(jpgAttachmentBytes, 'cat_riding_unicorn.jpg', {
  mimeType: 'image/jpeg',
  description: 'Cool cat riding a unicorn!',
  creationDate: new Date('2019/12/01'),
  modificationDate: new Date('2020/04/19'),
})

// Attach a PDF
const pdfAttachmentBytes = /* Uint8Array or ArrayBuffer */
await pdfDoc.attach(pdfAttachmentBytes, 'us_constitution.pdf', {
  mimeType: 'application/pdf',
  description: 'Constitution of the United States',
  creationDate: new Date('1787/09/17'),
  modificationDate: new Date('1992/05/07'),
})

const page = pdfDoc.addPage()
page.drawText('This PDF has two attachments', { x: 135, y: 415 })

const pdfBytes = await pdfDoc.save()
```

### attach() Signature

```typescript
pdfDoc.attach(
  attachment: string | Uint8Array | ArrayBuffer,  // File bytes
  name: string,                                     // Filename shown in viewer
  options?: {
    mimeType?: string,
    description?: string,
    creationDate?: Date,
    modificationDate?: Date,
  }
): Promise<void>
```

### Viewer Compatibility
- Attachments are viewable in: Adobe Reader, Foxit Reader, Firefox
- NOT all PDF viewers support viewing attachments
- Attachments do NOT render on pages — they appear in the viewer's attachment panel

---

## 12. PDFPage — Additional Methods

### Page Dimensions and Positioning

```typescript
// Get dimensions
page.getSize(): { width: number; height: number }
page.getWidth(): number
page.getHeight(): number

// Set dimensions
page.setSize(width: number, height: number): void
page.setWidth(width: number): void
page.setHeight(height: number): void

// Position cursor (affects subsequent draw calls without explicit x,y)
page.moveTo(x: number, y: number): void
page.moveUp(yIncrease: number): void
page.moveDown(yDecrease: number): void
page.moveLeft(xDecrease: number): void
page.moveRight(xIncrease: number): void
page.getPosition(): { x: number; y: number }
page.getX(): number
page.getY(): number
page.resetPosition(): void

// Set default text properties
page.setFont(font: PDFFont): void
page.setFontSize(fontSize: number): void
page.setFontColor(fontColor: Color): void
page.setLineHeight(lineHeight: number): void
```

### Page Boxes

```typescript
// Media box (physical page size)
page.setMediaBox(x: number, y: number, width: number, height: number): void
page.getMediaBox(): { x: number; y: number; width: number; height: number }

// Crop box (visible area)
page.setCropBox(x: number, y: number, width: number, height: number): void
page.getCropBox(): { x: number; y: number; width: number; height: number }

// Bleed box (printing bleed area)
page.setBleedBox(x: number, y: number, width: number, height: number): void
page.getBleedBox(): { x: number; y: number; width: number; height: number }

// Trim box (intended final size)
page.setTrimBox(x: number, y: number, width: number, height: number): void
page.getTrimBox(): { x: number; y: number; width: number; height: number }

// Art box (extent of meaningful content)
page.setArtBox(x: number, y: number, width: number, height: number): void
page.getArtBox(): { x: number; y: number; width: number; height: number }
```

### Page Transformations

```typescript
page.scale(x: number, y: number): void            // Scale page + content + annotations
page.scaleContent(x: number, y: number): void      // Scale content only
page.scaleAnnotations(x: number, y: number): void  // Scale annotations only
page.translateContent(x: number, y: number): void  // Move content
page.setRotation(angle: Rotation): void             // Set page rotation
page.getRotation(): Rotation                        // Get current rotation
```

---

## 13. PDFDocument — Additional Methods

### Embedding Content

```typescript
// Images
pdfDoc.embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
pdfDoc.embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>

// Fonts
pdfDoc.embedFont(font: StandardFonts | string | Uint8Array | ArrayBuffer, options?: EmbedFontOptions): Promise<PDFFont>
pdfDoc.embedStandardFont(font: StandardFonts, customName?: string): PDFFont

// Pages as images (embed pages from other PDFs)
pdfDoc.embedPdf(pdf: string | Uint8Array | ArrayBuffer | PDFDocument, indices: number[]): Promise<PDFEmbeddedPage[]>
pdfDoc.embedPage(page: PDFPage, boundingBox?: PageBoundingBox, transformationMatrix?: TransformationMatrix): Promise<PDFEmbeddedPage>
pdfDoc.embedPages(pages: PDFPage[], boundingBoxes?: PageBoundingBox[]): Promise<PDFEmbeddedPage[]>

// Font registration for custom fonts
pdfDoc.registerFontkit(fontkit: Fontkit): void
```

### Save Options

```typescript
pdfDoc.save(options?: {
  useObjectStreams?: boolean,     // Compress objects (default: true)
  addDefaultPage?: boolean,      // Add blank page if empty (default: true)
  objectsPerTick?: number,       // Objects to serialize per tick (default: 50)
  updateFieldAppearances?: boolean, // Update form appearances before save (default: true)
}): Promise<Uint8Array>

pdfDoc.saveAsBase64(options?: Base64SaveOptions): Promise<string>
```

### Document Copy

```typescript
pdfDoc.copy(): Promise<PDFDocument>  // Deep copy entire document
```

### JavaScript Actions

```typescript
pdfDoc.addJavaScript(name: string, script: string): void
```

---

## 14. Common Errors and Anti-Patterns

### From GitHub Issues Analysis

#### Error: Cross-Document Page Addition
**Issue pattern**: Trying to add a page from one document directly to another without `copyPages()`.
```typescript
// ANTI-PATTERN — will throw error
const sourcePage = sourceDoc.getPage(0)
targetDoc.addPage(sourcePage)

// CORRECT PATTERN
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(copiedPage)
```

#### Error: Form Field Not Found
**Issues**: #1670, #1620, #1585
**Cause**: Field names are case-sensitive and use fully qualified names (dot-separated hierarchy).
```typescript
// ANTI-PATTERN — guessing field names
form.getTextField('name')  // May throw if actual name is "form.personal.name"

// CORRECT PATTERN — enumerate fields first
const fields = form.getFields()
fields.forEach(field => {
  console.log(`Type: ${field.constructor.name}, Name: "${field.getName()}"`)
})
// Then use exact names from enumeration
```

#### Error: Font Not Applied to Form Fields
**Issues**: #1750, #1538, #1378
**Cause**: Default Helvetica only supports Latin characters. Custom fonts need explicit appearance update.
```typescript
// ANTI-PATTERN — setting text without font consideration
form.getTextField('name').setText('日本語')
// Result: Characters may not render

// CORRECT PATTERN — register fontkit and update appearances
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)
form.getTextField('name').setText('日本語')
form.updateFieldAppearances(customFont)
```

#### Error: Duplicate Field Names
**Issue**: #1652
**Cause**: Creating two fields with the same name generates an error.
```typescript
// ANTI-PATTERN
form.createTextField('myField')
form.createTextField('myField')  // ERROR: duplicate name

// CORRECT PATTERN — use unique names
form.createTextField('myField_1')
form.createTextField('myField_2')
```

#### Error: Form Fields Lost During copyPages()
**Issues**: #1205, #1587
**Cause**: Form field data (AcroForm entries) may not transfer correctly when copying pages between documents.
```typescript
// KNOWN LIMITATION
// When copying pages that contain form fields, the field widgets
// (visual representations) are copied but the AcroForm field
// definitions may not transfer. This can result in non-functional
// form fields in the merged document.

// WORKAROUND — re-create form fields after merge if needed
```

#### Error: Blank Pages in Merged PDFs
**Issues**: #1579, #1767
**Cause**: Content streams may reference resources that don't copy correctly. Particularly affects PDFs viewed in Adobe Acrobat Reader (may work in browsers).
```typescript
// No guaranteed workaround — this is a known limitation
// Test merged output in multiple viewers (Acrobat, browser, Foxit)
```

#### Error: File Size Bloat After Merge
**Issue**: #1338
**Cause**: Duplicate resources (fonts, images) from source documents are not deduplicated.
```typescript
// 433MB output from 15MB input reported in issue #1338
// No built-in deduplication — each copied page brings its own resources
```

#### Error: Textarea Only Shows Value on Focus
**Issue**: #1584
**Cause**: Appearance streams not updated after setting text.
```typescript
// ANTI-PATTERN
field.setText('value')
// Field appears empty until clicked

// CORRECT PATTERN — ensure appearances are updated
field.setText('value')
form.updateFieldAppearances()  // Or field.updateAppearances(font)
```

#### Error: Diacritics in Field Values
**Issue**: #1488
**Cause**: Standard fonts don't support diacritical marks (é, ñ, ü, etc.).
```typescript
// CORRECT PATTERN for diacritics
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(unicodeFontBytes)
field.setText('café résumé')
form.updateFieldAppearances(font)
```

#### Error: Restricted Document Form Filling
**Issue**: #1732
**Cause**: PDF has permission flags that restrict modification even when form filling is allowed.
```typescript
// WORKAROUND
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true
})
// Then proceed with form filling
```

---

## 15. Decision Trees

### Form Field Selection Decision Tree

```
Need to work with form fields?
├── Reading an existing form?
│   ├── Know field names? → form.getTextField('name') / getCheckBox() / etc.
│   └── Don't know names? → form.getFields().map(f => ({ name: f.getName(), type: f.constructor.name }))
├── Creating a new form?
│   ├── Text input → form.createTextField('name')
│   ├── Yes/No choice → form.createCheckBox('name')
│   ├── One-of-many choice → form.createRadioGroup('name')
│   ├── Selection from list → form.createDropdown('name')
│   ├── Multi-select list → form.createOptionList('name')
│   └── Image/button → form.createButton('name')
└── Making form read-only?
    ├── Permanent (static PDF) → form.flatten()
    └── Still interactive but locked → field.enableReadOnly() on each field
```

### Drawing Shape Decision Tree

```
Need to draw a shape?
├── Rectangle/Box → page.drawRectangle({ x, y, width, height })
├── Square → page.drawSquare({ x, y, size })
├── Circle → page.drawCircle({ x, y, size })
├── Ellipse/Oval → page.drawEllipse({ x, y, xScale, yScale })
├── Straight line → page.drawLine({ start, end, thickness })
├── Complex path → page.drawSvgPath(pathString, { x, y })
├── Text → page.drawText(string, { x, y, size, font })
└── Image → page.drawImage(image, { x, y, width, height })

Color options:
├── Need RGB → rgb(r, g, b)  // 0.0-1.0, NOT 0-255
├── Need CMYK (print) → cmyk(c, m, y, k)  // 0.0-1.0
└── Need grayscale → grayscale(value)  // 0.0=black, 1.0=white
```

### Document Merging Decision Tree

```
Need to combine PDFs?
├── Merge entire documents → Loop: copyPages(source, allIndices) + addPage()
├── Extract specific pages → copyPages(source, [specificIndices])
├── Split into single pages → Loop: create new doc per page, copyPages + addPage
├── Reorder pages → copyPages all, then addPage/insertPage in desired order
└── Remove pages → copyPages only the pages you want to KEEP

CRITICAL: ALWAYS use copyPages() — NEVER add pages directly from another document
```

### Font Decision Tree for Forms

```
Setting text in form fields?
├── Latin characters only (ASCII)?
│   └── Default Helvetica works — no extra setup needed
├── Extended Latin (diacritics: é, ñ, ü)?
│   ├── Register fontkit
│   ├── Embed a Unicode-supporting font (TTF/OTF)
│   └── Call form.updateFieldAppearances(customFont)
└── Non-Latin (CJK, Arabic, etc.)?
    ├── Register fontkit (REQUIRED)
    ├── Embed font that supports the script
    └── Call form.updateFieldAppearances(customFont)
```

---

## 16. Coordinate System Notes

- PDF coordinates use **lower-left origin** (0,0 is bottom-left of page)
- Y increases upward (opposite of most screen coordinate systems)
- Units are in **points** (1 point = 1/72 inch)
- Default page size: US Letter (612 x 792 points = 8.5 x 11 inches)
- A4 size: 595.28 x 841.89 points

```typescript
// Common mistake: treating coordinates as top-left origin
// WRONG — text appears near bottom of page
page.drawText('Hello', { x: 50, y: 50 })

// CORRECT — position from top by subtracting from height
const { height } = page.getSize()
page.drawText('Hello', { x: 50, y: height - 50 })
```

---

## 17. StandardFonts Reference

```typescript
import { StandardFonts } from 'pdf-lib'

// Available standard fonts (no embedding needed):
StandardFonts.Courier
StandardFonts.CourierBold
StandardFonts.CourierBoldOblique
StandardFonts.CourierOblique
StandardFonts.Helvetica
StandardFonts.HelveticaBold
StandardFonts.HelveticaBoldOblique
StandardFonts.HelveticaOblique
StandardFonts.TimesRoman
StandardFonts.TimesRomanBold
StandardFonts.TimesRomanBoldItalic
StandardFonts.TimesRomanItalic
StandardFonts.Symbol
StandardFonts.ZapfDingbats
```

---

## 18. Verification Status

| Source | URL | Status |
|--------|-----|--------|
| pdf-lib official site | https://pdf-lib.js.org | Fetched 2026-03-19 |
| pdf-lib API docs (PDFDocument) | https://pdf-lib.js.org/docs/api/classes/pdfdocument | Fetched 2026-03-19 |
| pdf-lib API docs (PDFPage) | https://pdf-lib.js.org/docs/api/classes/pdfpage | Fetched 2026-03-19 |
| pdf-lib API docs (PDFForm) | https://pdf-lib.js.org/docs/api/classes/pdfform | Fetched 2026-03-19 |
| pdf-lib API docs (PDFTextField) | https://pdf-lib.js.org/docs/api/classes/pdftextfield | Fetched 2026-03-19 |
| pdf-lib API docs (PDFCheckBox) | https://pdf-lib.js.org/docs/api/classes/pdfcheckbox | Fetched 2026-03-19 |
| pdf-lib API docs (PDFRadioGroup) | https://pdf-lib.js.org/docs/api/classes/pdfradiogroup | Fetched 2026-03-19 |
| pdf-lib API docs (PDFDropdown) | https://pdf-lib.js.org/docs/api/classes/pdfdropdown | Fetched 2026-03-19 |
| pdf-lib API docs (PDFOptionList) | https://pdf-lib.js.org/docs/api/classes/pdfoptionlist | Fetched 2026-03-19 |
| pdf-lib API docs (PDFButton) | https://pdf-lib.js.org/docs/api/classes/pdfbutton | Fetched 2026-03-19 |
| GitHub README | https://github.com/Hopding/pdf-lib | Fetched 2026-03-19 |
| GitHub Issues (forms) | https://github.com/Hopding/pdf-lib/issues?q=form+fill | Fetched 2026-03-19 |
| GitHub Issues (merge) | https://github.com/Hopding/pdf-lib/issues?q=merge+copy+pages | Fetched 2026-03-19 |
