# Method Signatures — Page Manipulation

## copyPages()

```typescript
async copyPages(
  srcDoc: PDFDocument,
  indices: number[]
): Promise<PDFPage[]>
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `srcDoc` | `PDFDocument` | The source document to copy pages from |
| `indices` | `number[]` | 0-based array of page indices to copy |

**Returns:** `Promise<PDFPage[]>` — Array of copied pages belonging to the calling document.

**Behavior:**
- ALWAYS call on the DESTINATION document, NOT the source
- Clones page content, resources, and annotations into the destination's context
- Returned pages are NOT automatically added to the document
- You MUST call `addPage()` or `insertPage()` for each returned page
- The order of returned pages matches the order of the `indices` array

**Example:**
```typescript
const destDoc = await PDFDocument.create();
const srcDoc = await PDFDocument.load(srcBytes);

// Copy pages 0, 2, 4 from source
const copiedPages = await destDoc.copyPages(srcDoc, [0, 2, 4]);
// copiedPages[0] = copy of srcDoc page 0
// copiedPages[1] = copy of srcDoc page 2
// copiedPages[2] = copy of srcDoc page 4
```

---

## addPage()

```typescript
addPage(page?: PDFPage | [number, number]): PDFPage
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | `PDFPage \| [number, number] \| undefined` | Page to add, or dimensions for a new blank page |

**Overloads:**
- `addPage()` — Creates and adds a blank page with default dimensions
- `addPage([width, height])` — Creates and adds a blank page with specified dimensions (PDF points)
- `addPage(copiedPage)` — Adds an existing PDFPage (from `copyPages()`)

**Returns:** `PDFPage` — The added page.

**Behavior:**
- ALWAYS appends the page at the END of the document
- When passing a `PDFPage`, it MUST belong to this document (use `copyPages()` first for cross-document)
- Synchronous method — no `await` needed

---

## insertPage()

```typescript
insertPage(index: number, page?: PDFPage | [number, number]): PDFPage
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `index` | `number` | 0-based position to insert the page at |
| `page` | `PDFPage \| [number, number] \| undefined` | Page to insert, or dimensions for a new blank page |

**Overloads:**
- `insertPage(0)` — Inserts a blank page at the beginning
- `insertPage(2, [612, 792])` — Inserts a Letter-sized blank page at index 2
- `insertPage(1, copiedPage)` — Inserts a copied page at index 1

**Returns:** `PDFPage` — The inserted page.

**Behavior:**
- Inserts BEFORE the page currently at the given index
- `insertPage(0, page)` makes it the first page
- `insertPage(doc.getPageCount(), page)` is equivalent to `addPage(page)`
- Synchronous method — no `await` needed

---

## removePage()

```typescript
removePage(index: number): void
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `index` | `number` | 0-based index of the page to remove |

**Behavior:**
- Removes the page at the specified index from the document
- Subsequent pages shift down by one index
- Throws if index is out of bounds
- Synchronous method — no `await` needed
- Does NOT return the removed page

**WARNING:** When removing multiple pages in a loop, ALWAYS iterate in reverse order to avoid index shifting:

```typescript
// CORRECT — reverse order
const pagesToRemove = [1, 3, 5];
for (const index of pagesToRemove.sort((a, b) => b - a)) {
  doc.removePage(index);
}

// WRONG — forward order causes index shift
// After removing index 1, old index 3 becomes index 2
for (const index of pagesToRemove) {
  doc.removePage(index); // indices are wrong after first removal
}
```

---

## getPageCount()

```typescript
getPageCount(): number
```

**Returns:** The total number of pages in the document.

---

## getPageIndices()

```typescript
getPageIndices(): number[]
```

**Returns:** An array of all valid page indices, e.g., `[0, 1, 2, 3]` for a 4-page document.

---

## getPage()

```typescript
getPage(index: number): PDFPage
```

**Returns:** The `PDFPage` at the given 0-based index. Throws if out of bounds.

---

## getPages()

```typescript
getPages(): PDFPage[]
```

**Returns:** An array of all `PDFPage` objects in document order.

---

## Helper: Build All-Pages Index Array

When you need to copy ALL pages from a source document:

```typescript
const pageCount = sourcePdf.getPageCount();
const allIndices = Array.from({ length: pageCount }, (_, i) => i);
// Result: [0, 1, 2, ..., pageCount - 1]

const copiedPages = await destDoc.copyPages(sourcePdf, allIndices);
```

Alternative using `getPageIndices()`:

```typescript
const allIndices = sourcePdf.getPageIndices();
const copiedPages = await destDoc.copyPages(sourcePdf, allIndices);
```
