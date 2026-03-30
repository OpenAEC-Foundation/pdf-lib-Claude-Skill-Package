---
name: pdflib-impl-form-filling
description: >
  Use when filling existing PDF forms or creating form-filling workflows with pdf-lib.
  Prevents the #1 form mistake: not calling form.updateFieldAppearances() with a
  custom font before flattening, causing unicode text to disappear.
  Covers form loading, field filling, checkbox/radio/dropdown, unicode fonts, flattening.
  Keywords: form.flatten, setText, check, select, updateFieldAppearances,
  PDFForm, fillable, fill PDF form, auto-fill, populate form fields,
  lock form after filling, checkbox dropdown.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Form Filling

Complete workflow for filling, creating, and flattening PDF forms with pdf-lib.

## Quick Reference

```typescript
import { PDFDocument } from 'pdf-lib'

// Load PDF and get form
const pdfDoc = await PDFDocument.load(existingPdfBytes)
const form = pdfDoc.getForm()

// ALWAYS enumerate fields first — names are case-sensitive
const fields = form.getFields()
fields.forEach(f => console.log(`${f.constructor.name}: "${f.getName()}"`))

// Fill by type
form.getTextField('Field.Name').setText('value')
form.getCheckBox('Check1').check()
form.getRadioGroup('Group1').select('Option1')
form.getDropdown('Drop1').select('Choice')

// Save editable or flatten
const pdfBytes = await pdfDoc.save()
// OR: form.flatten(); const pdfBytes = await pdfDoc.save()
```

## Step-by-Step: Fill an Existing Form

### Step 1: Load PDF and Get Form

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.load(formPdfBytes)
const form = pdfDoc.getForm()
```

ALWAYS use `PDFDocument.load()` for existing PDFs. NEVER use `PDFDocument.create()` when filling an existing form.

### Step 2: Enumerate Fields

ALWAYS enumerate fields before filling. Field names are case-sensitive and use fully qualified dot-separated names (e.g., `"form.section.fieldName"`).

```typescript
const fields = form.getFields()
fields.forEach(field => {
  const type = field.constructor.name  // PDFTextField, PDFCheckBox, etc.
  const name = field.getName()         // Exact name to use in getTextField() etc.
  console.log(`${type}: "${name}"`)
})
```

### Step 3: Fill Fields by Type

**Text fields:**
```typescript
form.getTextField('CharacterName').setText('Mario')
form.getTextField('Age').setText('24')
```

**Checkboxes:**
```typescript
form.getCheckBox('AcceptTerms').check()
form.getCheckBox('Newsletter').uncheck()
```

**Radio groups:**
```typescript
const radio = form.getRadioGroup('ShippingMethod')
console.log(radio.getOptions())  // See available options first
radio.select('Express')
```

**Dropdowns:**
```typescript
const dropdown = form.getDropdown('Country')
console.log(dropdown.getOptions())  // See available options first
dropdown.select('Netherlands')
```

**Option lists:**
```typescript
const list = form.getOptionList('Skills')
list.select(['TypeScript', 'Python'])  // Multi-select supported
```

**Buttons (with images):**
```typescript
const image = await pdfDoc.embedPng(imageBytes)
form.getButton('Photo').setImage(image)
```

### Step 4: Unicode / Custom Font Support

> See [references/methods.md](references/methods.md) for font method signatures.

#### Font Decision Tree for Forms

```
Setting text in form fields?
+-- ASCII / basic Latin only?
|   +-- Default Helvetica works. No extra setup needed.
+-- Extended Latin (diacritics)?
|   +-- MUST register fontkit + embed custom font
|   +-- MUST call form.updateFieldAppearances(customFont)
+-- Non-Latin (CJK, Arabic, Cyrillic, etc.)?
    +-- MUST register fontkit (REQUIRED)
    +-- MUST embed font that supports the target script
    +-- MUST call form.updateFieldAppearances(customFont)
```

```typescript
import fontkit from '@pdf-lib/fontkit'

// ALWAYS register fontkit BEFORE embedding custom fonts
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)  // TTF or OTF

// Fill fields first
form.getTextField('name').setText('René Descartes')
form.getTextField('city').setText('Tōkyō')

// THEN update appearances with the custom font
form.updateFieldAppearances(customFont)
```

ALWAYS register fontkit before calling `embedFont()` with custom font bytes. Omitting this step causes a runtime crash.

ALWAYS call `form.updateFieldAppearances(customFont)` AFTER setting all field values. This applies the custom font to all dirty fields.

### Step 5: Flatten or Save Editable

#### When to Flatten Decision Tree

```
What is the output purpose?
+-- Final document (invoice, certificate, report)?
|   +-- ALWAYS flatten: form.flatten()
+-- Template for re-use?
|   +-- NEVER flatten. Save editable.
+-- Distribute to prevent modification?
|   +-- ALWAYS flatten: form.flatten()
+-- Need to read values programmatically later?
    +-- NEVER flatten. Field data is destroyed.
```

**Flatten all fields (permanent, non-editable):**
```typescript
form.flatten()  // Converts all fields to static page content
const pdfBytes = await pdfDoc.save()
```

**Save as editable form:**
```typescript
const pdfBytes = await pdfDoc.save()  // Fields remain interactive
```

**Selective flattening workaround:**
`form.flatten()` flattens ALL fields. To keep some fields editable, remove specific fields before flattening:

```typescript
// Remove fields you want to keep as static content
const fieldsToFlatten = ['Name', 'Date', 'Signature']
for (const name of fieldsToFlatten) {
  const field = form.getField(name)
  form.removeField(field)
}
// Remaining fields stay editable — do NOT call form.flatten()
```

> Note: `removeField()` removes the field entirely (both data and visual widget). This is a workaround, not true selective flattening. See GitHub Issues #1758 and #1367 for progress on native selective flattening.

## Step-by-Step: Create a New Form

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage([550, 750])
const form = pdfDoc.getForm()

// Text field
page.drawText('Name:', { x: 50, y: 700, size: 14 })
const nameField = form.createTextField('user.name')
nameField.setText('Default Value')
nameField.addToPage(page, { x: 120, y: 685, width: 200, height: 25 })

// Checkbox
page.drawText('Accept terms:', { x: 50, y: 650, size: 14 })
const termsBox = form.createCheckBox('user.acceptTerms')
termsBox.addToPage(page, { x: 170, y: 645, width: 20, height: 20 })
termsBox.check()

// Radio group
const shipping = form.createRadioGroup('order.shipping')
page.drawText('Standard', { x: 75, y: 600, size: 12 })
shipping.addOptionToPage('Standard', page, { x: 55, y: 598, width: 15, height: 15 })
page.drawText('Express', { x: 75, y: 575, size: 12 })
shipping.addOptionToPage('Express', page, { x: 55, y: 573, width: 15, height: 15 })
shipping.select('Standard')

// Dropdown
const country = form.createDropdown('user.country')
country.addOptions(['Netherlands', 'Belgium', 'Germany'])
country.select('Netherlands')
country.addToPage(page, { x: 120, y: 520, width: 200, height: 25 })

const pdfBytes = await pdfDoc.save()
```

ALWAYS use unique field names. Duplicate names cause a runtime error.

ALWAYS call `addToPage()` (or `addOptionToPage()` for radio groups) to make the field visible. Creating a field without adding it to a page makes it invisible.

## XFA Forms

Some PDFs use XFA (XML Forms Architecture) instead of AcroForm. pdf-lib works with AcroForm only.

```typescript
if (form.hasXFA()) {
  form.deleteXFA()  // Remove XFA to work with AcroForm fields
}
```

ALWAYS check for and delete XFA if you encounter forms that do not respond to `getTextField()` / `getCheckBox()` calls as expected.

## Known Limitations

1. **Appearance streams** — Some viewers show blank fields until clicked. Call `form.updateFieldAppearances()` to force appearance stream generation.
2. **Restricted/encrypted PDFs** — `PDFDocument.load()` may fail on encrypted PDFs. No built-in decryption support.
3. **Form fields lost during `copyPages()`** — AcroForm field definitions may not transfer when copying pages between documents. Field widgets copy but may be non-functional. Workaround: re-create fields after merge.
4. **No selective flatten API** — `form.flatten()` flattens ALL fields. See the selective flattening workaround above.

## Reference Files

- [references/methods.md](references/methods.md) — Complete form method signatures by field type
- [references/examples.md](references/examples.md) — End-to-end form filling workflows
- [references/anti-patterns.md](references/anti-patterns.md) — Common form filling mistakes and fixes
