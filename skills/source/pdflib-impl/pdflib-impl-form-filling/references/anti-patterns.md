# Form Filling Anti-Patterns

Common mistakes when working with pdf-lib forms and how to fix them.

## AP-1: Guessing Field Names

Field names are case-sensitive and use fully qualified dot-separated paths.

```typescript
// BROKEN — guessing the field name
form.getTextField('name')
// Throws: No field with name "name" exists

// CORRECT — ALWAYS enumerate fields first
const fields = form.getFields()
fields.forEach(f => console.log(`${f.constructor.name}: "${f.getName()}"`))
// Output: PDFTextField: "form.personal.name"
form.getTextField('form.personal.name').setText('John')
```

NEVER guess field names. ALWAYS enumerate with `form.getFields()` first and use the exact string from `getName()`.

## AP-2: Unicode Text Without Custom Font

The default Helvetica font only supports WinAnsi (basic Latin) characters.

```typescript
// BROKEN — unicode text with default font
form.getTextField('name').setText('König')  // May render incorrectly
form.getTextField('city').setText('東京')  // Characters missing

// CORRECT — register fontkit and use custom font
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const customFont = await pdfDoc.embedFont(fontBytes)
form.getTextField('name').setText('König')
form.getTextField('city').setText('東京')
form.updateFieldAppearances(customFont)  // MUST call after setText
```

ALWAYS register fontkit and embed a custom font when form text contains ANY non-ASCII characters (diacritics, CJK, Cyrillic, Arabic, etc.).

## AP-3: Missing fontkit Registration

Attempting to embed a custom font without registering fontkit first.

```typescript
// BROKEN — fontkit not registered
const pdfDoc = await PDFDocument.load(pdfBytes)
const font = await pdfDoc.embedFont(fontBytes)  // CRASHES

// CORRECT — register fontkit first
import fontkit from '@pdf-lib/fontkit'
const pdfDoc = await PDFDocument.load(pdfBytes)
pdfDoc.registerFontkit(fontkit)                   // MUST come before embedFont
const font = await pdfDoc.embedFont(fontBytes)    // Now works
```

ALWAYS call `pdfDoc.registerFontkit(fontkit)` BEFORE any `pdfDoc.embedFont()` call with custom font bytes (TTF/OTF).

## AP-4: Forgetting updateFieldAppearances

Setting text on fields but not updating appearance streams. Some PDF viewers show blank fields until clicked.

```typescript
// BROKEN — field appears empty in some viewers
form.getTextField('name').setText('John Doe')
// Value is stored but appearance stream is stale

// CORRECT — force appearance update
form.getTextField('name').setText('John Doe')
form.updateFieldAppearances()  // Updates appearance streams for all dirty fields
```

ALWAYS call `form.updateFieldAppearances()` after setting field values if you observe blank fields in PDF viewers. When using a custom font, pass it: `form.updateFieldAppearances(customFont)`.

## AP-5: Duplicate Field Names

Creating two fields with the same name causes a runtime error.

```typescript
// BROKEN — duplicate name
form.createTextField('address')
form.createTextField('address')  // ERROR: A field with name "address" already exists

// CORRECT — use unique names
form.createTextField('address.line1')
form.createTextField('address.line2')
```

ALWAYS use unique field names. Use dot-separated hierarchical names for related fields (e.g., `user.address.city`).

## AP-6: Creating Fields Without Adding to Page

Fields exist in the form data but are invisible without a page widget.

```typescript
// BROKEN — field is created but invisible
const field = form.createTextField('name')
field.setText('John')
// Field exists in PDF data but has no visual representation

// CORRECT — add to page
const field = form.createTextField('name')
field.setText('John')
field.addToPage(page, { x: 50, y: 700, width: 200, height: 25 })
```

ALWAYS call `addToPage()` after creating a field. For radio groups, use `addOptionToPage()` for each option.

## AP-7: Flattening When Form Should Remain Editable

Flattening destroys all field interactivity permanently.

```typescript
// BROKEN — template meant for reuse is flattened
form.getTextField('name').setText('Template Default')
form.flatten()  // Now no one can fill the form

// CORRECT — save without flattening for reusable templates
form.getTextField('name').setText('Template Default')
const pdfBytes = await pdfDoc.save()  // Fields remain editable
```

NEVER flatten forms that need to be filled again later. NEVER flatten if you need to programmatically read field values from the saved PDF.

## AP-8: Reusing a PDFDocument for Multiple Fills

Form field values are modified in place. Filling the same PDFDocument twice overwrites the first fill.

```typescript
// BROKEN — second fill overwrites first
const pdfDoc = await PDFDocument.load(templateBytes)
const form = pdfDoc.getForm()
form.getTextField('name').setText('Alice')
const pdf1 = await pdfDoc.save()
form.getTextField('name').setText('Bob')  // Alice is gone
const pdf2 = await pdfDoc.save()

// CORRECT — load a fresh copy for each fill
for (const name of ['Alice', 'Bob']) {
  const pdfDoc = await PDFDocument.load(templateBytes)
  const form = pdfDoc.getForm()
  form.getTextField('name').setText(name)
  const pdfBytes = await pdfDoc.save()
  // Save pdfBytes to file
}
```

ALWAYS load a fresh `PDFDocument` from the original template bytes for each form fill in a batch operation.

## AP-9: Form Fields Lost During copyPages

When copying pages between documents, AcroForm field definitions may not transfer correctly.

```typescript
// BROKEN — form fields become non-functional after copy
const source = await PDFDocument.load(formPdfBytes)
const target = await PDFDocument.create()
const [page] = await target.copyPages(source, [0])
target.addPage(page)
// Field widgets are visible but non-functional — AcroForm data did not copy

// WORKAROUND — re-create fields after merge if form functionality is needed
const target = await PDFDocument.create()
const [page] = await target.copyPages(source, [0])
target.addPage(page)
const form = target.getForm()
// Re-create fields manually at the same positions
const nameField = form.createTextField('name')
nameField.addToPage(page, { x: 50, y: 700, width: 200, height: 25 })
```

NEVER assume form fields survive `copyPages()`. ALWAYS verify form functionality in the output document. Re-create fields if needed.

## AP-10: Wrong Getter for Field Type

Using the wrong typed getter throws a runtime error.

```typescript
// BROKEN — field is a checkbox but accessed as text field
form.getTextField('agreeTerms')  // Throws if field is actually a PDFCheckBox

// CORRECT — check type first, then use correct getter
const field = form.getField('agreeTerms')
console.log(field.constructor.name)  // "PDFCheckBox"
form.getCheckBox('agreeTerms').check()
```

ALWAYS verify field types via `field.constructor.name` during enumeration before using typed getters.

## AP-11: Button addToPage Signature Mismatch

`PDFButton.addToPage()` takes a text label as the FIRST argument, unlike all other field types.

```typescript
// BROKEN — wrong argument order
const button = form.createButton('submit')
button.addToPage(page, { x: 50, y: 50 })  // ERROR: page is not a string

// CORRECT — text label is the first argument
const button = form.createButton('submit')
button.addToPage('Submit', page, { x: 50, y: 50, width: 100, height: 30 })
```

ALWAYS pass a text label as the first argument to `PDFButton.addToPage()`.
