---
name: pdflib-errors-loading
description: "Diagnoses and fixes PDF loading errors in pdf-lib including encrypted PDF handling, malformed PDF recovery, ignoreEncryption usage, and load option configuration. Activates when encountering PDF loading errors, encrypted PDF errors, or malformed PDF parsing failures."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdflib-errors-loading

Diagnoses and fixes all errors related to `PDFDocument.load()` in pdf-lib 1.x. Covers encrypted PDFs, malformed documents, load option misconfiguration, and metadata side effects.

## Quick Reference

```typescript
// Standard load
const pdfDoc = await PDFDocument.load(pdfBytes);

// Load encrypted PDF (bypasses encryption check, does NOT decrypt)
const pdfDoc = await PDFDocument.load(pdfBytes, { ignoreEncryption: true });

// Load malformed PDF (strict mode — throws on invalid objects)
const pdfDoc = await PDFDocument.load(pdfBytes, { throwOnInvalidObject: true });

// Load without modifying metadata
const pdfDoc = await PDFDocument.load(pdfBytes, { updateMetadata: false });

// Load with numeric precision capping
const pdfDoc = await PDFDocument.load(pdfBytes, { capNumbers: true });

// Check encryption status after load
const isEnc: boolean = pdfDoc.isEncrypted;
```

## LoadOptions Reference

| Option | Type | Default | Purpose |
|--------|------|---------|---------|
| `ignoreEncryption` | `boolean` | `false` | Skip encryption validation on load |
| `throwOnInvalidObject` | `boolean` | `false` | Throw on malformed PDF objects instead of silently skipping |
| `updateMetadata` | `boolean` | `true` | Auto-set producer, creator, creation/modification dates |
| `capNumbers` | `boolean` | `false` | Limit numeric precision to avoid floating-point issues |
| `parseSpeed` | `number` | — | Control parsing performance level |

## Decision Tree: Loading Error Diagnosis

```
PDFDocument.load() throws an error
|
+-- Error mentions "encrypted" or "encryption"
|   +-- PDF has owner password (print/copy restrictions)?
|   |   +-- Use { ignoreEncryption: true }
|   |   +-- This bypasses the restriction CHECK, not the encryption itself
|   |   +-- Content is accessible because owner-password PDFs store content unencrypted
|   |
|   +-- PDF has user password (content is actually encrypted)?
|       +-- pdf-lib CANNOT decrypt this PDF
|       +-- NEVER use ignoreEncryption hoping it will decrypt content
|       +-- Pre-process with external tool: qpdf, mutool, or pdftk
|       +-- Then load the decrypted output with pdf-lib
|
+-- Error mentions "invalid object" or parsing failure
|   +-- PDF is malformed or corrupted?
|   |   +-- Try { throwOnInvalidObject: false } (default) to skip bad objects
|   |   +-- If that works: document loads but may have missing content
|   |   +-- If that fails: PDF is too corrupted for pdf-lib
|   |
|   +-- Need strict validation?
|       +-- Use { throwOnInvalidObject: true }
|       +-- Catch the error to identify which objects are invalid
|
+-- Error mentions unexpected token or format
|   +-- Input is not valid PDF bytes?
|   |   +-- Verify input is Uint8Array, ArrayBuffer, or base64 string
|   |   +-- Verify the file starts with %PDF header
|   |   +-- NEVER pass a file path string — pdf-lib does not read files
|   |
|   +-- Input is a file path or URL?
|       +-- Read the file first: fs.readFileSync() or fetch().arrayBuffer()
|       +-- Then pass the resulting bytes to PDFDocument.load()
|
+-- Metadata unexpectedly changed after load?
    +-- Default behavior: updateMetadata is true
    +-- Use { updateMetadata: false } to preserve original metadata
    +-- See "updateMetadata Side Effects" section below
```

## Error: Encrypted PDF

### Cause
pdf-lib throws when loading a PDF that contains encryption dictionary markers. This happens with BOTH owner-password (restriction-only) and user-password (content-encrypted) PDFs.

### The Critical Distinction

**Owner-password PDFs** (restrictions like no-print, no-copy):
- Content is stored UNENCRYPTED in the file
- The encryption dictionary only signals viewer restrictions
- `ignoreEncryption: true` works correctly — it skips the restriction check
- You can read, modify, and save the document normally

**User-password PDFs** (content is actually encrypted):
- Content bytes are encrypted and unreadable without the password
- `ignoreEncryption: true` skips the encryption CHECK but does NOT decrypt content
- Loading succeeds but content operations produce garbage or errors
- pdf-lib has NO decryption engine and NO password support

### Fix for Owner-Password PDFs

```typescript
import { PDFDocument } from 'pdf-lib';

const pdfBytes = fs.readFileSync('restricted.pdf');
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
});

// ALWAYS verify content is accessible after loading
const pages = pdfDoc.getPages();
console.log(`Loaded ${pages.length} pages`);

// Check encryption flag
if (pdfDoc.isEncrypted) {
  console.log('Document has encryption markers');
}
```

### Fix for User-Password PDFs

```typescript
// STEP 1: Decrypt with external tool (example with qpdf CLI)
// qpdf --password=secret --decrypt input.pdf decrypted.pdf

// STEP 2: Load the decrypted file with pdf-lib
import { PDFDocument } from 'pdf-lib';

const decryptedBytes = fs.readFileSync('decrypted.pdf');
const pdfDoc = await PDFDocument.load(decryptedBytes);
// Now content is accessible normally
```

### NEVER Do This

```typescript
// ANTI-PATTERN: Assuming ignoreEncryption decrypts content
const pdfDoc = await PDFDocument.load(encryptedBytes, {
  ignoreEncryption: true,
});
// If PDF has user-password encryption, content is STILL encrypted
// Text extraction, form filling, and page operations will fail or produce garbage
```

## Error: Malformed PDF Objects

### Cause
The PDF file contains objects that do not conform to PDF specification. This happens with PDFs generated by buggy software, corrupted during transfer, or hand-edited.

### Fix: Lenient Mode (Default)

```typescript
// Default behavior: silently skip invalid objects
const pdfDoc = await PDFDocument.load(corruptedBytes);
// Document loads but may have missing content where invalid objects were
```

### Fix: Strict Mode for Debugging

```typescript
try {
  const pdfDoc = await PDFDocument.load(corruptedBytes, {
    throwOnInvalidObject: true,
  });
} catch (error) {
  console.error('Invalid PDF object found:', error.message);
  // Use this information to identify the corruption
  // Then load again without throwOnInvalidObject to work around it
}
```

## updateMetadata Side Effects

### The Problem
By default, `PDFDocument.load()` sets `updateMetadata: true`. This AUTOMATICALLY overwrites four metadata fields:

| Field | Overwritten Value |
|-------|-------------------|
| `producer` | `"pdf-lib (https://github.com/Hopding/pdf-lib)"` |
| `creator` | `"pdf-lib (https://github.com/Hopding/pdf-lib)"` |
| `creationDate` | Current timestamp |
| `modificationDate` | Current timestamp |

### When This Causes Problems
- Archival workflows that must preserve original metadata
- Legal documents where creation date must remain unchanged
- Document audit trails that track producer software
- Round-trip editing where metadata must stay intact

### Fix: Preserve Original Metadata

```typescript
const pdfDoc = await PDFDocument.load(pdfBytes, {
  updateMetadata: false,
});
// All original metadata fields are preserved exactly as they were
```

### Fix: Selective Metadata Control

```typescript
// Load without auto-update, then set only what you need
const pdfDoc = await PDFDocument.load(pdfBytes, {
  updateMetadata: false,
});
// Update only modification date, keep everything else
pdfDoc.setModificationDate(new Date());
```

## Restricted Document Handling for Form Filling

### The Problem
Some PDFs have owner-password restrictions that specifically prohibit form filling. pdf-lib throws an encryption error when loading these documents, even though the form data itself is not encrypted.

### The Fix

```typescript
import { PDFDocument } from 'pdf-lib';

// Load with ignoreEncryption to bypass restriction check
const pdfDoc = await PDFDocument.load(restrictedFormBytes, {
  ignoreEncryption: true,
});

// Form filling works normally after bypassing restrictions
const form = pdfDoc.getForm();
form.getTextField('name').setText('John Doe');

const pdfBytes = await pdfDoc.save();
```

### CRITICAL: This works because owner-password restrictions are viewer-level enforcements, not content-level encryption. The form field data is stored unencrypted.

## capNumbers Option

### When to Use
Use `capNumbers: true` when loading PDFs that contain extremely large or precise floating-point numbers that cause JavaScript numeric overflow or precision issues.

```typescript
const pdfDoc = await PDFDocument.load(pdfBytes, {
  capNumbers: true,
});
```

### Symptoms That Indicate You Need capNumbers
- `NaN` or `Infinity` values appearing in page dimensions
- Rendering artifacts from extreme coordinate values
- JavaScript numeric overflow errors during page operations

## parseSpeed Option

### Purpose
Controls the parsing performance level during document loading. Higher values may skip certain validation steps for faster loading of large documents.

```typescript
const pdfDoc = await PDFDocument.load(largePdfBytes, {
  parseSpeed: 150,
});
```

## isEncrypted Property

```typescript
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
});

if (pdfDoc.isEncrypted) {
  // Document has encryption dictionary
  // With ignoreEncryption: true, this loads successfully
  // but encrypted content (user-password) is NOT decrypted
}
```

ALWAYS check `isEncrypted` after loading with `ignoreEncryption: true` to confirm the document's encryption status. Use this as a diagnostic flag, not a guarantee that content is accessible.

## Input Validation Errors

### Valid Input Types for PDFDocument.load()

| Type | Example |
|------|---------|
| `Uint8Array` | `fs.readFileSync('file.pdf')` in Node.js |
| `ArrayBuffer` | `await fetch(url).then(r => r.arrayBuffer())` |
| `string` (base64) | Base64-encoded PDF content |
| `string` (data URI) | `"data:application/pdf;base64,..."` |

### NEVER Pass These

```typescript
// WRONG: File path string
await PDFDocument.load('/path/to/file.pdf'); // NOT a valid PDF

// WRONG: URL string
await PDFDocument.load('https://example.com/file.pdf'); // NOT base64

// WRONG: Plain text
await PDFDocument.load('Hello World'); // NOT PDF bytes

// CORRECT: Read file first, then pass bytes
const bytes = fs.readFileSync('/path/to/file.pdf');
await PDFDocument.load(bytes);
```

## Combined Options Pattern

For maximum compatibility when loading unknown PDFs:

```typescript
const pdfDoc = await PDFDocument.load(unknownPdfBytes, {
  ignoreEncryption: true,
  throwOnInvalidObject: false,  // default, but explicit
  updateMetadata: false,        // preserve original metadata
  capNumbers: true,             // handle extreme numbers
});
```

ALWAYS start with the most lenient options when debugging loading failures, then tighten options once the root cause is identified.

## Reference Files

- [references/methods.md](references/methods.md) — Load method signatures and option types
- [references/examples.md](references/examples.md) — Error reproduction and fix examples
- [references/anti-patterns.md](references/anti-patterns.md) — Loading anti-patterns to avoid
