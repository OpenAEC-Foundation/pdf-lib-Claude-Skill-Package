# Load Methods and Error-Related Signatures

## PDFDocument.load() — Complete Signature

```typescript
static async load(
  pdf: string | Uint8Array | ArrayBuffer,
  options?: LoadOptions
): Promise<PDFDocument>
```

### Parameter: pdf

| Type | Description | Example |
|------|-------------|---------|
| `Uint8Array` | Raw PDF file bytes | `fs.readFileSync('file.pdf')` |
| `ArrayBuffer` | Raw PDF file bytes (browser) | `await response.arrayBuffer()` |
| `string` | Base64-encoded PDF content | `"JVBERi0xLjcK..."` |
| `string` | Data URI with base64 PDF | `"data:application/pdf;base64,JVBERi0..."` |

NEVER pass a file path or URL string. ALWAYS read the file into bytes first.

### Parameter: options (LoadOptions)

```typescript
interface LoadOptions {
  ignoreEncryption?: boolean;      // Default: false
  throwOnInvalidObject?: boolean;  // Default: false
  updateMetadata?: boolean;        // Default: true
  capNumbers?: boolean;            // Default: false
  parseSpeed?: number;             // Default: undefined
}
```

## LoadOptions — Detailed Behavior

### ignoreEncryption

```typescript
ignoreEncryption?: boolean  // Default: false
```

- When `false`: throws an error if the PDF contains an encryption dictionary
- When `true`: skips the encryption validation check and attempts to load
- Does NOT decrypt content — only bypasses the encryption check
- Use for owner-password (restriction) PDFs where content is not actually encrypted
- NEVER rely on this for user-password (content-encrypted) PDFs

### throwOnInvalidObject

```typescript
throwOnInvalidObject?: boolean  // Default: false
```

- When `false`: silently skips malformed PDF objects during parsing
- When `true`: throws an error when any invalid PDF object is encountered
- Use `true` for debugging to identify which objects are corrupt
- Use `false` (default) for maximum compatibility with real-world PDFs

### updateMetadata

```typescript
updateMetadata?: boolean  // Default: true
```

- When `true`: AUTOMATICALLY overwrites these fields on load:
  - `producer` -> `"pdf-lib (https://github.com/Hopding/pdf-lib)"`
  - `creator` -> `"pdf-lib (https://github.com/Hopding/pdf-lib)"`
  - `creationDate` -> current Date
  - `modificationDate` -> current Date
- When `false`: ALL original metadata fields are preserved unchanged
- ALWAYS use `false` when metadata preservation is required

### capNumbers

```typescript
capNumbers?: boolean  // Default: false
```

- When `false`: numeric values in the PDF are parsed as-is
- When `true`: limits numeric precision to prevent JavaScript overflow
- Use when PDFs contain extreme floating-point values causing NaN or Infinity

### parseSpeed

```typescript
parseSpeed?: number  // Default: undefined
```

- Controls parsing performance level
- Higher values may skip validation steps for faster loading

## PDFDocument.isEncrypted Property

```typescript
readonly isEncrypted: boolean
```

- Returns `true` if the document contains an encryption dictionary
- Available after successful load (including with `ignoreEncryption: true`)
- Does NOT indicate whether content is actually encrypted vs. restriction-only
- Use as a diagnostic flag after loading

## PDFDocument.create() — Related Options

```typescript
static async create(options?: CreateOptions): Promise<PDFDocument>

interface CreateOptions {
  updateMetadata?: boolean;  // Default: true
}
```

- Same `updateMetadata` behavior as `load()`
- When `true`: sets producer, creator, and dates on the new empty document
- When `false`: creates document with no metadata fields set

## Error Types Thrown by load()

| Error Condition | Thrown When |
|----------------|------------|
| Encryption error | PDF has encryption dictionary and `ignoreEncryption` is `false` |
| Invalid object error | PDF has malformed objects and `throwOnInvalidObject` is `true` |
| Parse error | Input bytes are not valid PDF format |
| Type error | Input is not `string`, `Uint8Array`, or `ArrayBuffer` |

## Method Chaining Pattern

```typescript
// Load and immediately check state
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
  updateMetadata: false,
});

const isEnc: boolean = pdfDoc.isEncrypted;
const pageCount: number = pdfDoc.getPageCount();
const title: string | undefined = pdfDoc.getTitle();
const producer: string | undefined = pdfDoc.getProducer();
```
