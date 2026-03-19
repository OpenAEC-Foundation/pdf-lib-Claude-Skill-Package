# pdf-lib Method Reference — Async/Sync Classification

## PDFDocument Static Methods

| Method | Async | Returns | Notes |
|--------|-------|---------|-------|
| `PDFDocument.create(options?)` | YES | `Promise<PDFDocument>` | ALWAYS await |
| `PDFDocument.load(pdf, options?)` | YES | `Promise<PDFDocument>` | ALWAYS await |

## PDFDocument Instance Methods — ASYNC (require await)

| Method | Returns | Notes |
|--------|---------|-------|
| `embedFont(font, options?)` | `Promise<PDFFont>` | For both standard and custom fonts |
| `embedPng(png)` | `Promise<PDFImage>` | PNG bytes, base64, or data URI |
| `embedJpg(jpg)` | `Promise<PDFImage>` | JPEG bytes, base64, or data URI |
| `copyPages(srcDoc, indices)` | `Promise<PDFPage[]>` | Cross-document page transfer |
| `embedPdf(pdf, indices?)` | `Promise<PDFEmbeddedPage[]>` | Embed pages for drawing |
| `embedPage(page, bbox?, matrix?)` | `Promise<PDFEmbeddedPage>` | Single page embed |
| `embedPages(pages, bboxes?, matrices?)` | `Promise<PDFEmbeddedPage[]>` | Multiple page embed |
| `save(options?)` | `Promise<Uint8Array>` | Serialize to bytes |
| `saveAsBase64(options?)` | `Promise<string>` | Serialize to base64 |
| `copy()` | `Promise<PDFDocument>` | Clone document |
| `flush()` | `Promise<void>` | Flush embedded resources |
| `attach(bytes, name, options?)` | `Promise<void>` | File attachment |

## PDFDocument Instance Methods — SYNC (no await needed)

| Method | Returns | Notes |
|--------|---------|-------|
| `embedStandardFont(font)` | `PDFFont` | ONLY sync font method |
| `addPage(page?)` | `PDFPage` | Add or create page |
| `insertPage(index, page?)` | `PDFPage` | Insert at position |
| `removePage(index)` | `void` | Remove by index |
| `getPage(index)` | `PDFPage` | Get by index |
| `getPages()` | `PDFPage[]` | Get all pages |
| `getPageCount()` | `number` | Page count |
| `getPageIndices()` | `number[]` | Array of indices |
| `getForm()` | `PDFForm` | Get form object |
| `registerFontkit(fontkit)` | `void` | Register fontkit |
| `addJavaScript(name, script)` | `void` | Add JS action |
| `setTitle(title, options?)` | `void` | Set metadata |
| `setAuthor(author)` | `void` | Set metadata |
| `setSubject(subject)` | `void` | Set metadata |
| `setKeywords(keywords)` | `void` | Set metadata |
| `setCreator(creator)` | `void` | Set metadata |
| `setProducer(producer)` | `void` | Set metadata |
| `setCreationDate(date)` | `void` | Set metadata |
| `setModificationDate(date)` | `void` | Set metadata |
| `setLanguage(language)` | `void` | Set metadata |
| `getTitle()` | `string \| undefined` | Get metadata |
| `getAuthor()` | `string \| undefined` | Get metadata |
| `getSubject()` | `string \| undefined` | Get metadata |
| `getKeywords()` | `string \| undefined` | Returns single string, NOT array |
| `getCreator()` | `string \| undefined` | Get metadata |
| `getProducer()` | `string \| undefined` | Get metadata |
| `getCreationDate()` | `Date \| undefined` | Get metadata |
| `getModificationDate()` | `Date \| undefined` | Get metadata |

## PDFPage Drawing Methods — ALL SYNC

| Method | Returns | Notes |
|--------|---------|-------|
| `drawText(text, options?)` | `void` | NEVER needs await |
| `drawImage(image, options?)` | `void` | NEVER needs await |
| `drawRectangle(options?)` | `void` | NEVER needs await |
| `drawSquare(options?)` | `void` | NEVER needs await |
| `drawCircle(options?)` | `void` | NEVER needs await |
| `drawEllipse(options?)` | `void` | NEVER needs await |
| `drawLine(options)` | `void` | NEVER needs await |
| `drawSvgPath(path, options?)` | `void` | NEVER needs await |
| `drawPage(embeddedPage, options?)` | `void` | NEVER needs await |

## PDFPage Dimension/Position Methods — ALL SYNC

| Method | Returns |
|--------|---------|
| `getSize()` | `{ width: number, height: number }` |
| `getWidth()` | `number` |
| `getHeight()` | `number` |
| `setSize(width, height)` | `void` |
| `setWidth(width)` | `void` |
| `setHeight(height)` | `void` |
| `getRotation()` | `Rotation` |
| `setRotation(angle)` | `void` |
| `getPosition()` | `{ x: number, y: number }` |
| `moveTo(x, y)` | `void` |
| `moveUp(y)` | `void` |
| `moveDown(y)` | `void` |
| `moveLeft(x)` | `void` |
| `moveRight(x)` | `void` |
| `resetPosition()` | `void` |
| `setFont(font)` | `void` |
| `setFontSize(size)` | `void` |
| `setFontColor(color)` | `void` |
| `setLineHeight(lineHeight)` | `void` |

## PDFFont Methods — ALL SYNC

| Method | Returns |
|--------|---------|
| `widthOfTextAtSize(text, size)` | `number` |
| `heightAtSize(size, options?)` | `number` |
| `sizeAtHeight(height)` | `number` |
| `encodeText(text)` | `PDFHexString` |
| `getCharacterSet()` | `number[]` |

## PDFImage Methods — ALL SYNC (except embed)

| Method | Returns | Async |
|--------|---------|-------|
| `scale(factor)` | `{ width, height }` | No |
| `scaleToFit(width, height)` | `{ width, height }` | No |
| `size()` | `{ width, height }` | No |
| `embed()` | `Promise<void>` | Yes (called automatically by save) |

## PDFForm Methods — ALL SYNC

| Method | Returns |
|--------|---------|
| `getFields()` | `PDFField[]` |
| `getField(name)` | `PDFField` |
| `getFieldMaybe(name)` | `PDFField \| undefined` |
| `getTextField(name)` | `PDFTextField` |
| `getCheckBox(name)` | `PDFCheckBox` |
| `getRadioGroup(name)` | `PDFRadioGroup` |
| `getDropdown(name)` | `PDFDropdown` |
| `getOptionList(name)` | `PDFOptionList` |
| `getButton(name)` | `PDFButton` |
| `getSignature(name)` | `PDFSignature` |
| `getDefaultFont()` | `PDFFont` |
| `createTextField(name)` | `PDFTextField` |
| `createCheckBox(name)` | `PDFCheckBox` |
| `createRadioGroup(name)` | `PDFRadioGroup` |
| `createDropdown(name)` | `PDFDropdown` |
| `createOptionList(name)` | `PDFOptionList` |
| `createButton(name)` | `PDFButton` |
| `removeField(field)` | `void` |
| `flatten(options?)` | `void` |
| `updateFieldAppearances(font?)` | `void` |
| `hasXFA()` | `boolean` |
| `deleteXFA()` | `void` |

## Common Async Mistake Patterns

**Pattern 1: Missing await on embedFont**
```typescript
// BROKEN: font is a Promise, not a PDFFont
const font = pdfDoc.embedFont(StandardFonts.Helvetica)
page.drawText('Hello', { font }) // Runtime error or silent failure

// FIXED:
const font = await pdfDoc.embedFont(StandardFonts.Helvetica)
```

**Pattern 2: Missing await on save**
```typescript
// BROKEN: pdfBytes is a Promise, not Uint8Array
const pdfBytes = pdfDoc.save()
fs.writeFileSync('output.pdf', pdfBytes) // Writes "[object Promise]"

// FIXED:
const pdfBytes = await pdfDoc.save()
```

**Pattern 3: Missing await on copyPages**
```typescript
// BROKEN: pages is a Promise
const pages = targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(pages[0]) // undefined — Promise has no index

// FIXED:
const pages = await targetDoc.copyPages(sourceDoc, [0])
targetDoc.addPage(pages[0])
```

**Pattern 4: Unnecessary await on sync methods**
```typescript
// UNNECESSARY but harmless: await on sync method
const page = await pdfDoc.addPage() // Works but misleading

// CORRECT:
const page = pdfDoc.addPage()
```
