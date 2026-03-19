# Extended Examples — Merge, Split, Extract, Reorder

## Merge Multiple PDFs with Metadata

```typescript
import { PDFDocument } from 'pdf-lib';

async function mergePdfsWithMetadata(
  pdfByteArrays: Uint8Array[],
  title: string,
  author: string,
): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create();

  for (const pdfBytes of pdfByteArrays) {
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const indices = sourcePdf.getPageIndices();
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  mergedPdf.setTitle(title);
  mergedPdf.setAuthor(author);
  mergedPdf.setCreationDate(new Date());
  mergedPdf.setModificationDate(new Date());

  return mergedPdf.save();
}
```

## Merge with Encrypted Source PDFs

```typescript
import { PDFDocument } from 'pdf-lib';

async function mergeWithEncrypted(
  pdfByteArrays: Uint8Array[],
): Promise<Uint8Array> {
  const mergedPdf = await PDFDocument.create();

  for (const pdfBytes of pdfByteArrays) {
    // ignoreEncryption allows loading PDFs with encryption markers
    const sourcePdf = await PDFDocument.load(pdfBytes, {
      ignoreEncryption: true,
    });
    const indices = sourcePdf.getPageIndices();
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  return mergedPdf.save();
}
```

## Extract Non-Contiguous Pages

```typescript
import { PDFDocument } from 'pdf-lib';

async function extractPages(
  pdfBytes: Uint8Array,
  pageIndices: number[], // e.g., [0, 3, 7, 12]
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  const copiedPages = await newPdf.copyPages(sourcePdf, pageIndices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}

// Usage: Extract pages 1, 4, and 8 (0-indexed)
const result = await extractPages(originalPdfBytes, [0, 3, 7]);
```

## Extract Even/Odd Pages

```typescript
import { PDFDocument } from 'pdf-lib';

async function extractEvenPages(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const newPdf = await PDFDocument.create();

  // Even pages: 0, 2, 4, ... (0-indexed)
  const evenIndices = Array.from({ length: pageCount }, (_, i) => i).filter(
    (i) => i % 2 === 0,
  );

  const copiedPages = await newPdf.copyPages(sourcePdf, evenIndices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}

async function extractOddPages(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const newPdf = await PDFDocument.create();

  // Odd pages: 1, 3, 5, ... (0-indexed)
  const oddIndices = Array.from({ length: pageCount }, (_, i) => i).filter(
    (i) => i % 2 === 1,
  );

  const copiedPages = await newPdf.copyPages(sourcePdf, oddIndices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}
```

## Reverse Page Order

```typescript
import { PDFDocument } from 'pdf-lib';

async function reversePages(pdfBytes: Uint8Array): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const reversedPdf = await PDFDocument.create();

  // Create reversed index array: [n-1, n-2, ..., 1, 0]
  const reversedIndices = Array.from(
    { length: pageCount },
    (_, i) => pageCount - 1 - i,
  );

  const copiedPages = await reversedPdf.copyPages(sourcePdf, reversedIndices);
  copiedPages.forEach((page) => reversedPdf.addPage(page));

  return reversedPdf.save();
}
```

## Interleave Pages from Two Documents

```typescript
import { PDFDocument } from 'pdf-lib';

async function interleavePages(
  pdfBytesA: Uint8Array,
  pdfBytesB: Uint8Array,
): Promise<Uint8Array> {
  const docA = await PDFDocument.load(pdfBytesA);
  const docB = await PDFDocument.load(pdfBytesB);
  const result = await PDFDocument.create();

  const countA = docA.getPageCount();
  const countB = docB.getPageCount();
  const maxCount = Math.max(countA, countB);

  for (let i = 0; i < maxCount; i++) {
    if (i < countA) {
      const [pageA] = await result.copyPages(docA, [i]);
      result.addPage(pageA);
    }
    if (i < countB) {
      const [pageB] = await result.copyPages(docB, [i]);
      result.addPage(pageB);
    }
  }

  return result.save();
}
```

## Split Into Chunks of N Pages

```typescript
import { PDFDocument } from 'pdf-lib';

async function splitIntoChunks(
  pdfBytes: Uint8Array,
  chunkSize: number,
): Promise<Uint8Array[]> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const chunks: Uint8Array[] = [];

  for (let start = 0; start < pageCount; start += chunkSize) {
    const end = Math.min(start + chunkSize, pageCount);
    const chunkPdf = await PDFDocument.create();

    const indices = Array.from({ length: end - start }, (_, i) => start + i);
    const copiedPages = await chunkPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => chunkPdf.addPage(page));

    chunks.push(await chunkPdf.save());
  }

  return chunks;
}

// Usage: Split a 10-page PDF into 3-page chunks
// Returns: [pages 0-2], [pages 3-5], [pages 6-8], [page 9]
const chunks = await splitIntoChunks(pdfBytes, 3);
```

## Remove Specific Pages

```typescript
import { PDFDocument } from 'pdf-lib';

async function removePages(
  pdfBytes: Uint8Array,
  pagesToRemove: number[], // 0-indexed
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const pageCount = sourcePdf.getPageCount();
  const newPdf = await PDFDocument.create();

  // Build array of indices to KEEP (exclude the ones to remove)
  const removeSet = new Set(pagesToRemove);
  const keepIndices = Array.from({ length: pageCount }, (_, i) => i).filter(
    (i) => !removeSet.has(i),
  );

  const copiedPages = await newPdf.copyPages(sourcePdf, keepIndices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}

// Usage: Remove pages 2 and 5 (0-indexed) from a PDF
const result = await removePages(pdfBytes, [2, 5]);
```

## Duplicate a Page Multiple Times

```typescript
import { PDFDocument } from 'pdf-lib';

async function duplicatePage(
  pdfBytes: Uint8Array,
  pageIndex: number,
  copies: number,
): Promise<Uint8Array> {
  const sourcePdf = await PDFDocument.load(pdfBytes);
  const newPdf = await PDFDocument.create();

  // Create an indices array with the same page repeated
  const indices = Array.from({ length: copies }, () => pageIndex);
  const copiedPages = await newPdf.copyPages(sourcePdf, indices);
  copiedPages.forEach((page) => newPdf.addPage(page));

  return newPdf.save();
}

// Usage: Create 5 copies of page 0
const result = await duplicatePage(pdfBytes, 0, 5);
```

## Merge with Page Number Validation

```typescript
import { PDFDocument } from 'pdf-lib';

async function safeMerge(
  pdfByteArrays: Uint8Array[],
): Promise<{ pdf: Uint8Array; totalPages: number }> {
  const mergedPdf = await PDFDocument.create();
  let totalPages = 0;

  for (let docIndex = 0; docIndex < pdfByteArrays.length; docIndex++) {
    const pdfBytes = pdfByteArrays[docIndex];
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const pageCount = sourcePdf.getPageCount();

    if (pageCount === 0) {
      console.warn(`Document ${docIndex} has no pages, skipping.`);
      continue;
    }

    const indices = sourcePdf.getPageIndices();
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
    totalPages += pageCount;
  }

  return {
    pdf: await mergedPdf.save(),
    totalPages,
  };
}
```

## Node.js File I/O Example

```typescript
import { PDFDocument } from 'pdf-lib';
import { readFile, writeFile } from 'fs/promises';

async function mergeFiles(
  inputPaths: string[],
  outputPath: string,
): Promise<void> {
  const mergedPdf = await PDFDocument.create();

  for (const filePath of inputPaths) {
    const pdfBytes = await readFile(filePath);
    const sourcePdf = await PDFDocument.load(pdfBytes);
    const indices = sourcePdf.getPageIndices();
    const copiedPages = await mergedPdf.copyPages(sourcePdf, indices);
    copiedPages.forEach((page) => mergedPdf.addPage(page));
  }

  const mergedBytes = await mergedPdf.save();
  await writeFile(outputPath, mergedBytes);
}

// Usage
await mergeFiles(
  ['./doc1.pdf', './doc2.pdf', './doc3.pdf'],
  './merged-output.pdf',
);
```
