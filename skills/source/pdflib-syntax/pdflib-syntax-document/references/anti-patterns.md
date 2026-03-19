# PDFDocument Anti-Patterns

## AP-001: Forgetting to Await Async Methods

**Severity**: Critical -- causes runtime errors or silent failures

```typescript
// WRONG -- create() returns a Promise, not a PDFDocument
const pdfDoc = PDFDocument.create();
pdfDoc.addPage();  // TypeError: pdfDoc.addPage is not a function

// WRONG -- save() returns a Promise, not Uint8Array
const bytes = pdfDoc.save();
fs.writeFileSync('out.pdf', bytes);  // Writes "[object Promise]" to file

// CORRECT -- ALWAYS await async methods
const pdfDoc = await PDFDocument.create();
const bytes = await pdfDoc.save();
fs.writeFileSync('out.pdf', bytes);
```

**Rule**: ALWAYS `await` these methods: `create()`, `load()`, `save()`, `saveAsBase64()`, `copy()`, `flush()`, `attach()`.

---

## AP-002: Metadata Overwritten on Load

**Severity**: High -- silently destroys original metadata

```typescript
// WRONG -- default updateMetadata: true overwrites original metadata
const pdfDoc = await PDFDocument.load(existingPdfBytes);
// Original producer, creator, creationDate, modificationDate are now GONE
// Replaced with pdf-lib defaults

// CORRECT -- preserve original metadata
const pdfDoc = await PDFDocument.load(existingPdfBytes, {
  updateMetadata: false,
});
// Original metadata intact
```

**Rule**: ALWAYS pass `{ updateMetadata: false }` when loading PDFs where original metadata must be preserved (archival, legal, compliance).

---

## AP-003: Using copy() for Full Document Cloning

**Severity**: High -- loses form fields and outlines

```typescript
// WRONG -- copy() does NOT copy forms, outlines, or catalog structures
const original = await PDFDocument.load(formPdfBytes);
const clone = await original.copy();
// clone has pages but NO form fields, NO bookmarks

// CORRECT for page-level cloning -- use copyPages
const source = await PDFDocument.load(formPdfBytes);
const dest = await PDFDocument.create();
const indices = source.getPageIndices();
const copiedPages = await dest.copyPages(source, indices);
copiedPages.forEach(page => dest.addPage(page));
```

**Rule**: NEVER use `copy()` when forms, outlines, or other catalog-level structures must be preserved. Use `copyPages()` for page-level operations.

---

## AP-004: Keywords Type Mismatch

**Severity**: Medium -- causes confusion in data processing

```typescript
// WRONG -- expecting getKeywords() to return string[]
pdfDoc.setKeywords(['finance', 'report', '2024']);
const keywords = pdfDoc.getKeywords();
keywords.forEach(k => console.log(k));  // TypeError: keywords.forEach is not a function
// getKeywords() returns "finance,report,2024" (a single string)

// CORRECT -- handle the type asymmetry
pdfDoc.setKeywords(['finance', 'report', '2024']);
const keywordsString = pdfDoc.getKeywords();  // "finance,report,2024"
const keywordsArray = keywordsString?.split(',').map(k => k.trim()) ?? [];
```

**Rule**: ALWAYS remember that `setKeywords()` accepts `string[]` but `getKeywords()` returns `string | undefined`. Split the result manually when you need an array.

---

## AP-005: Ignoring Encryption Limitations

**Severity**: High -- creates false expectations about encrypted PDF handling

```typescript
// WRONG -- assuming ignoreEncryption means pdf-lib can decrypt content
const pdfDoc = await PDFDocument.load(passwordProtectedPdf, {
  ignoreEncryption: true,
});
// If the PDF has actual content encryption, the content may be garbled

// CORRECT -- understand what ignoreEncryption actually does
const pdfDoc = await PDFDocument.load(pdfBytes, {
  ignoreEncryption: true,
});
// This skips encryption VALIDATION, not decryption
// Works for PDFs with encryption markers but no actual content encryption
// For truly encrypted PDFs, pre-process with qpdf or mutool first
```

**Rule**: NEVER rely on `ignoreEncryption` for password-protected PDFs. pdf-lib has NO decryption engine and NO encryption output support.

---

## AP-006: Saving Empty Documents Without addDefaultPage

**Severity**: Low -- causes viewer compatibility issues

```typescript
// RISKY -- some PDF viewers crash or refuse to open 0-page PDFs
const pdfDoc = await PDFDocument.create();
const pdfBytes = await pdfDoc.save({ addDefaultPage: false });
// Result: a valid PDF structure with 0 pages

// SAFER -- let the default behavior add a blank page
const pdfDoc = await PDFDocument.create();
const pdfBytes = await pdfDoc.save();
// addDefaultPage: true (default) inserts a blank page if needed
```

**Rule**: NEVER disable `addDefaultPage` unless you are certain the document has at least one page. ALWAYS verify `pdfDoc.getPageCount() > 0` before saving with `{ addDefaultPage: false }`.

---

## AP-007: Assuming All Viewers Support Attachments

**Severity**: Medium -- attachments invisible in many viewers

```typescript
// WRONG -- assuming all users will see the attachment
await pdfDoc.attach(dataBytes, 'report.csv', {
  mimeType: 'text/csv',
  description: 'Important data',
});
// Chrome PDF viewer, Safari, many mobile viewers do NOT show attachments

// CORRECT -- document the attachment and provide viewer guidance
page.drawText('This PDF contains an attached file (report.csv).', {
  x: 50, y: 50, size: 10, font,
});
page.drawText('Open with Adobe Acrobat Reader or Firefox to access attachments.', {
  x: 50, y: 35, size: 10, font,
});

await pdfDoc.attach(dataBytes, 'report.csv', {
  mimeType: 'text/csv',
  description: 'Important data',
});
```

**Rule**: ALWAYS add visible text noting the presence of attachments. NEVER assume the end user's PDF viewer supports attachment display. Only Adobe Acrobat Reader, Foxit Reader, and Firefox reliably show attachments.

---

## AP-008: Assuming JavaScript Runs Everywhere

**Severity**: Medium -- JavaScript silently ignored in most viewers

```typescript
// WRONG -- assuming the print dialog will trigger for all users
pdfDoc.addJavaScript('autoPrint',
  'this.print({bUI: true, bSilent: false});'
);
// Only works in Adobe Acrobat. Chrome, Firefox PDF viewer,
// Preview.app, and most mobile viewers ignore PDF JavaScript entirely.

// CORRECT -- treat JavaScript as enhancement, not requirement
pdfDoc.addJavaScript('autoPrint',
  'this.print({bUI: true, bSilent: false});'
);
// Document that auto-print requires Adobe Acrobat
// Provide manual print instructions as fallback
```

**Rule**: NEVER depend on `addJavaScript()` for critical functionality. ALWAYS treat PDF JavaScript as an optional enhancement that only works in Adobe Acrobat.

---

## AP-009: Duplicate JavaScript Names

**Severity**: Medium -- last one wins, silently overwriting previous

```typescript
// WRONG -- duplicate names cause silent overwrites
pdfDoc.addJavaScript('init', 'app.alert("First script");');
pdfDoc.addJavaScript('init', 'app.alert("Second script");');
// Only "Second script" will execute

// CORRECT -- use unique names
pdfDoc.addJavaScript('initWelcome', 'app.alert("Welcome!");');
pdfDoc.addJavaScript('initPrint', 'this.print();');
```

**Rule**: ALWAYS use unique names for `addJavaScript()` calls. The name parameter serves as a key -- duplicate names silently overwrite.

---

## AP-010: Not Setting Metadata for Searchability

**Severity**: Low -- reduces document discoverability

```typescript
// WRONG -- no metadata set, document is hard to find/identify
const pdfDoc = await PDFDocument.create();
const page = pdfDoc.addPage();
page.drawText('Invoice #2024-001', { x: 50, y: 700, size: 20 });
const bytes = await pdfDoc.save();
// File metadata is empty -- no title, no author, no keywords

// CORRECT -- set metadata for all distributed documents
const pdfDoc = await PDFDocument.create();
pdfDoc.setTitle('Invoice #2024-001', { showInWindowTitleBar: true });
pdfDoc.setAuthor('Billing Department');
pdfDoc.setSubject('Monthly invoice for December 2024');
pdfDoc.setKeywords(['invoice', '2024', 'billing']);
pdfDoc.setLanguage('en-US');
```

**Rule**: ALWAYS set at minimum `title`, `author`, and `language` for documents that will be distributed or archived. This enables search indexing, accessibility, and document management.

---

## AP-011: Using useObjectStreams: false Without Reason

**Severity**: Low -- unnecessarily large file sizes

```typescript
// WRONG -- disabling compression without need
const pdfBytes = await pdfDoc.save({
  useObjectStreams: false,
});
// File is significantly larger than necessary

// CORRECT -- only disable when needed for compatibility
// Default (useObjectStreams: true) produces smaller files
const pdfBytes = await pdfDoc.save();
```

**Rule**: ALWAYS use the default `useObjectStreams: true` unless targeting PDF readers that do not support PDF 1.5. Disabling object streams can significantly increase file size.
