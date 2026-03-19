---
name: pdflib-errors-common
description: "Diagnoses and fixes common pdf-lib mistakes including forgotten await, coordinate system confusion, unsupported image formats, cross-document page errors, form field name issues, color value range errors, and save option side effects. Activates when debugging pdf-lib code, encountering unexpected behavior, or reviewing pdf-lib code for correctness."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Common pdf-lib Errors — Diagnosis & Fixes

> Non-font, non-loading errors. For font/encoding errors see `pdflib-errors-fonts`.
> For document loading errors see `pdflib-errors-loading`.

## Quick Diagnostic Checklist

When pdf-lib code produces unexpected results, check these in order:

1. **Missing `await`?** — `embedFont`, `embedPng`, `embedJpg`, `copyPages`, `save` ALL return Promises
2. **Y-coordinate wrong?** — PDF origin is BOTTOM-LEFT, not top-left
3. **Unsupported image format?** — ONLY PNG and JPG are supported
4. **Cross-document page copy?** — MUST use `copyPages()` before `addPage()`
5. **Form field not found?** — Names are case-sensitive and fully qualified (dot-separated)
6. **Color looks wrong?** — `rgb()` takes 0.0–1.0, NOT 0–255
7. **Drawing looks wrong?** — `drawEllipse` uses `xScale`/`yScale`, `drawLine` uses `thickness`
8. **Unexpected blank page?** — `save()` adds one if document has zero pages
9. **Form appearances changed?** — `save()` auto-updates field appearances by default

---

## Decision Tree — "My PDF Output Looks Wrong"

```
PDF output is wrong
├── Content is missing entirely
│   ├── Used embedFont/embedPng without await? → See §1
│   ├── Copied page without copyPages()? → See §4
│   └── save() returned empty bytes? → Check await on save()
├── Content is in wrong position
│   ├── Text/image appears at bottom instead of top? → See §2
│   └── Ellipse/circle shape is wrong? → See §11
├── Colors are wrong
│   └── Used 0-255 instead of 0-1? → See §6
├── Form fields don't work
│   ├── "No field with name X" error? → See §5
│   ├── Duplicate field name error? → See §7
│   └── Field appears empty until clicked? → See §9
├── Merged PDF has issues
│   ├── Error when adding page from another doc? → See §4
│   └── copy() lost form fields/bookmarks? → See §8
└── Unexpected extra page
    └── Empty document got blank page on save? → See §10
```

---

## §1 Forgotten `await` on Async Methods

**Severity: CRITICAL** — Most common cause of "everything breaks silently."

These methods return `Promise` — you MUST `await` them:

| Method | Returns |
|--------|---------|
| `embedFont()` | `Promise<PDFFont>` |
| `embedPng()` | `Promise<PDFImage>` |
| `embedJpg()` | `Promise<PDFImage>` |
| `copyPages()` | `Promise<PDFPage[]>` |
| `save()` | `Promise<Uint8Array>` |
| `saveAsBase64()` | `Promise<string>` |
| `copy()` | `Promise<PDFDocument>` |
| `embedPdf()` | `Promise<PDFEmbeddedPage[]>` |

**Broken:**
```typescript
const font = pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font }); // font is a Promise, NOT a PDFFont!
```

**Fixed:**
```typescript
const font = await pdfDoc.embedFont(StandardFonts.Helvetica);
page.drawText('Hello', { font });
```

> ALWAYS use `await` on every `embed*`, `copy*`, `save*`, and `load` call.

---

## §2 Coordinate System Confusion (Bottom-Left Origin)

**Severity: HIGH** — PDF uses bottom-left origin. HTML/CSS/Canvas uses top-left.

**Broken:**
```typescript
// Developer expects y=50 to be near the top
page.drawText('Title', { x: 50, y: 50 }); // Appears near BOTTOM
```

**Fixed:**
```typescript
const { height } = page.getSize();
page.drawText('Title', { x: 50, y: height - 50 }); // Near TOP
```

> ALWAYS subtract from `page.getSize().height` when positioning from the top.
> NEVER assume y=0 is at the top of the page.

---

## §3 Unsupported Image Formats

**Severity: HIGH** — pdf-lib supports ONLY PNG and JPG. No GIF, BMP, SVG, WebP, or TIFF.

| Method | Format |
|--------|--------|
| `embedPng()` | PNG only |
| `embedJpg()` | JPG/JPEG only |

There is NO `embedImage()`, `embedGif()`, `embedBmp()`, or `embedSvg()` method.

**Fix:** Convert images to PNG or JPG before embedding using an external library (e.g., Sharp, Canvas API, or browser canvas).

> ALWAYS convert non-PNG/JPG images before passing to pdf-lib.
> NEVER attempt to pass GIF, SVG, BMP, or WebP bytes to embed methods.

---

## §4 Cross-Document Page Addition Without `copyPages()`

**Severity: CRITICAL** — Pages from one document CANNOT be added directly to another.

**Broken:**
```typescript
const sourcePage = sourceDoc.getPage(0);
targetDoc.addPage(sourcePage); // THROWS ERROR
```

**Fixed:**
```typescript
const [copiedPage] = await targetDoc.copyPages(sourceDoc, [0]);
targetDoc.addPage(copiedPage);
```

> ALWAYS call `copyPages()` on the TARGET document first.
> ALWAYS `addPage()` or `insertPage()` each copied page — `copyPages()` alone does NOT add them.

---

## §5 Form Field Not Found (Case-Sensitive, Fully Qualified Names)

**Severity: HIGH** — Field names are case-sensitive and use dot-separated hierarchical names.

**Broken:**
```typescript
form.getTextField('name'); // Throws if actual name is "form.personal.Name"
```

**Fixed:**
```typescript
// ALWAYS enumerate fields first to discover exact names
const fields = form.getFields();
fields.forEach(field => {
  console.log(`Type: ${field.constructor.name}, Name: "${field.getName()}"`);
});
// Then use the exact name from enumeration
const nameField = form.getTextField('form.personal.Name');
```

> ALWAYS enumerate fields with `getFields()` before accessing by name.
> NEVER guess field names — they are case-sensitive and may include parent prefixes.

---

## §6 Color Values: 0.0–1.0, NOT 0–255

**Severity: MEDIUM** — All color functions use normalized 0.0–1.0 range.

**Broken:**
```typescript
const red = rgb(255, 0, 0);   // Creates white/unexpected color
const gray = grayscale(128);   // Wrong
```

**Fixed:**
```typescript
const red = rgb(1, 0, 0);         // Correct red
const gray = grayscale(0.5);       // Correct 50% gray
const teal = cmyk(1, 0, 0.5, 0);  // CMYK also uses 0-1
```

> ALWAYS use 0.0–1.0 for `rgb()`, `cmyk()`, and `grayscale()`.
> NEVER pass 0–255 values to color functions.

---

## §7 Duplicate Field Name Errors

**Severity: MEDIUM** — Creating two form fields with the same name throws an error.

**Broken:**
```typescript
form.createTextField('email');
form.createTextField('email'); // ERROR: duplicate name
```

**Fixed:**
```typescript
form.createTextField('email_primary');
form.createTextField('email_secondary');
```

> ALWAYS use unique names for every form field.

---

## §8 `copy()` Does NOT Copy AcroForms or Outlines

**Severity: MEDIUM** — `PDFDocument.copy()` creates a shallow copy. Form fields (AcroForm) and bookmarks (outlines) are NOT included in the copy.

**Impact:** If you copy a document with forms, the copied document will have the pages but non-functional form fields.

**Workaround:** Re-create form fields on the copied document, or use `copyPages()` to selectively transfer pages instead.

> NEVER rely on `copy()` to preserve form fields or bookmarks.

---

## §9 `save()` Auto-Updates Field Appearances

**Severity: MEDIUM** — By default, `save()` calls `form.updateFieldAppearances()` automatically, which can change how fields look.

**To prevent this:**
```typescript
const pdfBytes = await pdfDoc.save({
  updateFieldAppearances: false,
});
```

> ALWAYS pass `{ updateFieldAppearances: false }` to `save()` when you want to preserve existing field appearances exactly as they are.

---

## §10 `save()` Adds Blank Page If Document Is Empty

**Severity: LOW** — Calling `save()` on a document with zero pages automatically inserts one blank page, because the PDF specification requires at least one page.

> ALWAYS add at least one page before calling `save()` to avoid unexpected blank pages.

---

## §11 Drawing Method Parameter Names

**Severity: MEDIUM** — Some drawing methods use non-obvious parameter names.

### `drawEllipse` — Uses `xScale`/`yScale`, NOT `width`/`height`

**Broken:**
```typescript
page.drawEllipse({ x: 200, y: 200, width: 100, height: 50 }); // width/height IGNORED
```

**Fixed:**
```typescript
page.drawEllipse({ x: 200, y: 200, xScale: 100, yScale: 50 });
```

### `drawLine` — Uses `thickness`, NOT `borderWidth`

**Broken:**
```typescript
page.drawLine({ start: { x: 0, y: 0 }, end: { x: 100, y: 100 }, borderWidth: 2 }); // IGNORED
```

**Fixed:**
```typescript
page.drawLine({ start: { x: 0, y: 0 }, end: { x: 100, y: 100 }, thickness: 2 });
```

> ALWAYS use `xScale`/`yScale` for `drawEllipse`.
> ALWAYS use `thickness` for `drawLine`.
> NEVER use `width`/`height` for ellipses or `borderWidth` for lines.

---

## §12 Keywords Setter/Getter Asymmetry

**Severity: LOW** — `setKeywords()` takes `string[]` but `getKeywords()` returns `string | undefined`.

```typescript
pdfDoc.setKeywords(['pdf', 'invoice', 'report']);
const keywords = pdfDoc.getKeywords(); // Returns "pdf,invoice,report" (single string)
const keywordArray = keywords?.split(',') ?? []; // Parse back to array
```

> ALWAYS split the return value of `getKeywords()` to get an array.

---

## §13 Page Rotation — Multiples of 90 Only

**Severity: LOW** — Page rotation ONLY accepts multiples of 90 degrees.

```typescript
page.setRotation(degrees(90));  // Valid: 0, 90, 180, 270
page.setRotation(degrees(45));  // NOT a valid page rotation
```

> ALWAYS use 0, 90, 180, or 270 for page rotation.
> NEVER use arbitrary angles for page rotation (drawing rotation supports any angle).

---

## Reference Files

- [references/methods.md](references/methods.md) — API signatures for methods involved in common errors
- [references/examples.md](references/examples.md) — Complete error reproduction and fix examples
- [references/anti-patterns.md](references/anti-patterns.md) — Comprehensive catalog of mistakes with root causes
