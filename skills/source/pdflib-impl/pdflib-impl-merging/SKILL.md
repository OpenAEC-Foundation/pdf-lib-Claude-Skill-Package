---
name: pdflib-impl-merging
description: >
  Use when merging, splitting, or reordering PDF pages across documents with pdf-lib.
  Prevents the critical cross-document error: directly adding pages from another
  document instead of using copyPages() first, which corrupts the PDF.
  Covers copyPages workflow, merge patterns, page extraction, splitting, reordering.
  Keywords: copyPages, merge, split, extract, addPage, insertPage,
  cross-document, merge PDFs, combine PDFs, split PDF, extract pages,
  join PDF files, reorder pages.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# PDF Merging, Splitting & Page Manipulation

## Critical Rule

**ALWAYS use `copyPages()` to transfer pages between documents. NEVER add a page from one PDFDocument directly to another PDFDocument.**

```typescript
// CORRECT — ALWAYS do this
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);

// WRONG — NEVER do this
const sourcePage = sourceDoc.getPage(0);
targetDoc.addPage(sourcePage); // THROWS ERROR — page belongs to sourceDoc
```

## Quick Reference

| Operation | Method Chain |
|-----------|-------------|
| Copy pages | `await destDoc.copyPages(srcDoc, indices)` |
| Add copied page at end | `destDoc.addPage(copiedPage)` |
| Insert copied page at index | `destDoc.insertPage(index, copiedPage)` |
| Remove page | `doc.removePage(index)` |
| Get page count | `doc.getPageCount()` |
| Get all page indices | `doc.getPageIndices()` |
| Get all indices array | `Array.from({ length: doc.getPageCount() }, (_, i) => i)` |

## Decision Tree

```
What do you need to do with PDF pages?
|
+-- Merge multiple PDFs into one?
|   +-- Use the Merge Multiple Documents pattern
|   +-- Loop over source PDFs: load -> copyPages -> addPage each
|
+-- Extract specific pages from a PDF?
|   +-- Single page? -> copyPages(source, [pageIndex])
|   +-- Page range? -> copyPages(source, [start..end])
|   +-- Non-contiguous? -> copyPages(source, [2, 5, 8])
|
+-- Split a PDF into individual pages?
|   +-- Loop: create new PDFDocument per page -> copyPages -> addPage -> save
|
+-- Reorder pages within a PDF?
|   +-- Create new doc -> copyPages with desired order -> addPage each
|
+-- Remove pages from a PDF?
|   +-- Option A: doc.removePage(index) — mutates in place
|   +-- Option B: copyPages only the pages you WANT to keep (safer)
|
+-- Embed a page as an image inside another page?
|   +-- Use embedPdf() / embedPage() instead (see note below)
```

## Core Workflow: copyPages()

### Signature

```typescript
async copyPages(
  srcDoc: PDFDocument,
  indices: number[]
): Promise<PDFPage[]>
```

### How It Works

1. **Load** the source document with `PDFDocument.load()`
2. **Call `copyPages()`** on the DESTINATION document, passing the source and page indices
3. **Add each copied page** to the destination with `addPage()` or `insertPage()`

The `copyPages()` method clones pages from the source into the destination's context. It returns an array of `PDFPage` objects that belong to the destination document. These pages are NOT automatically added — you MUST call `addPage()` or `insertPage()` for each one.

### Page Indices

Page indices are 0-based. The first page is index `0`, the last page is `doc.getPageCount() - 1`.

## Patterns

### Merge Multiple Documents

```typescript
import { PDFDocument } from 'pdf-lib';

async function mergePdfs(pdfByteArrays: Uint8Array[]): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create();

  for (const pdfBytes of pdfByteArrays) {
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const pageCount = sourcePdf.getPageCount();
    const indices = Array.from({ length: pageCount }, (_, i) => i);
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  return mergedPdf.save();
}
```

### Extract a Single Page

```typescript
async function extractPage(
  pdfBytes: Uint8Array,
  pageIndex: number,
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  const [extractedPage] = await newPdf.copyPages(sourcePdf, [pageIndex]);
  newPdf.addPage(extractedPage);

  return newPdf.save();
}
```

### Extract a Page Range

```typescript
async function extractPageRange(
  pdfBytes: Uint8Array,
  startPage: number, // 0-indexed
  endPage: number,   // 0-indexed, inclusive
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  const indices = Array.from(
    { length: endPage - startPage + 1 },
    (_, i) => startPage + i,
  );
  const pages = await newPdf.copyPages(sourcePdf, indices);
  pages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}
```

### Split Into Individual Pages

```typescript
async function splitAllPages(pdfBytes: Uint8Array): Promise<Uint8Array[]> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const results: Uint8Array[] = [];

  for (let i = 0; i < pageCount; i++) {
    const newPdf = await PDFDocument.create();
    const [page] = await newPdf.copyPages(sourcePdf, [i]);
    newPdf.addPage(page);
    results.push(await newPdf.save());
  }

  return results;
}
```

### Reorder Pages

```typescript
async function reorderPages(
  pdfBytes: Uint8Array,
  newOrder: number[], // e.g., [2, 0, 1] to move page 3 to front
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const reorderedPdf = await PDFDocument.create();

  const copiedPages = await reorderedPdf.copyPages(sourcePdf, newOrder);
  copiedPages.forEach((page) => reorderedPdf.addPage(page));

  return reorderedPdf.save();
}
```

### Insert Pages at Specific Position

```typescript
async function insertPagesAt(
  targetPdfBytes: Uint8Array,
  sourcePdfBytes: Uint8Array,
  insertAtIndex: number,
): Promise<Uint8Array> {
  const targetPdf = await PDFDocument.load(targetPdfBytes);
  const sourcePdf = await PDFDocument.load(sourcePdfBytes);

  const sourcePageCount = sourcePdf.getPageCount();
  const indices = Array.from({ length: sourcePageCount }, (_, i) => i);
  const copiedPages = await targetPdf.copyPages(sourcePdf, indices);

  // Insert pages in reverse order at the same index to maintain order
  for (let i = copiedPages.length - 1; i >= 0; i--) {
    targetPdf.insertPage(insertAtIndex, copiedPages[i]);
  }

  return targetPdf.save();
}
```

### Remove Pages (Keep Selected)

```typescript
async function keepPages(
  pdfBytes: Uint8Array,
  pagesToKeep: number[], // indices of pages to keep
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  const copiedPages = await newPdf.copyPages(sourcePdf, pagesToKeep);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}
```

## copyPages() vs embedPdf() / embedPage()

| Feature | `copyPages()` | `embedPdf()` / `embedPage()` |
|---------|--------------|-------------------------------|
| Purpose | Add full pages to a document | Embed a page as drawable content inside another page |
| Result | `PDFPage[]` — standalone pages | `PDFEmbeddedPage` — drawn with `page.drawPage()` |
| Use case | Merging, splitting, reordering | Thumbnails, N-up printing, watermarks |
| Page count | Increases page count | Does NOT increase page count |
| Interactivity | Preserves page structure | Flattened into visual content |

ALWAYS use `copyPages()` when you want pages to appear as separate pages in the output.
ALWAYS use `embedPdf()` / `embedPage()` when you want to draw a page inside another page.

## Known Limitations

### Form Fields Lost During Copy (GitHub #1205, #1587)

When copying pages that contain form fields, the visual widgets (field appearances) are copied but the AcroForm field definitions may NOT transfer. This results in form fields that appear visually but are NOT interactive.

**Workaround**: After merging, re-create form fields programmatically on the merged document if interactivity is required.

### Blank Pages in Merged PDFs (GitHub #1579, #1767)

Some PDFs produce blank pages after merging. Content streams may reference resources that do not copy correctly. This particularly affects PDFs viewed in Adobe Acrobat Reader (browser-based viewers may render correctly).

**No guaranteed workaround.** ALWAYS test merged output in multiple PDF viewers (Adobe Reader, browser, Foxit).

### File Size Bloat After Merge (GitHub #1338)

pdf-lib does NOT deduplicate resources when copying pages. Each copied page brings its own fonts, images, and other resources — even if identical resources already exist in the destination document. Reports exist of 15 MB input producing 433 MB output.

**Workaround**: Use an external tool (e.g., `qpdf --linearize`, Ghostscript) to optimize the merged PDF after creation.

### Encrypted Source PDFs

pdf-lib does NOT support password-based decryption. To copy pages from encrypted PDFs:

```typescript
const sourcePdf = await PDFDocument.load(encryptedBytes, {
  ignoreEncryption: true,
});
```

This works ONLY for PDFs with encryption markers but no actual content encryption. For truly encrypted PDFs, pre-process with an external tool (qpdf, mutool) before loading.

## Reference Files

- [Method Signatures](references/methods.md) — `copyPages`, `addPage`, `insertPage`, `removePage` full signatures
- [Extended Examples](references/examples.md) — Merge, split, extract, reorder patterns with edge cases
- [Anti-Patterns](references/anti-patterns.md) — Direct page addition, form field loss, file size bloat
