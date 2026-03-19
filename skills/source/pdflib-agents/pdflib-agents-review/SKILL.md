---
name: pdflib-agents-review
description: >
  Use when reviewing or validating pdf-lib code before deployment.
  Prevents shipping broken PDF generation: runs a 7-point checklist covering
  async/await, coordinate system, font handling, imports, form handling, and
  known anti-patterns. Catches issues Claude's default code generation misses.
  Keywords: review, validate, audit, checklist, code review, anti-pattern, quality check.
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# pdf-lib Code Review Agent

## Purpose

This skill provides a step-by-step validation checklist for reviewing pdf-lib code. Follow EVERY check in order. Each check includes what to verify, why it matters, and how to fix violations.

## Quick Reference — Severity Legend

| Level | Meaning |
|-------|---------|
| CRITICAL | Code WILL fail at runtime or produce corrupt output |
| WARNING | Code MAY produce incorrect visual output or unexpected behavior |
| INFO | Code works but violates best practices or may cause issues at scale |

---

## Validation Checklist

### CHECK 1: Async/Await Audit [CRITICAL]

**What**: Verify that ALL async pdf-lib methods use `await`.

**Async methods that ALWAYS require `await`:**

| Method | Returns |
|--------|---------|
| `PDFDocument.create()` | `Promise<PDFDocument>` |
| `PDFDocument.load()` | `Promise<PDFDocument>` |
| `pdfDoc.embedFont()` | `Promise<PDFFont>` |
| `pdfDoc.embedPng()` | `Promise<PDFImage>` |
| `pdfDoc.embedJpg()` | `Promise<PDFImage>` |
| `pdfDoc.copyPages()` | `Promise<PDFPage[]>` |
| `pdfDoc.embedPdf()` | `Promise<PDFEmbeddedPage[]>` |
| `pdfDoc.embedPage()` | `Promise<PDFEmbeddedPage>` |
| `pdfDoc.embedPages()` | `Promise<PDFEmbeddedPage[]>` |
| `pdfDoc.save()` | `Promise<Uint8Array>` |
| `pdfDoc.saveAsBase64()` | `Promise<string>` |
| `pdfDoc.copy()` | `Promise<PDFDocument>` |
| `pdfDoc.flush()` | `Promise<void>` |
| `pdfDoc.attach()` | `Promise<void>` |

**Sync exception**: `pdfDoc.embedStandardFont()` does NOT need `await`.

**How to fix**: Add `await` before every call. Ensure the containing function is `async`.

> See [references/methods.md](references/methods.md) for the complete async/sync method reference.

---

### CHECK 2: Import Completeness [CRITICAL]

**What**: Verify ALL used pdf-lib exports are imported.

**Required imports by operation type:**

| Operation | Required Imports |
|-----------|-----------------|
| Any pdf-lib usage | `PDFDocument` from `'pdf-lib'` |
| Standard fonts | `StandardFonts` from `'pdf-lib'` |
| Color (RGB) | `rgb` from `'pdf-lib'` |
| Color (CMYK) | `cmyk` from `'pdf-lib'` |
| Color (Grayscale) | `grayscale` from `'pdf-lib'` |
| Rotation | `degrees` or `radians` from `'pdf-lib'` |
| Page sizes | `PageSizes` from `'pdf-lib'` |
| Text alignment | `TextAlignment` from `'pdf-lib'` |
| Blend modes | `BlendMode` from `'pdf-lib'` |
| Custom fonts | `fontkit` from `'@pdf-lib/fontkit'` (separate package) |

**How to fix**: Add missing imports. NEVER use destructured names without importing them.

---

### CHECK 3: Coordinate System Verification [WARNING]

**What**: Verify coordinates use bottom-left origin (PDF standard).

**Rules:**
- y=0 is the BOTTOM of the page, NOT the top
- To position content near the top, use `height - offset`
- ALWAYS call `page.getSize()` or `page.getHeight()` to calculate top-relative positions

**Pattern to verify:**
```typescript
const { width, height } = page.getSize()
// Top of page: y = height - margin
// Bottom of page: y = margin
```

**Red flags:**
- Using small y values (e.g., `y: 50`) and expecting content at the top
- Using large y values without referencing page height
- Hardcoded y values without comments explaining positioning intent

**How to fix**: Replace hardcoded top-relative values with `height - offset` calculations.

---

### CHECK 4: Font Handling [CRITICAL]

**What**: Verify font usage matches character requirements.

**Sub-checks:**

**4a. Standard font character support:**
- Standard fonts (Helvetica, TimesRoman, Courier, etc.) support ONLY WinAnsi encoding
- WinAnsi covers basic Latin and Western European characters
- NEVER use standard fonts with: Cyrillic, CJK, Arabic, Hebrew, Thai, emoji
- Error produced: `"WinAnsi cannot encode "X" (0xNNNN)"`

**4b. Custom font registration:**
- If code uses `pdfDoc.embedFont(fontBytes)` with raw bytes, `pdfDoc.registerFontkit(fontkit)` MUST be called first
- `@pdf-lib/fontkit` MUST be installed as a dependency
- Registration MUST happen before ANY `embedFont()` call with custom bytes

**4c. Form field fonts:**
- Default form font is Helvetica (Latin-only)
- For non-Latin text in form fields, ALWAYS call `form.updateFieldAppearances(customFont)` after setting values
- fontkit registration is REQUIRED for custom form fonts

**How to fix**: See [references/anti-patterns.md](references/anti-patterns.md) sections on font handling.

---

### CHECK 5: Image Format Validation [CRITICAL]

**What**: Verify only PNG and JPG images are used.

**Rules:**
- ONLY `pdfDoc.embedPng()` and `pdfDoc.embedJpg()` exist
- There is NO `embedGif()`, `embedBmp()`, `embedSvg()`, `embedWebP()`, or generic `embedImage()`
- NEVER pass non-PNG/JPG data to these methods

**How to fix**: Convert unsupported formats to PNG or JPG before embedding.

---

### CHECK 6: Cross-Document Operations [CRITICAL]

**What**: Verify pages are copied correctly between documents.

**Rules:**
- ALWAYS use `await targetDoc.copyPages(sourceDoc, indices)` to transfer pages
- NEVER add a page from one document directly to another via `addPage(sourcePage)`
- After `copyPages()`, ALWAYS call `addPage()` or `insertPage()` for each copied page
- `copyPages()` alone does NOT add pages to the document

**How to fix**: Replace direct page addition with the copyPages + addPage pattern.

---

### CHECK 7: Form Field Validation [WARNING]

**What**: Verify form operations follow correct patterns.

**Sub-checks:**

**7a. Field enumeration before access:**
- ALWAYS enumerate fields with `form.getFields()` before accessing by name
- Field names are CASE-SENSITIVE
- Field names use fully qualified dot-notation (e.g., `"form.section.fieldName"`)

**7b. Duplicate field names:**
- NEVER create two fields with the same name — this throws an error
- Use unique suffixed names when creating multiple similar fields

**7c. Form flattening:**
- ALWAYS flatten forms when generating final output PDFs
- NEVER flatten if the form needs to be filled again later
- Call `form.flatten()` AFTER setting all field values

**7d. Appearance updates:**
- After setting form field values, appearances MUST be updated for correct rendering
- Either rely on `save({ updateFieldAppearances: true })` (default) or call `form.updateFieldAppearances()` explicitly
- For custom fonts in forms, ALWAYS pass the font: `form.updateFieldAppearances(customFont)`

**7e. XFA forms:**
- If working with XFA-based PDFs, call `form.deleteXFA()` before manipulating AcroForm fields

> See [references/examples.md](references/examples.md) for correct form handling patterns.

---

### CHECK 8: Color Value Range [WARNING]

**What**: Verify all color values are in the 0.0-1.0 range.

**Rules:**
- `rgb(r, g, b)` — each value MUST be 0.0 to 1.0, NOT 0 to 255
- `cmyk(c, m, y, k)` — each value MUST be 0.0 to 1.0
- `grayscale(g)` — value MUST be 0.0 to 1.0
- `opacity` values — MUST be 0.0 to 1.0

**Red flags:**
- Values greater than 1.0 (e.g., `rgb(255, 0, 0)`)
- Integer values where decimals are expected

**How to fix**: Divide by 255 if converting from 0-255 range: `rgb(r/255, g/255, b/255)`.

---

### CHECK 9: Save Options Review [INFO]

**What**: Verify save options are appropriate for the use case.

**Sub-checks:**

**9a. `updateFieldAppearances` option:**
- Default: `true` — refreshes form field visual rendering on save
- Set to `false` ONLY if you manually called `form.updateFieldAppearances()` or have no form fields
- Setting to `false` when form fields exist may cause fields to appear empty

**9b. `addDefaultPage` option:**
- Default: `true` — inserts a blank page if the document has zero pages
- Set to `false` when intentionally creating empty documents or when page count is managed explicitly

**9c. `useObjectStreams` option:**
- Default: `true` — enables compression for smaller file size
- Set to `false` for maximum compatibility with older PDF readers

**How to fix**: Review save options against the intended output requirements.

---

### CHECK 10: Image Aspect Ratio [INFO]

**What**: Verify images maintain correct proportions.

**Rules:**
- ALWAYS use `image.scale(factor)` or `image.scaleToFit(maxWidth, maxHeight)` for proportional sizing
- NEVER set arbitrary `width` and `height` independently without calculating aspect ratio
- For manual scaling: calculate one dimension from the other using the original ratio

**How to fix**: Replace independent width/height with `scale()` or `scaleToFit()` calls.

---

## Review Execution Order

When reviewing pdf-lib code, execute checks in this exact order (CRITICAL first):

1. **CHECK 1** — Async/Await Audit [CRITICAL]
2. **CHECK 2** — Import Completeness [CRITICAL]
3. **CHECK 4** — Font Handling [CRITICAL]
4. **CHECK 5** — Image Format Validation [CRITICAL]
5. **CHECK 6** — Cross-Document Operations [CRITICAL]
6. **CHECK 3** — Coordinate System Verification [WARNING]
7. **CHECK 7** — Form Field Validation [WARNING]
8. **CHECK 8** — Color Value Range [WARNING]
9. **CHECK 9** — Save Options Review [INFO]
10. **CHECK 10** — Image Aspect Ratio [INFO]

## Review Output Format

For each violation found, report:

```
[SEVERITY] CHECK N: Brief description
  Line(s): <line numbers>
  Issue: <what is wrong>
  Fix: <how to correct it>
```

## Reference Files

- [references/methods.md](references/methods.md) — Complete async/sync method reference
- [references/examples.md](references/examples.md) — Before/after code review examples
- [references/anti-patterns.md](references/anti-patterns.md) — Complete anti-pattern checklist
