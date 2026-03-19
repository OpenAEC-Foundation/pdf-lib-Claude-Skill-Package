# API Signatures — Methods Involved in Common Errors

## Async Methods (MUST await)

```typescript
// PDFDocument — Static
static create(options?: CreateOptions): Promise<PDFDocument>
static load(pdf: string | Uint8Array | ArrayBuffer, options?: LoadOptions): Promise<PDFDocument>

// PDFDocument — Embedding
embedFont(font: StandardFonts | string | Uint8Array | ArrayBuffer, options?: EmbedFontOptions): Promise<PDFFont>
embedPng(png: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
embedJpg(jpg: string | Uint8Array | ArrayBuffer): Promise<PDFImage>
embedPdf(pdf: string | Uint8Array | ArrayBuffer | PDFDocument, indices?: number[]): Promise<PDFEmbeddedPage[]>
embedPage(page: PDFPage, boundingBox?: BoundingBox, transformationMatrix?: TransformationMatrix): Promise<PDFEmbeddedPage>

// PDFDocument — Page operations
copyPages(srcDoc: PDFDocument, indices: number[]): Promise<PDFPage[]>

// PDFDocument — Save
save(options?: SaveOptions): Promise<Uint8Array>
saveAsBase64(options?: SaveOptions): Promise<string>

// PDFDocument — Copy
copy(): Promise<PDFDocument>
```

## Synchronous Methods (do NOT await)

```typescript
// PDFDocument — Page management
addPage(page?: PDFPage | [number, number]): PDFPage
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
removePage(index: number): void
getPage(index: number): PDFPage
getPages(): PDFPage[]
getPageCount(): number

// PDFDocument — Metadata
setKeywords(keywords: string[]): void       // Takes string[]
getKeywords(): string | undefined            // Returns single string!

// PDFDocument — Form
getForm(): PDFForm

// PDFDocument — Font registration
registerFontkit(fontkit: Fontkit): void

// PDFPage — Dimensions
getSize(): { width: number; height: number }
getWidth(): number
getHeight(): number
setSize(width: number, height: number): void
setRotation(rotation: Rotation): void        // Only multiples of 90°
getRotation(): Rotation
```

## Drawing Methods — Correct Parameter Names

```typescript
// PDFPage — drawLine
drawLine(options: {
  start: { x: number; y: number };
  end: { x: number; y: number };
  thickness?: number;        // NOT borderWidth
  color?: Color;
  opacity?: number;
  dashArray?: number[];
  dashPhase?: number;
  lineCap?: LineCapStyle;
}): void

// PDFPage — drawEllipse
drawEllipse(options?: {
  x?: number;
  y?: number;
  xScale?: number;           // NOT width
  yScale?: number;           // NOT height
  color?: Color;
  opacity?: number;
  borderColor?: Color;
  borderWidth?: number;
  borderOpacity?: number;
  borderDashArray?: number[];
  borderDashPhase?: number;
  borderLineCap?: LineCapStyle;
  rotate?: Rotation;
}): void

// PDFPage — drawCircle
drawCircle(options?: {
  x?: number;
  y?: number;
  size?: number;             // Radius, not diameter
  color?: Color;
  opacity?: number;
  borderColor?: Color;
  borderWidth?: number;
}): void
```

## Color Functions — 0.0–1.0 Range

```typescript
rgb(red: number, green: number, blue: number): RGB           // All params: 0.0–1.0
cmyk(cyan: number, magenta: number, yellow: number, key: number): CMYK  // All: 0.0–1.0
grayscale(gray: number): Grayscale                            // 0.0–1.0
```

## Save Options

```typescript
interface SaveOptions {
  useObjectStreams?: boolean;
  addDefaultPage?: boolean;              // Default: true — adds blank page if 0 pages
  objectsPerTick?: number;
  updateFieldAppearances?: boolean;      // Default: true — auto-updates form appearances
}
```

## PDFImage — Scaling Methods

```typescript
// PDFImage
scale(factor: number): { width: number; height: number }
scaleToFit(width: number, height: number): { width: number; height: number }
size(): { width: number; height: number }
```

## PDFForm — Field Access

```typescript
// PDFForm
getFields(): PDFField[]
getTextField(name: string): PDFTextField          // Throws if not found
getCheckBox(name: string): PDFCheckBox            // Throws if not found
getDropdown(name: string): PDFDropdown            // Throws if not found
getRadioGroup(name: string): PDFRadioGroup        // Throws if not found
createTextField(name: string): PDFTextField        // Throws if name already exists
createCheckBox(name: string): PDFCheckBox          // Throws if name already exists
updateFieldAppearances(font?: PDFFont): void
flatten(options?: FlattenOptions): void

// PDFField (base)
getName(): string
```

## Rotation Functions

```typescript
degrees(angle: number): Rotation    // Page rotation: ONLY 0, 90, 180, 270
radians(angle: number): Rotation    // Drawing rotation: any angle allowed
```
