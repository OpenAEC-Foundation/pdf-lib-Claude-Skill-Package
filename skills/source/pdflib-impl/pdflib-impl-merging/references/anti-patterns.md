# Anti-Patterns — PDF Merging & Page Manipulation

## Anti-Pattern 1: Direct Cross-Document Page Addition

**Severity: CRITICAL — causes runtime error**

NEVER add a page from one PDFDocument directly to another. Pages are bound to their source document context. You MUST use `copyPages()` to clone pages into the destination document first.

```typescript
// BROKEN — NEVER do this
const sourceDoc = await PDFDocument.load(sourceBytes);
const targetDoc = await PDFDocument.create();
const page = sourceDoc.getPage(0);
targetDoc.addPage(page); // THROWS — page belongs to sourceDoc

// CORRECT — ALWAYS do this
const sourceDoc = await PDFDocument.load(sourceBytes);
const targetDoc = await PDFDocument.create();
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage); // Works — page now belongs to targetDoc
```

## Anti-Pattern 2: Forgetting to Add Copied Pages

**Severity: HIGH — produces empty or incomplete documents**

`copyPages()` does NOT add pages to the document. It only clones them into the document's context. You MUST explicitly call `addPage()` or `insertPage()` for each copied page.

```typescript
// BROKEN — pages are copied but never added
const copiedPages = await destDoc.copyPages(sourceDoc, [0, 1, 2]);
// Document is empty! No pages were added.
const pdfBytes = await destDoc.save(); // Saves a PDF with 0 (or 1 default) pages

// CORRECT — ALWAYS add each copied page
const copiedPages = await destDoc.copyPages(sourceDoc, [0, 1, 2]);
copiedPages.forEach((page) => destDoc.addPage(page));
const pdfBytes = await destDoc.save(); // Saves a PDF with 3 pages
```

## Anti-Pattern 3: Forward-Order Page Removal

**Severity: HIGH — removes wrong pages due to index shifting**

When removing multiple pages by index, removing in forward order causes subsequent indices to shift, leading to wrong pages being removed or index-out-of-bounds errors.

```typescript
// BROKEN — index shifting causes wrong page removal
const pagesToRemove = [1, 3, 5];
for (const index of pagesToRemove) {
  doc.removePage(index);
  // After removing index 1: old page 3 is now at index 2
  // Next iteration removes index 3, which is now the OLD page 5
}

// CORRECT — remove in reverse order
const pagesToRemove = [1, 3, 5];
for (const index of pagesToRemove.sort((a, b) => b - a)) {
  doc.removePage(index); // 5 first, then 3, then 1 — no shifting issues
}

// ALTERNATIVE — copy only the pages you want to KEEP (safer approach)
const removeSet = new Set(pagesToRemove);
const keepIndices = doc.getPageIndices().filter((i) => !removeSet.has(i));
const newDoc = await PDFDocument.create();
const kept = await newDoc.copyPages(doc, keepIndices);
kept.forEach((page) => newDoc.addPage(page));
```

## Anti-Pattern 4: Assuming Form Fields Transfer During Copy

**Severity: HIGH — form fields become non-functional (GitHub #1205, #1587)**

When copying pages that contain form fields, the visual widgets (field appearances) are copied, but the AcroForm field definitions in the document catalog may NOT transfer. This results in fields that look correct but are not interactive.

```typescript
// PROBLEM — form fields appear but do not work after merge
const formPdf = await PDFDocument.load(formBytes);
const mergedPdf = await PDFDocument.create();
const indices = formPdf.getPageIndices();
const copiedPages = await mergedPdf.copyPages(formPdf, indices);
copiedPages.forEach((page) => mergedPdf.addPage(page));
// Form fields on copied pages are likely non-functional

// WORKAROUND — re-create form fields after merge if interactivity is required
const form = mergedPdf.getForm();
// Manually create new fields and position them on the copied pages
```

**There is NO automatic solution in pdf-lib 1.x.** If you need functional form fields in a merged document, you MUST re-create them programmatically after merging.

## Anti-Pattern 5: Ignoring File Size Bloat (GitHub #1338)

**Severity: MEDIUM — can cause 10x-30x file size increase**

pdf-lib does NOT deduplicate resources when copying pages. If two source PDFs embed the same font (e.g., Helvetica), the merged document contains TWO copies of that font. With images, the bloat can be extreme.

```typescript
// PROBLEM — each source contributes its own copy of shared resources
for (const pdfBytes of manyPdfs) {
  const source = await PDFDocument.load(pdfBytes);
  const pages = await merged.copyPages(source, source.getPageIndices());
  pages.forEach((p) => merged.addPage(p));
}
// Output file can be 10-30x the sum of input sizes

// MITIGATION — post-process with an external tool
const mergedBytes = await merged.save();
// Then optimize with qpdf, Ghostscript, or similar:
// qpdf --linearize --object-streams=generate input.pdf output.pdf
// gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -o output.pdf input.pdf
```

## Anti-Pattern 6: Not Handling Encrypted PDFs

**Severity: MEDIUM — causes load failure**

Loading an encrypted PDF without `ignoreEncryption: true` throws an error. However, `ignoreEncryption` only works for PDFs with encryption markers that lack actual content encryption.

```typescript
// BROKEN — throws on encrypted PDFs
const sourcePdf = await PDFDocument.load(encryptedBytes);

// PARTIAL FIX — works for some encrypted PDFs
const sourcePdf = await PDFDocument.load(encryptedBytes, {
  ignoreEncryption: true,
});

// For truly encrypted PDFs, pre-process externally:
// qpdf --decrypt --password=PASS input.pdf decrypted.pdf
```

## Anti-Pattern 7: Blank Pages After Merge (GitHub #1579, #1767)

**Severity: MEDIUM — content silently disappears**

Some PDFs produce blank pages after merging. This happens when content streams reference resources (fonts, images, color spaces) using names that do not resolve correctly in the destination document's resource dictionary.

**Symptoms:**
- Pages appear blank in Adobe Acrobat Reader
- Same pages may render correctly in browser-based PDF viewers
- No error is thrown during merge or save

**Mitigations:**
- ALWAYS test merged output in multiple PDF viewers
- Try loading source PDFs with `{ throwOnInvalidObject: true }` to detect issues early
- If a specific source PDF causes blank pages, try re-saving it with another tool first

## Anti-Pattern 8: Using embedPdf() When You Mean copyPages()

**Severity: MEDIUM — wrong output structure**

`embedPdf()` embeds a page as drawable content INSIDE another page (like an image). It does NOT add the page as a standalone page in the document. Use `copyPages()` when you want separate pages.

```typescript
// WRONG for merging — embeds page as content, does NOT add a new page
const [embedded] = await destDoc.embedPdf(sourceBytes);
const page = destDoc.addPage();
page.drawPage(embedded); // Source page is drawn inside this page, possibly scaled

// CORRECT for merging — adds page as a standalone page
const sourceDoc = await PDFDocument.load(sourceBytes);
const [copied] = await destDoc.copyPages(sourceDoc, [0]);
destDoc.addPage(copied); // Source page IS this page, at original size
```

## Summary: Rules for Safe Page Manipulation

1. **ALWAYS** use `copyPages()` to transfer pages between documents
2. **ALWAYS** call `addPage()` or `insertPage()` after `copyPages()`
3. **ALWAYS** remove pages in reverse index order
4. **ALWAYS** test merged output in multiple PDF viewers
5. **NEVER** add a page from one document directly to another
6. **NEVER** assume form fields will work after copying pages
7. **NEVER** assume merged file size will be the sum of input sizes
8. **NEVER** confuse `copyPages()` (for merging) with `embedPdf()` (for drawing)
