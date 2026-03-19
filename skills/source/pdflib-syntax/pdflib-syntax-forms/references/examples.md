# Form Field Examples — Access, Creation & Property Manipulation

## Example 1: Discover All Fields in an Existing PDF

```typescript
import { PDFDocument } from 'pdf-lib'

const pdfDoc = await PDFDocument.load(existingPdfBytes)
const form = pdfDoc.getForm()

// List all fields with their names and types
const fields = form.getFields()
fields.forEach(field => {
  const name = field.getName()
  const type = field.constructor.name
  console.log(`${name} (${type})`)
})
// Output example:
// Full Name (PDFTextField)
// Agree to Terms (PDFCheckBox)
// Priority (PDFRadioGroup)
// Country (PDFDropdown)
```

ALWAYS discover field names before accessing them. Field names are case-sensitive and often contain spaces or dots.

## Example 2: Safe Field Access with getFieldMaybe

```typescript
const form = pdfDoc.getForm()

// Safe access — does NOT throw if field is missing
const field = form.getFieldMaybe('Optional Field')
if (field) {
  // Field exists — determine type and act accordingly
  if (field.constructor.name === 'PDFTextField') {
    form.getTextField('Optional Field').setText('Found it')
  }
}
```

## Example 3: Text Field with All Options

```typescript
const form = pdfDoc.getForm()
const field = form.createTextField('address.street')

// Layout
field.enableMultiline()
field.enableScrolling()
field.setAlignment(TextAlignment.Left)
field.setFontSize(12)

// Constraints
field.setMaxLength(200)

// Properties
field.enableRequired()
field.enableReadOnly()  // lock after filling

// Add to page
field.setText('123 Main Street\nApt 4B')
field.addToPage(page, {
  x: 50,
  y: 400,
  width: 300,
  height: 60,
  borderColor: rgb(0, 0, 0),
  borderWidth: 1,
})
```

## Example 4: Combed Text Field (Fixed-Width Cells)

```typescript
const form = pdfDoc.getForm()
const ssnField = form.createTextField('ssn')

// Combing distributes characters into equal-width cells
// MUST set maxLength BEFORE enabling combing
ssnField.setMaxLength(9)
ssnField.enableCombing()
ssnField.setText('123456789')
ssnField.addToPage(page, { x: 50, y: 500, width: 270, height: 30 })
```

## Example 5: Password Field

```typescript
const form = pdfDoc.getForm()
const passField = form.createTextField('user.password')
passField.enablePassword()  // Masks input with dots/asterisks
passField.addToPage(page, { x: 50, y: 450, width: 200, height: 30 })
```

## Example 6: Checkbox — Create and Read

```typescript
const form = pdfDoc.getForm()

// Create
const agreeBox = form.createCheckBox('terms.agree')
agreeBox.check()
agreeBox.addToPage(page, { x: 50, y: 400, width: 20, height: 20 })

// Read from existing
const existingBox = form.getCheckBox('terms.agree')
const isChecked = existingBox.isChecked() // true or false
```

## Example 7: Radio Group with Multiple Options

```typescript
const form = pdfDoc.getForm()
const priority = form.createRadioGroup('task.priority')

// Each option gets its own widget on the page
priority.addOptionToPage('High', page, { x: 50, y: 350, width: 20, height: 20 })
priority.addOptionToPage('Medium', page, { x: 50, y: 320, width: 20, height: 20 })
priority.addOptionToPage('Low', page, { x: 50, y: 290, width: 20, height: 20 })

// Select one
priority.select('High')

// Read
const selected = priority.getSelected()  // 'High'
const options = priority.getOptions()    // ['High', 'Medium', 'Low']

// Mutual exclusion (default: enabled) — selecting one deselects others
priority.enableMutualExclusion()

// Off toggling — allow clicking selected option to deselect it
priority.enableOffToggling()
```

## Example 8: Dropdown — Create, Fill, Configure

```typescript
const form = pdfDoc.getForm()
const country = form.createDropdown('address.country')

// Add options
country.addOptions(['Netherlands', 'Germany', 'France', 'Belgium'])

// Select value
country.select('Netherlands')

// Read
const selected = country.getSelected() // ['Netherlands']
const options = country.getOptions()   // ['Netherlands', 'Germany', 'France', 'Belgium']

// Replace all options
country.setOptions(['UK', 'Ireland', 'Scotland'])

// Enable user to type a custom value (not just pick from list)
country.enableEditing()

// Enable multi-select
country.enableMultiselect()
country.select(['UK', 'Ireland']) // select multiple

// Sort options alphabetically
country.enableSorting()

// Display
country.setFontSize(12)
country.addToPage(page, { x: 50, y: 250, width: 200, height: 30 })
```

## Example 9: Option List — Visible Multi-Select

```typescript
const form = pdfDoc.getForm()
const langs = form.createOptionList('skills.languages')

langs.addOptions(['TypeScript', 'Python', 'Rust', 'Go', 'Java'])
langs.enableMultiselect()
langs.select(['TypeScript', 'Rust'])

// Read
const selected = langs.getSelected()  // ['TypeScript', 'Rust']
const allOptions = langs.getOptions() // ['TypeScript', 'Python', 'Rust', 'Go', 'Java']

// Clear selection
langs.clear()

// Sort alphabetically
langs.enableSorting()

// Display (ALWAYS use a tall height to show multiple options)
langs.setFontSize(11)
langs.addToPage(page, { x: 50, y: 100, width: 200, height: 100 })
```

## Example 10: Button with Image

```typescript
const form = pdfDoc.getForm()
const logoButton = form.createButton('company.logo')

// Set image on button
const logoImage = await pdfDoc.embedPng(logoPngBytes)
logoButton.setImage(logoImage)

// CRITICAL: addToPage for buttons takes a text LABEL as the first argument
logoButton.addToPage('Click Me', page, { x: 50, y: 50, width: 150, height: 50 })
```

## Example 11: Batch Field Properties

```typescript
const form = pdfDoc.getForm()
const fields = form.getFields()

// Make all fields read-only
fields.forEach(field => {
  field.enableReadOnly()
  field.enableExporting()
})

// Make specific fields required
const requiredFieldNames = ['Full Name', 'Email', 'Phone']
requiredFieldNames.forEach(name => {
  const field = form.getFieldMaybe(name)
  if (field) {
    field.enableRequired()
  }
})
```

## Example 12: Remove Specific Fields

```typescript
const form = pdfDoc.getForm()

// Remove a single field by name
const obsoleteField = form.getFieldMaybe('old.field.name')
if (obsoleteField) {
  form.removeField(obsoleteField)
}

// Remove all fields of a specific type
const fields = form.getFields()
fields.forEach(field => {
  if (field.constructor.name === 'PDFButton') {
    form.removeField(field)
  }
})
```

## Example 13: Flatten with Custom Font

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'

const pdfDoc = await PDFDocument.load(formPdfBytes)
pdfDoc.registerFontkit(fontkit)

const customFont = await pdfDoc.embedFont(fontBytes)
const form = pdfDoc.getForm()

// Fill all fields
form.getTextField('name').setText('Ünïcödé Tëxt')

// Update appearances with custom font BEFORE flattening
form.updateFieldAppearances(customFont)

// Flatten — bakes values into page content permanently
form.flatten()

const pdfBytes = await pdfDoc.save()
```

## Example 14: Selective Flattening Workaround

```typescript
const form = pdfDoc.getForm()

// Suppose you want to flatten all fields EXCEPT 'signature'
const signatureField = form.getFieldMaybe('signature')

// Remove the field you want to keep interactive
if (signatureField) {
  form.removeField(signatureField)
}

// Flatten remaining fields
form.flatten()

// NOTE: The removed field is gone entirely — there is no way to
// "flatten some and keep others" without losing the kept fields.
// This is a known limitation of pdf-lib's current API.
```

## Example 15: Form Field Appearance Customization

```typescript
const form = pdfDoc.getForm()
const field = form.createTextField('styled.field')

field.setText('Styled Input')
field.addToPage(page, {
  x: 50,
  y: 300,
  width: 250,
  height: 35,
  textColor: rgb(0, 0, 0.5),         // dark blue text
  backgroundColor: rgb(0.95, 0.95, 1), // light blue background
  borderColor: rgb(0, 0, 0.8),        // blue border
  borderWidth: 2,
})
```
