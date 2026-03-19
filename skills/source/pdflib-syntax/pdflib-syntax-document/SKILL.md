---
name: pdflib-syntax-document
description: >
  Use when creating, loading, or saving PDFs with pdf-lib, or setting PDF metadata.
  Prevents common mistakes: missing load options for encrypted PDFs, forgetting
  saveAsBase64 for browser contexts, and metadata not persisting due to wrong setter order.
  Covers PDFDocument.create(), load(), save(), metadata, attachments.
  Keywords: PDFDocument, create, load, save, saveAsBase64, metadata, title, author, encrypt.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-syntax-document

## Quick Reference

### PDFDocument Lifecycle Methods

| Method | Returns | Async | Purpose |
|--------|---------|-------|---------|
| `PDFDocument.create(options?)` | `Promise<PDFDocument>` | Yes | Create a new empty PDF |
| `PDFDocument.load(pdf, options?)` | `Promise<PDFDocument>` | Yes | Load an existing PDF from bytes/base64 |
| `pdfDoc.save(options?)` | `Promise<Uint8Array>` | Yes | Serialize to binary |
| `pdfDoc.saveAsBase64(options?)` | `Promise<string>` | Yes | Serialize to base64 string |
| `pdfDoc.copy()` | `Promise<PDFDocument>` | Yes | Shallow copy (no forms/outlines) |
| `pdfDoc.flush()` | `Promise<void>` | Yes | Flush embedded resources (auto-called by save) |

### Metadata Methods

| Getter | Setter | Type |
|--------|--------|------|
| `getTitle()` | `setTitle(title, options?)` | `string` |
| `getAuthor()` | `setAuthor(author)` | `string` |
| `getSubject()` | `setSubject(subject)` | `string` |
| `getKeywords()` | `setKeywords(keywords)` | getter: `string`, setter: `string[]` |
| `getCreator()` | `setCreator(creator)` | `string` |
| `getProducer()` | `setProducer(producer)` | `string` |
| `getCreationDate()` | `setCreationDate(date)` | `Date` |
| `getModificationDate()` | `setModificationDate(date)` | `Date` |
| -- | `setLanguage(language)` | `string` (RFC 3066) |

### Additional Document Methods

| Method | Returns | Async | Purpose |
|--------|---------|-------|---------|
| `pdfDoc.attach(bytes, name, options?)` | `Promise<void>` | Yes | Attach a file to the PDF |
| `pdfDoc.addJavaScript(name, script)` | `void` | No | Add named JavaScript action |

### PDFDocument Properties

| Property | Type | Description |
|----------|------|-------------|
| `catalog` | `PDFCatalog` | The document catalog |
| `context` | `PDFContext` | Low-level document context |
| `defaultWordBreaks` | `string[]` | Word break characters (default: `[' ']`) |
| `isEncrypted` | `boolean` | Whether document is encrypted |

### Critical Warnings

**NEVER** forget to `await` PDFDocument.create(), load(), save(), or saveAsBase64() -- they ALL return Promises. Passing a Promise instead of a resolved value causes silent failures or runtime errors.

**NEVER** assume `copy()` produces a full clone -- it does NOT copy AcroForm fields, outlines, or other catalog-level structures. ALWAYS use `copyPages()` for page-level operations instead.

**NEVER** rely on default `updateMetadata: true` when loading existing PDFs for archival -- it silently overwrites the original producer, creator, and modification date. ALWAYS pass `{ updateMetadata: false }` to preserve original metadata.

**NEVER** set keywords with `setKeywords()` and expect `getKeywords()` to return the same type -- `setKeywords()` takes `string[]` but `getKeywords()` returns `string | undefined` (comma-separated).

---

## Decision Trees

### Create vs Load

```
Need a PDFDocument?
+-- Starting from scratch?
|   +-- Need default metadata? -> await PDFDocument.create()
|   +-- Preserving existing metadata pipeline? -> await PDFDocument.create({ updateMetadata: false })
+-- Working with existing PDF?
    +-- From file bytes (Uint8Array/ArrayBuffer)? -> await PDFDocument.load(bytes)
    +-- From base64 string? -> await PDFDocument.load(base64String)
    +-- PDF is encrypted? -> await PDFDocument.load(bytes, { ignoreEncryption: true })
    +-- Must preserve original metadata? -> await PDFDocument.load(bytes, { updateMetadata: false })
```

### Save Format Selection

```
Need to output the PDF?
+-- Writing to filesystem (Node.js)?
|   +-- await pdfDoc.save() -> returns Uint8Array -> fs.writeFileSync('out.pdf', bytes)
+-- Sending as HTTP response?
|   +-- Binary response -> await pdfDoc.save()
|   +-- Base64 in JSON -> await pdfDoc.saveAsBase64()
+-- Embedding in HTML (iframe/embed)?
|   +-- await pdfDoc.saveAsBase64({ dataUri: true }) -> set as src attribute
+-- Need smaller file size?
|   +-- await pdfDoc.save({ useObjectStreams: true })  // default
+-- Need maximum viewer compatibility?
    +-- await pdfDoc.save({ useObjectStreams: false })
```

### Metadata Strategy

```
Setting document metadata?
+-- New document for distribution?
|   +-- Set title with showInWindowTitleBar: true
|   +-- Set author, subject, keywords for searchability
|   +-- Set language for accessibility (RFC 3066 tag)
+-- Loading existing PDF for modification?
|   +-- Want to keep original metadata? -> load with { updateMetadata: false }
|   +-- Want fresh metadata? -> load with defaults (updateMetadata: true)
+-- Archival / compliance requirement?
    +-- ALWAYS set updateMetadata: false on load
    +-- Explicitly set only the fields you need to change
    +-- Set modificationDate to current time
```

---

## Essential Patterns

### Pattern 1: Create, Populate, Save

```typescript
import { PDFDocument, StandardFonts, rgb } from 'pdf-lib';

const pdfDoc = await PDFDocument.create();
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
const page = pdfDoc.addPage();

page.drawText('Hello World', {
  x: 50,
  y: page.getHeight() - 50,
  size: 24,
  font,
  color: rgb(0, 0, 0),
});

const pdfBytes = await pdfDoc.save();
```

### Pattern 2: Load, Modify, Save with Metadata Preservation

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  updateMetadata: false,  // preserve original metadata
});

// Make modifications...
pdfDoc.setModificationDate(new Date());  // only update what changed

const pdfBytes = await pdfDoc.save();
```

### Pattern 3: Full Metadata Setup

```typescript
const pdfDoc = await PDFDocument.create();

pdfDoc.setTitle('Annual Report 2024', { showInWindowTitleBar: true });
pdfDoc.setAuthor('Finance Department');
pdfDoc.setSubject('Q4 Financial Summary');
pdfDoc.setKeywords(['finance', 'quarterly', 'report']);
pdfDoc.setCreator('MyApp v2.0');
pdfDoc.setProducer('pdf-lib');
pdfDoc.setLanguage('en-US');
pdfDoc.setCreationDate(new Date());
pdfDoc.setModificationDate(new Date());
```

### Pattern 4: Attach Files to PDF

```typescript
const pdfDoc = await PDFDocument.create();
// ... add pages and content ...

await pdfDoc.attach(csvBytes, 'data.csv', {
  mimeType: 'text/csv',
  description: 'Raw data export',
  creationDate: new Date(),
  modificationDate: new Date(),
});

const pdfBytes = await pdfDoc.save();
```

### Pattern 5: Save as Data URI for Browser Display

```typescript
const pdfDoc = await PDFDocument.create();
// ... add content ...

const dataUri = await pdfDoc.saveAsBase64({ dataUri: true });
// Result: "data:application/pdf;base64,JVBERi0xLj..."

// Use in HTML
const iframe = document.createElement('iframe');
iframe.src = dataUri;
document.body.appendChild(iframe);
```

### Pattern 6: Load Encrypted PDF

```typescript
const pdfDoc = await PDFDocument.load(encryptedPdfBytes, {
  ignoreEncryption: true,
});

// Check encryption status
if (pdfDoc.isEncrypted) {
  console.log('Document has encryption markers');
}

// Note: pdf-lib cannot decrypt password-protected content
// or encrypt output PDFs
```

---

## Reference Links

- [references/methods.md](references/methods.md) -- Complete PDFDocument API with all method signatures, parameter types, and option interfaces
- [references/examples.md](references/examples.md) -- Working code examples for every PDFDocument operation
- [references/anti-patterns.md](references/anti-patterns.md) -- Common mistakes with PDFDocument and how to avoid them

### Official Sources

- https://pdf-lib.js.org/docs/api/classes/pdfdocument
- https://github.com/Hopding/pdf-lib
- https://pdf-lib.js.org/
