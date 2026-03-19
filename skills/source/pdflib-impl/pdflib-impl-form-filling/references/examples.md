# Form Filling Examples

Complete end-to-end workflows for common form filling scenarios.

## Example 1: Fill an Existing PDF Form (Basic)

```typescript
import { PDFDocument } from 'pdf-lib'
import fs from 'fs'

async function fillBasicForm(): Promise<Uint8Array> {
  // Step 1: Load PDF
  const formPdfBytes = fs.readFileSync('template.pdf')
  const pdfDoc = await PDFDocument.load(formPdfBytes)
  const form = pdfDoc.getForm()

  // Step 2: ALWAYS enumerate fields first
  const fields = form.getFields()
  fields.forEach(field => {
    console.log(`${field.constructor.name}: "${field.getName()}"`)
  })

  // Step 3: Fill fields using exact names from enumeration
  form.getTextField('applicant.firstName').setText('John')
  form.getTextField('applicant.lastName').setText('Doe')
  form.getTextField('applicant.email').setText('john@example.com')
  form.getCheckBox('applicant.agreeTerms').check()
  form.getDropdown('applicant.department').select('Engineering')

  // Step 4: Flatten for final output
  form.flatten()

  // Step 5: Save
  return await pdfDoc.save()
}
```

## Example 2: Fill Form with Unicode / Custom Font

```typescript
import { PDFDocument } from 'pdf-lib'
import fontkit from '@pdf-lib/fontkit'
import fs from 'fs'

async function fillUnicodeForm(): Promise<Uint8Array> {
  const formPdfBytes = fs.readFileSync('template.pdf')
  const pdfDoc = await PDFDocument.load(formPdfBytes)

  // MUST register fontkit BEFORE embedding custom fonts
  pdfDoc.registerFontkit(fontkit)

  // Embed a font that supports the required characters
  const fontBytes = fs.readFileSync('NotoSans-Regular.ttf')
  const customFont = await pdfDoc.embedFont(fontBytes)

  const form = pdfDoc.getForm()

  // Enumerate fields
  form.getFields().forEach(f =>
    console.log(`${f.constructor.name}: "${f.getName()}"`)
  )

  // Fill with unicode text
  form.getTextField('name').setText('René Descartes')
  form.getTextField('city').setText('Malmö')
  form.getTextField('notes').setText('Begründet durch Leibniz')

  // MUST call updateFieldAppearances AFTER setting all values
  form.updateFieldAppearances(customFont)

  form.flatten()
  return await pdfDoc.save()
}
```

## Example 3: Fill Form with Images in Buttons

```typescript
import { PDFDocument } from 'pdf-lib'
import fs from 'fs'

async function fillFormWithImages(): Promise<Uint8Array> {
  const formPdfBytes = fs.readFileSync('template.pdf')
  const pdfDoc = await PDFDocument.load(formPdfBytes)
  const form = pdfDoc.getForm()

  // Embed the image (ONLY png and jpg supported)
  const photoBytes = fs.readFileSync('photo.png')
  const photo = await pdfDoc.embedPng(photoBytes)

  const signatureBytes = fs.readFileSync('signature.jpg')
  const signature = await pdfDoc.embedJpg(signatureBytes)

  // Set images on button fields
  form.getButton('PHOTO').setImage(photo)
  form.getButton('SIGNATURE').setImage(signature)

  form.flatten()
  return await pdfDoc.save()
}
```

## Example 4: Create a New Interactive Form

```typescript
import { PDFDocument, rgb } from 'pdf-lib'

async function createNewForm(): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.create()
  const page = pdfDoc.addPage([595.28, 841.89])  // A4
  const form = pdfDoc.getForm()

  // Title
  page.drawText('Registration Form', { x: 50, y: 780, size: 24 })

  // Text field: Full Name
  page.drawText('Full Name:', { x: 50, y: 740, size: 12 })
  const nameField = form.createTextField('registration.fullName')
  nameField.addToPage(page, { x: 150, y: 725, width: 300, height: 25 })

  // Text field: Email
  page.drawText('Email:', { x: 50, y: 700, size: 12 })
  const emailField = form.createTextField('registration.email')
  emailField.addToPage(page, { x: 150, y: 685, width: 300, height: 25 })

  // Multiline text field: Comments
  page.drawText('Comments:', { x: 50, y: 660, size: 12 })
  const commentsField = form.createTextField('registration.comments')
  commentsField.enableMultiline()
  commentsField.addToPage(page, { x: 150, y: 590, width: 300, height: 80 })

  // Checkbox: Newsletter
  page.drawText('Subscribe to newsletter:', { x: 50, y: 560, size: 12 })
  const newsletterBox = form.createCheckBox('registration.newsletter')
  newsletterBox.addToPage(page, { x: 230, y: 555, width: 18, height: 18 })

  // Radio group: Preferred contact method
  page.drawText('Contact method:', { x: 50, y: 520, size: 12 })
  const contactMethod = form.createRadioGroup('registration.contactMethod')
  page.drawText('Email', { x: 75, y: 495, size: 11 })
  contactMethod.addOptionToPage('Email', page, { x: 55, y: 493, width: 15, height: 15 })
  page.drawText('Phone', { x: 75, y: 475, size: 11 })
  contactMethod.addOptionToPage('Phone', page, { x: 55, y: 473, width: 15, height: 15 })
  contactMethod.select('Email')

  // Dropdown: Country
  page.drawText('Country:', { x: 50, y: 440, size: 12 })
  const countryField = form.createDropdown('registration.country')
  countryField.addOptions(['Netherlands', 'Belgium', 'Germany', 'France', 'United Kingdom'])
  countryField.select('Netherlands')
  countryField.addToPage(page, { x: 150, y: 425, width: 200, height: 25 })

  return await pdfDoc.save()
}
```

## Example 5: Read Form Field Values

```typescript
import { PDFDocument, PDFTextField, PDFCheckBox, PDFDropdown } from 'pdf-lib'

async function readFormValues(pdfBytes: Uint8Array): Promise<void> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const form = pdfDoc.getForm()

  const fields = form.getFields()
  for (const field of fields) {
    const name = field.getName()
    const type = field.constructor.name

    if (field instanceof PDFTextField) {
      console.log(`${name} (text): ${field.getText()}`)
    } else if (field instanceof PDFCheckBox) {
      console.log(`${name} (checkbox): ${field.isChecked()}`)
    } else if (field instanceof PDFDropdown) {
      console.log(`${name} (dropdown): ${field.getSelected()}`)
    } else {
      console.log(`${name} (${type}): [use type-specific getter]`)
    }
  }
}
```

## Example 6: Batch Fill Multiple PDFs from Data

```typescript
import { PDFDocument } from 'pdf-lib'

interface FormData {
  name: string
  email: string
  department: string
  approved: boolean
}

async function batchFillForms(
  templateBytes: Uint8Array,
  dataRecords: FormData[]
): Promise<Uint8Array[]> {
  const results: Uint8Array[] = []

  for (const data of dataRecords) {
    // ALWAYS load a fresh copy for each fill — forms are modified in place
    const pdfDoc = await PDFDocument.load(templateBytes)
    const form = pdfDoc.getForm()

    form.getTextField('employee.name').setText(data.name)
    form.getTextField('employee.email').setText(data.email)
    form.getDropdown('employee.department').select(data.department)

    if (data.approved) {
      form.getCheckBox('employee.approved').check()
    }

    form.flatten()
    results.push(await pdfDoc.save())
  }

  return results
}
```

## Example 7: Handle XFA Forms

```typescript
import { PDFDocument } from 'pdf-lib'

async function handleXfaForm(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const form = pdfDoc.getForm()

  // Check for XFA — pdf-lib only works with AcroForm
  if (form.hasXFA()) {
    console.log('XFA form detected — removing XFA to use AcroForm fields')
    form.deleteXFA()
  }

  // Now work with AcroForm fields as normal
  const fields = form.getFields()
  fields.forEach(f => console.log(`${f.constructor.name}: "${f.getName()}"`))

  // Fill fields...
  return await pdfDoc.save()
}
```

## Example 8: Selective Field Removal Before Flatten

```typescript
import { PDFDocument } from 'pdf-lib'

async function selectiveFlatten(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const pdfDoc = await PDFDocument.load(pdfBytes)
  const form = pdfDoc.getForm()

  // Fill all fields
  form.getTextField('name').setText('John Doe')
  form.getTextField('date').setText('2026-03-19')
  form.getTextField('notes').setText('Approved')

  // Remove fields that should become static (their visual content stays)
  // Keep 'notes' field editable
  const fieldsToRemove = ['name', 'date']
  for (const fieldName of fieldsToRemove) {
    const field = form.getField(fieldName)
    form.removeField(field)
  }

  // Do NOT call form.flatten() — removed fields are gone,
  // remaining fields ('notes') stay editable
  return await pdfDoc.save()
}
```
