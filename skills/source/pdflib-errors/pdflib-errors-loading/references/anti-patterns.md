# Loading Anti-Patterns

## Anti-Pattern 1: Assuming ignoreEncryption Decrypts Content

**NEVER assume `ignoreEncryption: true` makes encrypted content readable.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load(passwordProtectedBytes, {
  ignoreEncryption: true,
});
// Developer assumes content is now decrypted — it is NOT
const form = pdfDoc.getForm();
form.getTextField('name').setText('John'); // May produce garbage output
```

**Why it fails**: `ignoreEncryption` only skips the encryption CHECK during loading. It does NOT decrypt the actual PDF content. For user-password PDFs, the content bytes remain encrypted and unusable.

**Correct approach**: Pre-process with an external decryption tool (qpdf, mutool, pdftk), then load the decrypted output with pdf-lib.

---

## Anti-Pattern 2: Loading with Default updateMetadata for Archival

**NEVER use default `updateMetadata: true` when metadata preservation matters.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load(archivalPdfBytes);
// Original producer, creator, and dates are SILENTLY OVERWRITTEN
const savedBytes = await pdfDoc.save();
// Output has pdf-lib as producer and current timestamp as creation date
```

**Why it fails**: The default `updateMetadata: true` overwrites producer, creator, creationDate, and modificationDate without warning. This destroys audit trails and violates archival requirements.

**Correct approach**: ALWAYS use `{ updateMetadata: false }` when you need to preserve original metadata.

---

## Anti-Pattern 3: Passing File Paths to PDFDocument.load()

**NEVER pass a file path string to `PDFDocument.load()`.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load('C:\\Documents\\report.pdf');
// Throws: the string is not valid PDF content

// ANTI-PATTERN
const pdfDoc = await PDFDocument.load('./report.pdf');
// Throws: the string is not valid PDF content
```

**Why it fails**: `PDFDocument.load()` expects PDF content (bytes or base64), not a file system path. The path string is interpreted as base64-encoded content, which fails parsing.

**Correct approach**: Read the file into bytes first, then pass the bytes.

```typescript
import fs from 'fs';
const pdfBytes = fs.readFileSync('report.pdf');
const pdfDoc = await PDFDocument.load(pdfBytes);
```

---

## Anti-Pattern 4: Using throwOnInvalidObject in Production

**NEVER use `throwOnInvalidObject: true` in production code that handles user-uploaded PDFs.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load(userUploadedBytes, {
  throwOnInvalidObject: true,
});
// Crashes on any slightly malformed PDF — many real-world PDFs have minor issues
```

**Why it fails**: Real-world PDFs from various generators (scanners, web tools, legacy software) frequently contain minor specification violations. Strict mode rejects these documents unnecessarily.

**Correct approach**: Use `throwOnInvalidObject: true` ONLY for debugging and validation. Use the default `false` in production.

```typescript
// Production: lenient
const pdfDoc = await PDFDocument.load(userUploadedBytes);

// Debugging: strict
try {
  await PDFDocument.load(userUploadedBytes, { throwOnInvalidObject: true });
} catch (e) {
  console.error('PDF validation issue:', e.message);
}
```

---

## Anti-Pattern 5: No Error Handling on Load

**NEVER call `PDFDocument.load()` without error handling.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load(pdfBytes);
// If loading fails, the error propagates unhandled
```

**Why it fails**: Loading can fail for many reasons — encryption, corruption, invalid input, memory issues. Without error handling, the application crashes with an unhelpful error.

**Correct approach**: ALWAYS wrap `PDFDocument.load()` in try-catch.

```typescript
try {
  const pdfDoc = await PDFDocument.load(pdfBytes);
} catch (error) {
  const message = (error as Error).message;
  if (message.includes('encrypt')) {
    // Handle encryption error
  } else if (message.includes('invalid')) {
    // Handle malformed PDF
  } else {
    // Handle other loading errors
  }
}
```

---

## Anti-Pattern 6: Ignoring isEncrypted After Load

**NEVER skip the `isEncrypted` check after loading with `ignoreEncryption: true`.**

```typescript
// ANTI-PATTERN
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
});
// Proceeds without checking if content is actually accessible
const form = pdfDoc.getForm();
form.getTextField('field').setText('value');
// May silently produce corrupt output if PDF had user-password encryption
```

**Correct approach**: ALWAYS check `isEncrypted` and verify content accessibility.

```typescript
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
});

if (pdfDoc.isEncrypted) {
  console.warn('Document has encryption markers — verify content integrity');
  // Test a basic operation to confirm content is accessible
  const pageCount = pdfDoc.getPageCount();
  if (pageCount === 0) {
    throw new Error('Encrypted content is not accessible — decrypt with external tool first');
  }
}
```

---

## Anti-Pattern 7: Loading Base64 with Data URI Prefix Inconsistency

**NEVER mix up base64 string formats.**

```typescript
// ANTI-PATTERN: Double-wrapping data URI
const base64 = 'data:application/pdf;base64,JVBERi0xLjcK...';
const dataUri = `data:application/pdf;base64,${base64}`;
// The prefix is now doubled — load will fail

// ANTI-PATTERN: Stripping prefix when not needed
const dataUri = 'data:application/pdf;base64,JVBERi0xLjcK...';
const stripped = dataUri.replace('data:application/pdf;base64,', '');
// Unnecessary — PDFDocument.load() handles both formats
```

**Correct approach**: Pass either pure base64 OR a complete data URI. pdf-lib accepts both.

```typescript
// Both work correctly:
const pdfDoc1 = await PDFDocument.load('JVBERi0xLjcK...');  // Pure base64
const pdfDoc2 = await PDFDocument.load('data:application/pdf;base64,JVBERi0xLjcK...');  // Data URI
```

---

## Anti-Pattern 8: Not Preserving Metadata When Round-Tripping

**NEVER load-modify-save without considering metadata when the PDF will be compared or audited.**

```typescript
// ANTI-PATTERN: Load -> modify -> save with default options
const pdfDoc = await PDFDocument.load(originalBytes);
// Metadata is now changed even if you only wanted to add a watermark
page.drawText('DRAFT', { x: 200, y: 400, size: 48 });
const outputBytes = await pdfDoc.save();
// Output has different producer, creator, and dates than original
```

**Correct approach**: Use `updateMetadata: false` and only update what you intentionally want to change.

```typescript
const pdfDoc = await PDFDocument.load(originalBytes, {
  updateMetadata: false,
});
page.drawText('DRAFT', { x: 200, y: 400, size: 48 });
pdfDoc.setModificationDate(new Date()); // Only update mod date
const outputBytes = await pdfDoc.save();
```

---

## Summary Table

| Anti-Pattern | Risk | Fix |
|-------------|------|-----|
| Assuming ignoreEncryption decrypts | Corrupt output from encrypted content | Pre-decrypt with external tool |
| Default updateMetadata for archival | Silent metadata destruction | Use `{ updateMetadata: false }` |
| Passing file paths to load() | Immediate parse error | Read file to bytes first |
| throwOnInvalidObject in production | Rejects valid real-world PDFs | Use `false` in production, `true` for debugging |
| No error handling on load | Unhandled crashes | Wrap in try-catch with specific error handling |
| Ignoring isEncrypted after load | Silent corruption | Check isEncrypted and verify content |
| Base64 format confusion | Parse failure | Pass pure base64 or complete data URI |
| Not preserving metadata on round-trip | Audit trail destroyed | Use `{ updateMetadata: false }` |
