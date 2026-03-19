# Form Filling Method Signatures

Complete API reference for pdf-lib form operations, organized by type.

## PDFForm (from `pdfDoc.getForm()`)

### Field Retrieval

```typescript
form.getFields(): PDFField[]                          // All fields
form.getField(name: string): PDFField                 // By name (throws if not found)
form.getFieldMaybe(name: string): PDFField | undefined // By name (returns undefined)
form.getTextField(name: string): PDFTextField
form.getCheckBox(name: string): PDFCheckBox
form.getRadioGroup(name: string): PDFRadioGroup
form.getDropdown(name: string): PDFDropdown
form.getOptionList(name: string): PDFOptionList
form.getButton(name: string): PDFButton
form.getSignature(name: string): PDFSignature
form.getDefaultFont(): PDFFont
```

### Field Creation

```typescript
form.createTextField(name: string): PDFTextField
form.createCheckBox(name: string): PDFCheckBox
form.createRadioGroup(name: string): PDFRadioGroup
form.createDropdown(name: string): PDFDropdown
form.createOptionList(name: string): PDFOptionList
form.createButton(name: string): PDFButton
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
form.flatten(options?: { updateFieldAppearances?: boolean }): void
// Default: { updateFieldAppearances: true }

form.updateFieldAppearances(font?: PDFFont): void
// Pass custom font to render non-Latin text in all dirty fields

form.hasXFA(): boolean
form.deleteXFA(): void
```

## PDFTextField

### Core Operations

```typescript
field.setText(text: string | undefined): void
field.getText(): string | undefined
field.setFontSize(fontSize: number): void
field.setAlignment(alignment: TextAlignment): void
// TextAlignment.Left | TextAlignment.Center | TextAlignment.Right
```

### Layout

```typescript
field.enableMultiline(): void
field.disableMultiline(): void
field.enableScrolling(): void
field.disableScrolling(): void
field.isMultiline(): boolean
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
field.enableCombing(): void     // Distributes chars equally across cells
field.disableCombing(): void
field.enablePassword(): void    // Masks input display
field.disablePassword(): void
field.enableFileSelection(): void
field.disableFileSelection(): void
```

### Appearance

```typescript
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(font: PDFFont, provider?: AppearanceProvider): void
field.defaultUpdateAppearances(font: PDFFont): void
field.setImage(image: PDFImage): void
```

### Properties

```typescript
field.getName(): string
field.enableReadOnly(): void   / field.disableReadOnly(): void
field.enableRequired(): void   / field.disableRequired(): void
field.enableExporting(): void  / field.disableExporting(): void
field.isReadOnly(): boolean
field.isRequired(): boolean
field.isExported(): boolean
field.needsAppearancesUpdate(): boolean
```

## PDFCheckBox

```typescript
field.check(): void
field.uncheck(): void
field.isChecked(): boolean
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(provider?: AppearanceProviderFor<PDFCheckBox>): void
field.defaultUpdateAppearances(): void
field.getName(): string
field.enableReadOnly(): void   / field.disableReadOnly(): void
field.enableRequired(): void   / field.disableRequired(): void
field.isReadOnly(): boolean
field.needsAppearancesUpdate(): boolean
```

## PDFRadioGroup

```typescript
field.select(option: string): void
field.getSelected(): string | undefined
field.getOptions(): string[]
field.clear(): void
field.addOptionToPage(option: string, page: PDFPage, options?: FieldAppearanceOptions): void
field.enableMutualExclusion(): void   / field.disableMutualExclusion(): void
field.isMutuallyExclusive(): boolean
field.enableOffToggling(): void       / field.disableOffToggling(): void
field.isOffToggleable(): boolean
field.updateAppearances(provider?: AppearanceProviderFor<PDFRadioGroup>): void
field.defaultUpdateAppearances(): void
field.getName(): string
field.enableReadOnly(): void   / field.disableReadOnly(): void
field.needsAppearancesUpdate(): boolean
```

## PDFDropdown

```typescript
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void  // Replaces entire list
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void
field.enableMultiselect(): void  / field.disableMultiselect(): void
field.enableEditing(): void      / field.disableEditing(): void
field.enableSorting(): void      / field.disableSorting(): void
field.enableSelectOnClick(): void / field.disableSelectOnClick(): void
field.enableSpellChecking(): void / field.disableSpellChecking(): void
field.getName(): string
field.enableReadOnly(): void   / field.disableReadOnly(): void
field.needsAppearancesUpdate(): boolean
```

## PDFOptionList

```typescript
field.select(options: string | string[], merge?: boolean): void
field.getSelected(): string[]
field.clear(): void
field.addOptions(options: string | string[]): void
field.getOptions(): string[]
field.setOptions(options: string[]): void
field.addToPage(page: PDFPage, options?: FieldAppearanceOptions): void
field.setFontSize(fontSize: number): void
field.enableMultiselect(): void  / field.disableMultiselect(): void
field.isMultiselect(): boolean
field.enableSorting(): void      / field.disableSorting(): void
field.isSorted(): boolean
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFOptionList>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.getName(): string
field.needsAppearancesUpdate(): boolean
```

## PDFButton

```typescript
field.setImage(image: PDFImage, alignment?: ImageAlignment): void
field.setFontSize(fontSize: number): void
// NOTE: addToPage takes a text label as FIRST argument (unlike other field types)
field.addToPage(text: string, page: PDFPage, options?: FieldAppearanceOptions): void
field.updateAppearances(font: PDFFont, provider?: AppearanceProviderFor<PDFButton>): void
field.defaultUpdateAppearances(font: PDFFont): void
field.getName(): string
field.enableReadOnly(): void   / field.disableReadOnly(): void
field.needsAppearancesUpdate(): boolean
```

## PDFSignature

```typescript
const field = form.getSignature('fieldName')
// Read-only metadata container for digital signature fields
// No create method available — signatures are read from existing PDFs only
```

## FieldAppearanceOptions

Used in `addToPage()` calls:

```typescript
interface FieldAppearanceOptions {
  x?: number
  y?: number
  width?: number
  height?: number
  textColor?: Color
  backgroundColor?: Color
  borderColor?: Color
  borderWidth?: number
  font?: PDFFont
  hidden?: boolean
}
```
