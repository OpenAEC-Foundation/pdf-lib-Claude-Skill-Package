# PDFForm & Field Type API — Complete Method Reference

## PDFForm

```typescript
const form = pdfDoc.getForm()
```

### Field Retrieval

```typescript
// Get all fields
form.getFields(): PDFField[]

// Get by name (throws if not found)
form.getField(name: string): PDFField

// Get by name (returns undefined if not found)
form.getFieldMaybe(name: string): PDFField | undefined

// Typed getters (throw if not found OR if field is wrong type)
form.getTextField(name: string): PDFTextField
form.getCheckBox(name: string): PDFCheckBox
form.getRadioGroup(name: string): PDFRadioGroup
form.getDropdown(name: string): PDFDropdown
form.getOptionList(name: string): PDFOptionList
form.getButton(name: string): PDFButton
form.getSignature(name: string): PDFSignature

// Default font used for field appearances
form.getDefaultFont(): PDFFont
```

### Field Creation

```typescript
// ALWAYS throws if a field with the same name already exists
form.createTextField(name: string): PDFTextField
form.createCheckBox(name: string): PDFCheckBox
form.createRadioGroup(name: string): PDFRadioGroup
form.createDropdown(name: string): PDFDropdown
form.createOptionList(name: string): PDFOptionList
form.createButton(name: string): PDFButton
// NOTE: There is NO createSignature() — signatures are read-only from existing PDFs
```

### Field Management

```typescript
form.removeField(field: PDFField): void
form.markFieldAsDirty(fieldRef: PDFRef): void
form.markFieldAsClean(fieldRef: PDFRef): void
form.fieldIsDirty(fieldRef: PDFRef): boolean
```

### Form Operations

```typescript
// Flatten: converts all interactive fields to static page content
form.flatten(options?: { updateFieldAppearances?: boolean }): void
// Default: { updateFieldAppearances: true }

// Update visual appearance of all fields (call after changing values with custom fonts)
form.updateFieldAppearances(font?: PDFFont): void

// XFA detection and removal
form.hasXFA(): boolean
form.deleteXFA(): void
```

---

## PDFTextField

```typescript
const field = form.getTextField('name')   // existing
const field = form.createTextField('name') // new
```

### Core Operations

```typescript
field.setText(text: string | undefined): void
field.getText(): string | undefined
field.setFontSize(fontSize: number): void
```

### Layout & Display

```typescript
field.setAlignment(alignment: TextAlignment): void
// TextAlignment.Left | TextAlignment.Center | TextAlignment.Right

field.enableMultiline(): void
field.disableMultiline(): void
field.isMultiline(): boolean

field.enableScrolling(): void
field.disableScrolling(): void
field.isScrollable(): boolean
```

### Input Constraints

```typescript
field.setMaxLength(maxLength?: number | undefined): void
field.getMaxLength(): number | undefined
field.removeMaxLength(): void
```

### Special Behaviors

```typescript
// Combing: distributes characters equally across fixed-width cells
// REQUIRES maxLength to be set first
field.enableCombing(): void
field.disableCombing(): void
field.isCombed(): boolean

// Password masking
field.enablePassword(): void
field.disablePassword(): void
field.isPassword(): boolean

// File selection
field.enableFileSelection(): void
field.disableFileSelection(): void
field.isFileSelector(): boolean

// Rich formatting
field.enableRichFormatting(): void
field.disableRichFormatting(): void
field.isRichFormatted(): boolean

// Spell checking
field.enableSpellChecking(): void
field.disableSpellChecking(): void
field.isSpellChecked(): boolean
```

### Appearance

```typescript
// Add widget to a page (for newly created fields)
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
// FieldAppearanceOptions: { x?: number, y?: number, width?: number, height?: number,
//   textColor?: Color, backgroundColor?: Color, borderColor?: Color, borderWidth?: number,
//   font?: PDFFont }

field.updateAppearances(font: PDFFont, provider?: AppearanceProvider): void
field.defaultUpdateAppearances(font: PDFFont): void
field.setImage(image: PDFImage): void
```

### Query Methods

```typescript
field.getName(): string
field.getAlignment(): TextAlignment
field.needsAppearancesUpdate(): boolean
```

---

## PDFCheckBox

```typescript
const field = form.getCheckBox('name')   // existing
const field = form.createCheckBox('name') // new
```

### Core Operations

```typescript
field.check(): void
field.uncheck(): void
field.isChecked(): boolean
```

### Appearance

```typescript
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(provider?: AppearanceProviderFor<PDFCheckBox>): void
field.defaultUpdateAppearances(): void
```

### Properties (inherited from PDFField)

```typescript
field.getName(): string
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isExported(): boolean
field.needsAppearancesUpdate(): boolean
```

---

## PDFRadioGroup

```typescript
const field = form.getRadioGroup('groupName')   // existing
const field = form.createRadioGroup('groupName') // new
```

### Core Operations

```typescript
field.select(option: string): void
field.getSelected(): string | undefined
field.getOptions(): string[]
field.clear(): void
```

### Widget Management

```typescript
// NOTE: Radio groups use addOptionToPage, NOT addToPage
// Each option gets its own widget on the page
field.addOptionToPage(option: string, page: PDFPage, options?: FieldAppearanceOptions): void
```

### Configuration

```typescript
// Mutual exclusion (default: enabled)
field.enableMutualExclusion(): void
field.disableMutualExclusion(): void
field.isMutuallyExclusive(): boolean

// Off toggling — allow deselecting by clicking the selected option
field.enableOffToggling(): void
field.disableOffToggling(): void
field.isOffToggleable(): boolean
```

### Appearance

```typescript
field.updateAppearances(provider?: AppearanceProviderFor<PDFRadioGroup>): void
field.defaultUpdateAppearances(): void
field.needsAppearancesUpdate(): boolean
```

---

## PDFDropdown

```typescript
const field = form.getDropdown('name')   // existing
const field = form.createDropdown('name') // new
```

### Core Operations

```typescript
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void  // Replaces the ENTIRE option list
```

### Appearance

```typescript
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void
```

### Configuration

```typescript
field.enableMultiselect(): void / field.disableMultiselect(): void
field.enableEditing(): void / field.disableEditing(): void
field.enableSorting(): void / field.disableSorting(): void
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.enableSpellChecking(): void / field.disableSpellChecking(): void
```

---

## PDFOptionList

```typescript
const field = form.getOptionList('name')   // existing
const field = form.createOptionList('name') // new
```

### Core Operations

```typescript
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.clear(): void
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void  // Replaces the ENTIRE option list
```

### Appearance

```typescript
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFOptionList>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean
```

### Configuration

```typescript
field.enableMultiselect(): void / field.disableMultiselect(): void
field.isMultiselect(): boolean
field.enableSorting(): void / field.disableSorting(): void
field.isSorted(): boolean
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.isSelectOnClick(): boolean
```

---

## PDFButton

```typescript
const field = form.getButton('name')   // existing
const field = form.createButton('name') // new
```

### Core Operations

```typescript
field.setImage(image: PDFImage, alignment?: ImageAlignment): void
field.setFontSize(fontSize: number): void
```

### Appearance

```typescript
// CRITICAL: addToPage takes a text LABEL as first argument — this is DIFFERENT from all other field types
field.addToPage(text: string, page: PDFPage, options?: FieldAppearanceOptions): void

field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFButton>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.needsAppearancesUpdate(): boolean
```

### Properties (inherited from PDFField)

```typescript
field.getName(): string
field.enableReadOnly(): void / field.disableReadOnly(): void
field.enableRequired(): void / field.disableRequired(): void
field.enableExporting(): void / field.disableExporting(): void
```

---

## PDFSignature

```typescript
const field = form.getSignature('name') // existing only
// NOTE: There is NO createSignature() method
// Signatures are read-only metadata containers for digital signature fields
```

---

## FieldAppearanceOptions

Used by `addToPage()` on all field types:

```typescript
interface FieldAppearanceOptions {
  x?: number             // Horizontal position (default: 0)
  y?: number             // Vertical position (default: 0)
  width?: number         // Widget width (default: varies by type)
  height?: number        // Widget height (default: varies by type)
  textColor?: Color      // Text color (rgb(), cmyk(), grayscale())
  backgroundColor?: Color // Background fill color
  borderColor?: Color    // Border color
  borderWidth?: number   // Border thickness
  font?: PDFFont         // Font for text rendering
}
```

ALWAYS specify `x`, `y`, `width`, and `height` when calling `addToPage()` — the defaults produce unpredictable layouts.
