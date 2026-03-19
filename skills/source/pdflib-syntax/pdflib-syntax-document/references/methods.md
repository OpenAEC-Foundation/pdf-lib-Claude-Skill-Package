# PDFDocument API Reference

## Static Methods

### PDFDocument.create()

```typescript
static async create(options?: CreateOptions): Promise<PDFDocument>
```

Creates a new, empty PDF document.

**CreateOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, creation/modification dates |

When `updateMetadata` is `true` (default), the library automatically sets:
- `producer` to `"pdf-lib (https://github.com/Hopding/pdf-lib)"`
- `creator` to `"pdf-lib (https://github.com/Hopding/pdf-lib)"`
- `creationDate` to current time
- `modificationDate` to current time

---

### PDFDocument.load()

```typescript
static async load(
  pdf: string | Uint8Array | ArrayBuffer,
  options?: LoadOptions
): Promise<PDFDocument>
```

Loads an existing PDF document from binary data or base64.

**Input types:**
- `string` -- Base64-encoded string or data URI (`data:application/pdf;base64,...`)
- `Uint8Array` -- Raw PDF bytes
- `ArrayBuffer` -- Raw PDF bytes

**LoadOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ignoreEncryption` | `boolean` | `false` | Skip encryption validation; allows loading PDFs with encryption markers |
| `parseSpeed` | `number` | -- | Parsing performance level |
| `throwOnInvalidObject` | `boolean` | `false` | Throw error on malformed PDF objects instead of skipping them |
| `updateMetadata` | `boolean` | `true` | Auto-update producer, modification date, creator, creation date |
| `capNumbers` | `boolean` | `false` | Limit numeric precision to avoid floating-point edge cases |

**CRITICAL**: When `updateMetadata` is `true` (default), loading a PDF silently overwrites:
- The original `producer` field
- The original `creator` field
- The `creationDate` (reset to now)
- The `modificationDate` (reset to now)

ALWAYS pass `{ updateMetadata: false }` when preserving original document metadata matters.

---

## Instance Methods -- Save & Export

### pdfDoc.save()

```typescript
save(options?: SaveOptions): Promise<Uint8Array>
```

Serializes the document to a PDF byte array.

**SaveOptions:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `useObjectStreams` | `boolean` | `true` | Enable object stream compression (smaller files, requires PDF 1.5+ viewers) |
| `addDefaultPage` | `boolean` | `true` | Insert a blank page if the document has zero pages |
| `objectsPerTick` | `number` | `50` | Number of PDF objects processed per event loop tick (higher = faster but blocks longer) |
| `updateFieldAppearances` | `boolean` | `true` | Refresh visual rendering of form fields before saving |

**When to disable `useObjectStreams`:**
- Target viewers do not support PDF 1.5+
- Maximum compatibility with older PDF readers is required
- Debugging raw PDF structure (uncompressed is easier to inspect)

**When to disable `updateFieldAppearances`:**
- Performance optimization when form fields are not modified
- Using custom appearance providers that handle updates separately

---

### pdfDoc.saveAsBase64()

```typescript
saveAsBase64(options?: Base64SaveOptions): Promise<string>
```

Serializes the document to a base64-encoded string.

**Base64SaveOptions** extends SaveOptions with:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dataUri` | `boolean` | `false` | Prefix with `data:application/pdf;base64,` for direct use in HTML src attributes |

All SaveOptions properties are also available.

---

### pdfDoc.copy()

```typescript
copy(): Promise<PDFDocument>
```

Creates a shallow copy of the document.

**Limitations -- CRITICAL:**
- Does NOT copy AcroForm fields (interactive forms)
- Does NOT copy document outlines (bookmarks)
- Does NOT copy other catalog-level structures
- Copies page tree and basic document structure only

For page-level operations, ALWAYS use `copyPages()` instead.

---

### pdfDoc.flush()

```typescript
flush(): Promise<void>
```

Flushes embedded fonts, pages, and images to the document context. ALWAYS called automatically by `save()` and `saveAsBase64()`. Manual calls are rarely needed.

---

## Instance Methods -- Metadata

### Getters

All metadata getters return `undefined` if the field has not been set.

```typescript
getTitle(): string | undefined
getAuthor(): string | undefined
getSubject(): string | undefined
getKeywords(): string | undefined    // Returns comma-separated string, NOT string[]
getCreator(): string | undefined
getProducer(): string | undefined
getCreationDate(): Date | undefined
getModificationDate(): Date | undefined
```

**IMPORTANT**: `getKeywords()` returns a single comma-separated `string`, NOT the `string[]` that `setKeywords()` accepts. This asymmetry is by design in pdf-lib.

### Setters

```typescript
setTitle(title: string, options?: SetTitleOptions): void
setAuthor(author: string): void
setSubject(subject: string): void
setKeywords(keywords: string[]): void    // Accepts string[] array
setCreator(creator: string): void
setProducer(producer: string): void
setCreationDate(creationDate: Date): void
setModificationDate(modificationDate: Date): void
setLanguage(language: string): void      // RFC 3066 language tags (e.g., 'en-US', 'nl-NL', 'de-DE')
```

**SetTitleOptions:**

| Property | Type | Description |
|----------|------|-------------|
| `showInWindowTitleBar` | `boolean` | When `true`, PDF viewers display the document title in the window title bar instead of the filename |

---

## Instance Methods -- Attachments

### pdfDoc.attach()

```typescript
async attach(
  attachment: string | Uint8Array | ArrayBuffer,
  name: string,
  options?: AttachmentOptions
): Promise<void>
```

Attaches a file to the PDF document.

**Input types for `attachment`:**
- `string` -- Base64-encoded file data
- `Uint8Array` -- Raw file bytes
- `ArrayBuffer` -- Raw file bytes

**AttachmentOptions:**

| Property | Type | Description |
|----------|------|-------------|
| `mimeType` | `string` | MIME type of the attachment (e.g., `'text/csv'`, `'application/json'`) |
| `description` | `string` | Human-readable description shown in PDF viewers |
| `creationDate` | `Date` | Creation timestamp for the attachment |
| `modificationDate` | `Date` | Last modification timestamp for the attachment |

**Viewer support**: Only some PDF readers display attachments -- Adobe Acrobat Reader, Foxit Reader, and Firefox. Other viewers may silently ignore attachments.

---

## Instance Methods -- JavaScript

### pdfDoc.addJavaScript()

```typescript
addJavaScript(name: string, script: string): void
```

Adds a named JavaScript action that executes when the document is opened.

- The `name` parameter MUST be unique within the document
- JavaScript runs in the PDF viewer's scripting engine (Adobe Acrobat JavaScript API)
- NOT all PDF viewers support JavaScript -- Adobe Acrobat does, most others do not

---

## Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `catalog` | `PDFCatalog` | The document's root catalog dictionary; provides low-level access to the PDF structure |
| `context` | `PDFContext` | The document's object context; used for advanced low-level PDF manipulation |
| `defaultWordBreaks` | `string[]` | Characters used for word wrapping in `drawText()` with `maxWidth`; defaults to `[' ']` |
| `isEncrypted` | `boolean` | `true` if the document has encryption markers; does NOT mean pdf-lib can decrypt it |

---

## Option Interfaces Summary

| Interface | Used By | Key Properties |
|-----------|---------|----------------|
| `CreateOptions` | `PDFDocument.create()` | `updateMetadata` |
| `LoadOptions` | `PDFDocument.load()` | `ignoreEncryption`, `parseSpeed`, `throwOnInvalidObject`, `updateMetadata`, `capNumbers` |
| `SaveOptions` | `pdfDoc.save()` | `useObjectStreams`, `addDefaultPage`, `objectsPerTick`, `updateFieldAppearances` |
| `Base64SaveOptions` | `pdfDoc.saveAsBase64()` | All SaveOptions + `dataUri` |
| `SetTitleOptions` | `pdfDoc.setTitle()` | `showInWindowTitleBar` |
| `AttachmentOptions` | `pdfDoc.attach()` | `mimeType`, `description`, `creationDate`, `modificationDate` |
