---
name: pdflib-syntax-forms
description: >
  Use when working with PDF form fields in pdf-lib — reading, creating, or
  manipulating fields. Prevents the common mistake of not calling form.flatten()
  after filling, leaving forms editable when they should be locked.
  Covers getForm, all field types (text, checkbox, radio, dropdown), field properties, flattening.
  Keywords: getForm, PDFForm, PDFTextField, PDFCheckBox, PDFRadioGroup, PDFDropdown, flatten.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Form Fields: Syntax Reference

## Quick Reference

```typescript
import { PDFDocument } from 'pdf-lib'

// Get form from existing PDF
const pdfDoc = await PDFDocument.load(existingPdfBytes)
const form = pdfDoc.getForm()

// Discover all field names
const fields = form.getFields()
fields.forEach(f => console.log(f.getName(), f.constructor.name))

// Access typed fields (throws if not found or wrong type)
const textField = form.getTextField('Full Name')
const checkbox = form.getCheckBox('Agree')
const radio = form.getRadioGroup('Priority')
const dropdown = form.getDropdown('Country')
const optionList = form.getOptionList('Languages')
const button = form.getButton('Logo')

// Safe access (returns undefined if not found)
const maybeMissing = form.getFieldMaybe('MightNotExist')

// Set values
textField.setText('John Doe')
checkbox.check()
radio.select('High')
dropdown.select('Netherlands')
optionList.select(['TypeScript', 'Python'])

// Flatten (makes all fields non-editable, bakes values into page content)
form.flatten()

const pdfBytes = await pdfDoc.save()
```

## Critical Warnings

> **CASE-SENSITIVE FIELD NAMES**: Field names are ALWAYS case-sensitive. `form.getTextField('fullName')` and `form.getTextField('FullName')` are DIFFERENT fields. ALWAYS use `form.getFields().map(f => f.getName())` to discover exact field names before accessing them. A single wrong character causes a throw.

> **DEFAULT FONT LIMITATION**: The default form font (Helvetica) ONLY supports Latin characters (WinAnsi encoding). For ANY non-Latin text (accented characters, CJK, Arabic, Cyrillic), you MUST register fontkit, embed a custom font, and call `form.updateFieldAppearances(customFont)`. Failure to do this produces garbled output or throws.

> **XFA FORMS**: Some PDFs use XFA (XML Forms Architecture) instead of AcroForm. pdf-lib does NOT support XFA. ALWAYS call `form.deleteXFA()` before working with fields from XFA-based PDFs — this forces fallback to AcroForm fields.

> **FLATTEN IS ALL-OR-NOTHING**: `form.flatten()` flattens ALL fields. There is no built-in selective flattening. To flatten specific fields only, use `form.removeField(field)` on the fields you want to keep interactive BEFORE calling `form.flatten()`.

## Essential Patterns

### Pattern 1: Fill an Existing Form

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.load(formPdfBytes)
const form = pdfDoc.getForm()

// ALWAYS discover field names first
const fieldNames = form.getFields().map(f => f.getName())
console.log(fieldNames) // e.g., ['CharacterName 2', 'Check Box3', 'Group2']

// Fill fields by exact name
form.getTextField('CharacterName 2').setText('Mario')
form.getCheckBox('Check Box3').check()
form.getRadioGroup('Group2').select('Choice1')
form.getDropdown('Dropdown7').select('Infinity')

// Embed image on button
const marioImage = await pdfDoc.embedPng(marioImageBytes)
form.getButton('CHARACTER IMAGE').setImage(marioImage)

// Flatten to prevent editing
form.flatten()
const pdfBytes = await pdfDoc.save()
```

### Pattern 2: Create a New Form

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.create()
const page = pdfDoc.addPage([550, 750])
const form = pdfDoc.getForm()

// Text field
const nameField = form.createTextField('user.name')
nameField.setText('Default Name')
nameField.addToPage(page, { x: 50, y: 650, width: 200, height: 30 })

// Checkbox
const agreeField = form.createCheckBox('user.agree')
agreeField.addToPage(page, { x: 50, y: 600, width: 20, height: 20 })
agreeField.check()

// Radio group
const priorityField = form.createRadioGroup('user.priority')
priorityField.addOptionToPage('High', page, { x: 50, y: 550, width: 20, height: 20 })
priorityField.addOptionToPage('Low', page, { x: 50, y: 520, width: 20, height: 20 })
priorityField.select('High')

// Dropdown
const countryField = form.createDropdown('user.country')
countryField.addOptions(['Netherlands', 'Germany', 'France'])
countryField.select('Netherlands')
countryField.addToPage(page, { x: 50, y: 460, width: 200, height: 30 })

// Option list
const langField = form.createOptionList('user.languages')
langField.addOptions(['TypeScript', 'Python', 'Rust'])
langField.select('TypeScript')
langField.addToPage(page, { x: 50, y: 350, width: 200, height: 80 })

const pdfBytes = await pdfDoc.save()
```

### Pattern 3: Non-Latin Text in Form Fields

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.load(formPdfBytes)
pdfDoc.registerFontkit(fontkit)

const customFont = await pdfDoc.embedFont(unicodeFontBytes) // TTF or OTF
const form = pdfDoc.getForm()

// Fill fields with non-Latin text
form.getTextField('name').setText('日本語テスト')

// CRITICAL: update appearances with the custom font
form.updateFieldAppearances(customFont)

const pdfBytes = await pdfDoc.save()
```

### Pattern 4: Make Fields Read-Only Without Flattening

```typescript
const form = pdfDoc.getForm()
const fields = form.getFields()

// Lock all fields but keep them interactive (visually editable but not modifiable)
fields.forEach(field => field.enableReadOnly())

// To unlock later:
// fields.forEach(field => field.disableReadOnly())
```

### Pattern 5: XFA Form Handling

```typescript
const pdfDoc = await PDFDocument.load(xfaPdfBytes)
const form = pdfDoc.getForm()

if (form.hasXFA()) {
  // ALWAYS delete XFA to work with AcroForm fields
  form.deleteXFA()
}

// Now access fields normally
const fields = form.getFields()
```

## Decision Tree: Field Type Selection

```
What kind of input do you need?
├── Free text input? → createTextField('name')
│   ├── Single line? → default (no extra config)
│   ├── Multiline? → field.enableMultiline()
│   ├── Password? → field.enablePassword()
│   └── Fixed-width cells? → field.enableCombing() + field.setMaxLength(n)
├── Yes/No toggle? → createCheckBox('name')
├── One choice from several? (mutually exclusive)
│   ├── Small set (2-5)? → createRadioGroup('name')
│   └── Large set (6+)? → createDropdown('name')
├── Multiple choices from a list?
│   ├── Compact display? → createDropdown('name') + enableMultiselect()
│   └── Visible list? → createOptionList('name') + enableMultiselect()
└── Image or button? → createButton('name')
```

## Decision Tree: Form Font Selection

```
Setting text in form fields?
├── Latin characters only (ASCII)?
│   └── Default Helvetica works — no extra setup needed
├── Extended Latin (accents, diacritics)?
│   ├── Register fontkit
│   ├── Embed a Unicode-supporting font (TTF/OTF)
│   └── Call form.updateFieldAppearances(customFont)
└── Non-Latin (CJK, Arabic, Cyrillic)?
    ├── Register fontkit (REQUIRED)
    ├── Embed font that supports the target script
    └── Call form.updateFieldAppearances(customFont)
```

## Decision Tree: Form Finalization

```
Done filling the form?
├── Make permanently read-only (static PDF)?
│   └── form.flatten()
├── Keep interactive but locked?
│   └── fields.forEach(f => f.enableReadOnly())
├── Need selective flattening?
│   └── Remove fields to KEEP interactive, then form.flatten()
└── Keep fully editable?
    └── Just save — do nothing extra
```

## PDFForm API Summary

| Method | Returns | Throws |
|--------|---------|--------|
| `pdfDoc.getForm()` | `PDFForm` | Never |
| `form.getFields()` | `PDFField[]` | Never |
| `form.getField(name)` | `PDFField` | If not found |
| `form.getFieldMaybe(name)` | `PDFField \| undefined` | Never |
| `form.getTextField(name)` | `PDFTextField` | If not found or wrong type |
| `form.getCheckBox(name)` | `PDFCheckBox` | If not found or wrong type |
| `form.getRadioGroup(name)` | `PDFRadioGroup` | If not found or wrong type |
| `form.getDropdown(name)` | `PDFDropdown` | If not found or wrong type |
| `form.getOptionList(name)` | `PDFOptionList` | If not found or wrong type |
| `form.getButton(name)` | `PDFButton` | If not found or wrong type |
| `form.getSignature(name)` | `PDFSignature` | If not found or wrong type |
| `form.createTextField(name)` | `PDFTextField` | If name already exists |
| `form.createCheckBox(name)` | `PDFCheckBox` | If name already exists |
| `form.createRadioGroup(name)` | `PDFRadioGroup` | If name already exists |
| `form.createDropdown(name)` | `PDFDropdown` | If name already exists |
| `form.createOptionList(name)` | `PDFOptionList` | If name already exists |
| `form.createButton(name)` | `PDFButton` | If name already exists |
| `form.removeField(field)` | `void` | Never |
| `form.flatten(options?)` | `void` | Never |
| `form.updateFieldAppearances(font?)` | `void` | Never |
| `form.hasXFA()` | `boolean` | Never |
| `form.deleteXFA()` | `void` | Never |
| `form.getDefaultFont()` | `PDFFont` | Never |
| `form.markFieldAsDirty(ref)` | `void` | Never |
| `form.markFieldAsClean(ref)` | `void` | Never |

## Common Field Properties (All Field Types)

| Method | Purpose |
|--------|---------|
| `field.getName()` | Get fully qualified field name |
| `field.enableReadOnly()` | Prevent user modification |
| `field.disableReadOnly()` | Allow user modification |
| `field.enableRequired()` | Mark as required |
| `field.disableRequired()` | Mark as optional |
| `field.enableExporting()` | Include in form data export |
| `field.disableExporting()` | Exclude from form data export |
| `field.isReadOnly()` | Check read-only state |
| `field.isRequired()` | Check required state |
| `field.isExported()` | Check export state |
| `field.needsAppearancesUpdate()` | Check if visual refresh needed |

## Reference Links

- [PDFForm & Field Type API Methods](references/methods.md)
- [Field Access, Creation & Property Examples](references/examples.md)
- [Anti-Patterns — Field Names, Duplicates, XFA Issues](references/anti-patterns.md)
- [pdf-lib API Docs — PDFForm](https://pdf-lib.js.org/docs/api/classes/pdfform)
- [pdf-lib API Docs — PDFTextField](https://pdf-lib.js.org/docs/api/classes/pdftextfield)
- [pdf-lib API Docs — PDFCheckBox](https://pdf-lib.js.org/docs/api/classes/pdfcheckbox)
- [pdf-lib API Docs — PDFRadioGroup](https://pdf-lib.js.org/docs/api/classes/pdfradiogroup)
- [pdf-lib API Docs — PDFDropdown](https://pdf-lib.js.org/docs/api/classes/pdfdropdown)
- [pdf-lib API Docs — PDFOptionList](https://pdf-lib.js.org/docs/api/classes/pdfoptionlist)
- [pdf-lib API Docs — PDFButton](https://pdf-lib.js.org/docs/api/classes/pdfbutton)
