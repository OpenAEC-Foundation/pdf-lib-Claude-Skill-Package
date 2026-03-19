# API Method Signatures — pdf-lib 1.x

## PDFDocument — Static Methods

```typescript
static create(options?: CreateOptions): Promise<PDFDocument>
static load(pdf: string | Uint8Array | ArrayBuffer, options?: LoadOptions): Promise<PDFDocument>
```

### CreateOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, dates |

### LoadOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ignoreEncryption` | `boolean` | `false` | Skip encryption validation |
| `parseSpeed` | `number` | — | Parsing performance level |
| `throwOnInvalidObject` | `boolean` | `false` | Throw on malformed PDF objects |
| `updateMetadata` | `boolean` | `true` | Auto-update metadata fields |
| `capNumbers` | `boolean` | `false` | Limit numeric precision |

---

## PDFDocument — Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `catalog` | `PDFCatalog` | The document catalog |
| `context` | `PDFContext` | Low-level document context |
| `defaultWordBreaks` | `string[]` | Word breaks for text wrapping (default: `[' ']`) |
| `isEncrypted` | `boolean` | Whether document is encrypted |

---

## PDFDocument — Page Methods

```typescript
addPage(page?: PDFPage | [number, number]): PDFPage
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
removePage(index: number): void
getPage(index: number): PDFPage
getPages(): PDFPage[]
getPageCount(): number
getPageIndices(): number[]
copyPages(srcDoc: PDFDocument, indices: number[]): Promise<PDFPage[]>
```

---

## PDFDocument — Embed Methods

```typescript
// Fonts
embedFont(font: StandardFonts | string | Uint8Array | ArrayBuffer, options?: EmbedFontOptions): Promise<PDFFont>
embedStandardFont(font: StandardFonts): PDFFont  // SYNC — the only sync embed method

// Images
embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>

// Pages from other PDFs
embedPdf(pdf: string | Uint8Array | ArrayBuffer | PDFDocument, indices?: number[]): Promise<PDFEmbeddedPage[]>
embedPage(page: PDFPage, boundingBox?: PageBoundingBox, transformationMatrix?: TransformationMatrix): Promise<PDFEmbeddedPage>
embedPages(pages: PDFPage[], boundingBoxes?: PageBoundingBox[], transformationMatrices?: TransformationMatrix[]): Promise<PDFEmbeddedPage[]>
```

### EmbedFontOptions

| Property | Type | Description |
|----------|------|-------------|
| `subset` | `boolean` | Enable font subsetting (smaller file, only used glyphs) |
| `customName` | `string` | Custom name for the embedded font |
| `features` | `TypeFeatures` | OpenType font feature configuration |

### PageBoundingBox

```typescript
{ left: number; bottom: number; right: number; top: number }
```

---

## PDFDocument — Save Methods

```typescript
save(options?: SaveOptions): Promise<Uint8Array>
saveAsBase64(options?: Base64SaveOptions): Promise<string>
copy(): Promise<PDFDocument>
flush(): Promise<void>
```

### SaveOptions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `useObjectStreams` | `boolean` | `true` | Enable object stream compression |
| `addDefaultPage` | `boolean` | `true` | Insert blank page if empty |
| `objectsPerTick` | `number` | `50` | Processing batch size per event loop tick |
| `updateFieldAppearances` | `boolean` | `true` | Refresh form field visuals |

### Base64SaveOptions (extends SaveOptions)

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dataUri` | `boolean` | `false` | Return `data:application/pdf;base64,...` format |

---

## PDFDocument — Metadata Methods

```typescript
// Getters
getTitle(): string | undefined
getAuthor(): string | undefined
getSubject(): string | undefined
getKeywords(): string | undefined       // Returns comma-separated string, NOT array
getCreator(): string | undefined
getProducer(): string | undefined
getCreationDate(): Date | undefined
getModificationDate(): Date | undefined

// Setters
setTitle(title: string, options?: { showInWindowTitleBar?: boolean }): void
setAuthor(author: string): void
setSubject(subject: string): void
setKeywords(keywords: string[]): void   // Takes array, getter returns string
setCreator(creator: string): void
setProducer(producer: string): void
setCreationDate(date: Date): void
setModificationDate(date: Date): void
setLanguage(language: string): void     // RFC 3066 language tags (e.g., 'en-US')
```

---

## PDFDocument — Other Methods

```typescript
getForm(): PDFForm
attach(bytes: string | Uint8Array | ArrayBuffer, name: string, options?: AttachmentOptions): Promise<void>
addJavaScript(name: string, script: string): void
registerFontkit(fontkit: Fontkit): void
```

### AttachmentOptions

| Property | Type | Description |
|----------|------|-------------|
| `mimeType` | `string` | MIME type of the attachment |
| `description` | `string` | Human-readable description |
| `creationDate` | `Date` | Creation timestamp |
| `modificationDate` | `Date` | Modification timestamp |

---

## PDFPage — Properties

| Property | Type | Description |
|----------|------|-------------|
| `doc` | `PDFDocument` | The document this page belongs to |
| `node` | `PDFPageLeaf` | Low-level PDF dictionary |
| `ref` | `PDFRef` | Unique reference within the document |

---

## PDFPage — Dimension Methods

```typescript
getSize(): { width: number; height: number }
setSize(width: number, height: number): void
getWidth(): number
setWidth(width: number): void
getHeight(): number
setHeight(height: number): void
```

---

## PDFPage — Box Methods

Each returns/sets `{ x: number; y: number; width: number; height: number }`:

```typescript
getMediaBox(): { x: number; y: number; width: number; height: number }
setMediaBox(x: number, y: number, width: number, height: number): void
getCropBox(): { x: number; y: number; width: number; height: number }
setCropBox(x: number, y: number, width: number, height: number): void
getBleedBox(): { x: number; y: number; width: number; height: number }
setBleedBox(x: number, y: number, width: number, height: number): void
getTrimBox(): { x: number; y: number; width: number; height: number }
setTrimBox(x: number, y: number, width: number, height: number): void
getArtBox(): { x: number; y: number; width: number; height: number }
setArtBox(x: number, y: number, width: number, height: number): void
```

---

## PDFPage — Rotation & Position Methods

```typescript
getRotation(): Rotation
setRotation(angle: Rotation): void    // MUST be multiple of 90 degrees

getPosition(): { x: number; y: number }
getX(): number
getY(): number
moveTo(x: number, y: number): void
moveUp(yIncrease: number): void
moveDown(yDecrease: number): void
moveLeft(xDecrease: number): void
moveRight(xIncrease: number): void
resetPosition(): void
```

---

## PDFPage — Content Transformation Methods

```typescript
translateContent(x: number, y: number): void
scale(x: number, y: number): void           // Scale size + content + annotations
scaleContent(x: number, y: number): void     // Scale content only
scaleAnnotations(x: number, y: number): void // Scale annotations only
```

---

## PDFPage — Default Style Methods

```typescript
setFont(font: PDFFont): void
setFontSize(fontSize: number): void
setFontColor(fontColor: Color): void
setLineHeight(lineHeight: number): void
```

---

## PDFPage — Drawing Methods

```typescript
drawText(text: string, options?: PDFPageDrawTextOptions): void
drawImage(image: PDFImage, options?: PDFPageDrawImageOptions): void
drawRectangle(options?: PDFPageDrawRectangleOptions): void
drawSquare(options?: PDFPageDrawSquareOptions): void
drawCircle(options?: PDFPageDrawCircleOptions): void
drawEllipse(options?: PDFPageDrawEllipseOptions): void
drawLine(options: PDFPageDrawLineOptions): void
drawSvgPath(path: string, options?: PDFPageDrawSVGOptions): void
drawPage(embeddedPage: PDFEmbeddedPage, options?: PDFPageDrawPageOptions): void
pushOperators(...operator: PDFOperator[]): void
```

---

## PDFFont — Properties & Methods

| Property | Type |
|----------|------|
| `doc` | `PDFDocument` |
| `name` | `string` |
| `ref` | `PDFRef` |

```typescript
widthOfTextAtSize(text: string, size: number): number
heightAtSize(size: number, options?: { descender?: boolean }): number
sizeAtHeight(height: number): number
encodeText(text: string): PDFHexString
getCharacterSet(): number[]
embed(): Promise<void>    // Called automatically by save()
```

---

## PDFImage — Properties & Methods

| Property | Type |
|----------|------|
| `doc` | `PDFDocument` |
| `width` | `number` (original pixels) |
| `height` | `number` (original pixels) |
| `ref` | `PDFRef` |

```typescript
scale(factor: number): { width: number; height: number }
scaleToFit(width: number, height: number): { width: number; height: number }
size(): { width: number; height: number }
embed(): Promise<void>    // Called automatically by save()
```

---

## PDFForm — Key Methods

```typescript
// Retrieve fields
getFields(): PDFField[]
getField(name: string): PDFField
getFieldMaybe(name: string): PDFField | undefined
getTextField(name: string): PDFTextField
getCheckBox(name: string): PDFCheckBox
getRadioGroup(name: string): PDFRadioGroup
getDropdown(name: string): PDFDropdown
getOptionList(name: string): PDFOptionList
getButton(name: string): PDFButton
getSignature(name: string): PDFSignature
getDefaultFont(): PDFFont

// Create fields
createTextField(name: string): PDFTextField
createCheckBox(name: string): PDFCheckBox
createRadioGroup(name: string): PDFRadioGroup
createDropdown(name: string): PDFDropdown
createOptionList(name: string): PDFOptionList
createButton(name: string): PDFButton

// Management
removeField(field: PDFField): void
flatten(options?: { updateFieldAppearances?: boolean }): void
updateFieldAppearances(font?: PDFFont): void
hasXFA(): boolean
deleteXFA(): void
```

---

## Helper Functions

```typescript
// Colors (ALL values 0.0 to 1.0)
rgb(red: number, green: number, blue: number): RGB
cmyk(cyan: number, magenta: number, yellow: number, key: number): CMYK
grayscale(gray: number): Grayscale

// Rotation
degrees(angle: number): Rotation
radians(angle: number): Rotation
degreesToRadians(degree: number): number
radiansToDegrees(radian: number): number
```

---

## Enums

| Enum | Values |
|------|--------|
| `StandardFonts` | `Courier`, `CourierBold`, `CourierOblique`, `CourierBoldOblique`, `Helvetica`, `HelveticaBold`, `HelveticaOblique`, `HelveticaBoldOblique`, `TimesRoman`, `TimesRomanBold`, `TimesRomanItalic`, `TimesRomanBoldItalic`, `Symbol`, `ZapfDingbats` |
| `PageSizes` | `A0`-`A10`, `B0`-`B10`, `C0`-`C10`, `RA0`-`RA4`, `SRA0`-`SRA4`, `Executive`, `Folio`, `Legal`, `Letter`, `Tabloid` |
| `TextAlignment` | `Left`, `Center`, `Right` |
| `LineCapStyle` | `Butt`, `Round`, `Square` |
| `LineJoinStyle` | `Miter`, `Round`, `Bevel` |
| `BlendMode` | `Normal`, `Multiply`, `Screen`, `Overlay`, `Darken`, `Lighten`, `ColorDodge`, `ColorBurn`, `HardLight`, `SoftLight`, `Difference`, `Exclusion` |
