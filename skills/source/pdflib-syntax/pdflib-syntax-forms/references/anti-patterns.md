# Form Field Anti-Patterns — Common Mistakes & Fixes

## Anti-Pattern 1: Case-Sensitive Field Name Mismatch

Field names in PDF forms are ALWAYS case-sensitive. A single wrong character causes `getTextField()` (and similar methods) to throw.

```typescript
// WRONG — field name has different casing than the actual PDF field
form.getTextField('fullname')  // THROWS — field does not exist

// WRONG — missing space that exists in the actual name
form.getTextField('FullName')  // THROWS — actual name is 'Full Name'

// CORRECT — ALWAYS discover exact names first
const fields = form.getFields()
fields.forEach(f => console.log(`"${f.getName()}" → ${f.constructor.name}`))
// Output: "Full Name" → PDFTextField

form.getTextField('Full Name').setText('John Doe')  // Works
```

**Rule:** ALWAYS call `form.getFields().map(f => f.getName())` to discover exact field names before accessing any field. NEVER guess field names.

## Anti-Pattern 2: Wrong Field Type Getter

Using `getTextField()` on a checkbox (or any type mismatch) throws an error, even if the field name exists.

```typescript
// WRONG — 'Agree' is a PDFCheckBox, not a PDFTextField
form.getTextField('Agree')  // THROWS — wrong type

// CORRECT — use the matching getter
form.getCheckBox('Agree').check()

// SAFEST — discover type first, then use the correct getter
const field = form.getField('Agree')
console.log(field.constructor.name) // 'PDFCheckBox'
```

**Rule:** ALWAYS verify the field type before using a typed getter. Use `form.getField(name).constructor.name` to check.

## Anti-Pattern 3: Duplicate Field Names on Creation

Creating a field with a name that already exists throws an error.

```typescript
// WRONG — creating the same field twice
form.createTextField('user.name')
form.createTextField('user.name')  // THROWS — name already exists

// CORRECT — check first, or use unique names
const existing = form.getFieldMaybe('user.name')
if (!existing) {
  form.createTextField('user.name')
}
```

**Rule:** ALWAYS check if a field name already exists before calling `createTextField()` or any create method. Use `form.getFieldMaybe(name)` to check safely.

## Anti-Pattern 4: Non-Latin Text Without Custom Font

The default form font (Helvetica) only supports WinAnsi (Latin-1) characters. Setting non-Latin text without a custom font produces garbled output or throws.

```typescript
// WRONG — Helvetica cannot render Japanese
form.getTextField('name').setText('田中太郎')
const pdfBytes = await pdfDoc.save() // Text appears garbled or missing

// CORRECT — register fontkit and embed a Unicode font
import fontkit from '@pdf-lib/fontkit'
pdfDoc.registerFontkit(fontkit)
const font = await pdfDoc.embedFont(japaneseFontBytes)

form.getTextField('name').setText('田中太郎')
form.updateFieldAppearances(font) // CRITICAL — MUST call this
const pdfBytes = await pdfDoc.save()
```

**Rule:** For ANY non-Latin text in form fields, ALWAYS: (1) register fontkit, (2) embed a custom font that supports the required characters, (3) call `form.updateFieldAppearances(customFont)` BEFORE saving.

## Anti-Pattern 5: Forgetting updateFieldAppearances After Changing Values

When using a custom font, setting field values without calling `updateFieldAppearances()` results in the old appearance being shown in PDF viewers.

```typescript
// WRONG — appearances not updated after setText with custom font
const font = await pdfDoc.embedFont(fontBytes)
form.getTextField('name').setText('Ünïcödé')
const pdfBytes = await pdfDoc.save() // Shows old/blank appearance

// CORRECT — ALWAYS update appearances after setting values
form.getTextField('name').setText('Ünïcödé')
form.updateFieldAppearances(font) // Refreshes visual rendering
const pdfBytes = await pdfDoc.save()
```

**Rule:** ALWAYS call `form.updateFieldAppearances(font)` after setting field values when using a custom font.

## Anti-Pattern 6: Working with XFA Forms Without Deleting XFA

XFA (XML Forms Architecture) forms are NOT supported by pdf-lib. Attempting to access fields in an XFA form without deleting the XFA data first produces unexpected results or empty fields.

```typescript
// WRONG — XFA data interferes with AcroForm field access
const form = pdfDoc.getForm()
form.getTextField('name').setText('Test') // May not work — XFA takes priority

// CORRECT — ALWAYS check and delete XFA first
const form = pdfDoc.getForm()
if (form.hasXFA()) {
  form.deleteXFA() // Forces PDF viewer to use AcroForm fields instead
}
form.getTextField('name').setText('Test') // Now works correctly
```

**Rule:** ALWAYS call `form.hasXFA()` when working with unknown PDFs. If true, call `form.deleteXFA()` before accessing any fields.

## Anti-Pattern 7: Combing Without MaxLength

Enabling combing on a text field without setting maxLength first produces undefined behavior — the field may not render correctly.

```typescript
// WRONG — combing without maxLength
const field = form.createTextField('code')
field.enableCombing() // Undefined behavior — no cell count defined
field.setText('ABC')

// CORRECT — ALWAYS set maxLength before enabling combing
const field = form.createTextField('code')
field.setMaxLength(6)  // MUST come first
field.enableCombing()  // Now works — 6 equal-width cells
field.setText('ABC123')
```

**Rule:** ALWAYS call `setMaxLength(n)` BEFORE calling `enableCombing()`. Combing without a maxLength produces unpredictable layouts.

## Anti-Pattern 8: Button addToPage Without Text Label

`PDFButton.addToPage()` has a DIFFERENT signature from all other field types — it takes a text label as its first argument.

```typescript
// WRONG — treating button like other field types
const button = form.createButton('submit')
button.addToPage(page, { x: 50, y: 100, width: 100, height: 30 })
// THROWS — first argument must be a string (label text)

// CORRECT — pass text label as first argument
button.addToPage('Submit', page, { x: 50, y: 100, width: 100, height: 30 })
```

**Rule:** ALWAYS pass a text label string as the FIRST argument to `PDFButton.addToPage()`. This is the ONLY field type with this signature difference.

## Anti-Pattern 9: Radio Group Using addToPage Instead of addOptionToPage

Radio groups do NOT have an `addToPage()` method. Each option must be added individually with `addOptionToPage()`.

```typescript
// WRONG — radio groups do NOT have addToPage
const radio = form.createRadioGroup('choice')
radio.addToPage(page, { x: 50, y: 300, width: 20, height: 20 })
// THROWS — method does not exist

// CORRECT — add each option individually
radio.addOptionToPage('Option A', page, { x: 50, y: 300, width: 20, height: 20 })
radio.addOptionToPage('Option B', page, { x: 50, y: 270, width: 20, height: 20 })
radio.select('Option A')
```

**Rule:** ALWAYS use `addOptionToPage(optionName, page, options)` for radio groups. NEVER try to use `addToPage()` on a PDFRadioGroup.

## Anti-Pattern 10: Flattening Before Setting All Values

Once flattened, field values cannot be changed. Setting values after `form.flatten()` has no effect because the fields no longer exist.

```typescript
// WRONG — flattening before all values are set
form.getTextField('name').setText('John')
form.flatten()
form.getTextField('email').setText('john@example.com')
// THROWS — 'email' field no longer exists after flatten

// CORRECT — ALWAYS set all values BEFORE flattening
form.getTextField('name').setText('John')
form.getTextField('email').setText('john@example.com')
form.flatten() // Flatten LAST, after all values are set
```

**Rule:** ALWAYS set ALL field values BEFORE calling `form.flatten()`. Flattening removes all fields from the form — they become static page content.

## Anti-Pattern 11: Forgetting addToPage on Newly Created Fields

Creating a field without calling `addToPage()` means the field exists in the form data but has NO visual widget on any page — it is invisible in the PDF.

```typescript
// WRONG — field created but never placed on a page
const field = form.createTextField('invisible.field')
field.setText('You cannot see me')
// Field exists but has no visual representation

// CORRECT — ALWAYS add to a page after creation
const field = form.createTextField('visible.field')
field.setText('Now you can see me')
field.addToPage(page, { x: 50, y: 500, width: 200, height: 30 })
```

**Rule:** ALWAYS call `addToPage()` (or `addOptionToPage()` for radio groups) after creating a new field. Without it, the field is invisible.
