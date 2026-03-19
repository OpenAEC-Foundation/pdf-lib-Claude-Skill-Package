# Anti-Patterns Catalog — Common pdf-lib Mistakes

Comprehensive catalog of mistakes organized by category, with root cause analysis.

---

## Category A: Async/Await Mistakes

### A1. Forgotten `await` on `embedFont()`
- **Root cause:** Developer forgets `embedFont()` returns `Promise<PDFFont>`
- **Symptom:** TypeError or silent failure when using the font
- **Fix:** ALWAYS `await` the `embedFont()` call
- **Frequency:** Very high — the #1 most common mistake overall

### A2. Forgotten `await` on `embedPng()` / `embedJpg()`
- **Root cause:** Developer forgets image embedding is async
- **Symptom:** TypeError when passing Promise to `drawImage()`
- **Fix:** ALWAYS `await` image embed calls
- **Frequency:** High

### A3. Forgotten `await` on `save()`
- **Root cause:** Developer treats `save()` as synchronous
- **Symptom:** Writing a Promise object to file instead of Uint8Array bytes
- **Fix:** ALWAYS `await pdfDoc.save()`
- **Frequency:** Medium

### A4. Forgotten `await` on `copyPages()`
- **Root cause:** Developer forgets cross-document copy is async
- **Symptom:** Trying to add undefined/Promise pages to document
- **Fix:** ALWAYS `await` the `copyPages()` call
- **Frequency:** Medium

---

## Category B: Coordinate System Mistakes

### B1. Y-Coordinate Reversed
- **Root cause:** Assuming HTML/CSS coordinate system (y=0 at top)
- **Symptom:** Content appears at the opposite vertical position
- **Fix:** Use `height - y` to convert from top-down coordinates
- **Frequency:** Very high

### B2. Content Off-Page
- **Root cause:** Using coordinates outside the page dimensions
- **Symptom:** Content exists in PDF but is not visible
- **Fix:** ALWAYS check against `page.getSize()` before drawing
- **Frequency:** Medium

### B3. Forgetting Page Size Varies
- **Root cause:** Assuming all pages are the same size (e.g., A4)
- **Symptom:** Content positioned incorrectly on non-standard pages
- **Fix:** ALWAYS call `page.getSize()` for each page, not once globally
- **Frequency:** Low

---

## Category C: Image Mistakes

### C1. Unsupported Image Format
- **Root cause:** Attempting to use GIF, BMP, SVG, WebP, or TIFF
- **Symptom:** Error on embed, or corrupted image data
- **Fix:** Convert to PNG or JPG before embedding
- **Frequency:** Medium

### C2. Image Aspect Ratio Distortion
- **Root cause:** Setting arbitrary width and height instead of using `scale()`
- **Symptom:** Stretched or squished images
- **Fix:** Use `image.scale(factor)` or `image.scaleToFit(maxW, maxH)` to calculate dimensions
- **Frequency:** Medium

### C3. Mixing Up `embedPng` and `embedJpg`
- **Root cause:** Passing JPG bytes to `embedPng()` or vice versa
- **Symptom:** Error during embedding or corrupted image
- **Fix:** ALWAYS match the embed method to the actual image format
- **Frequency:** Low

---

## Category D: Cross-Document Mistakes

### D1. Direct Page Addition Without `copyPages()`
- **Root cause:** Trying to `addPage()` with a page from a different document
- **Symptom:** Error thrown — pages are bound to their source document
- **Fix:** ALWAYS use `copyPages()` first, then `addPage()` the copied page
- **Frequency:** High

### D2. Forgetting to `addPage()` After `copyPages()`
- **Root cause:** Assuming `copyPages()` automatically adds pages
- **Symptom:** Pages are copied into memory but never appear in the document
- **Fix:** ALWAYS call `addPage()` or `insertPage()` for each copied page
- **Frequency:** Medium

### D3. `copy()` Losing AcroForm and Outlines
- **Root cause:** `copy()` does not deep-copy form fields or bookmarks
- **Symptom:** Copied document has pages but non-functional forms, no bookmarks
- **Fix:** Re-create form fields after copying, or use alternative approach
- **Frequency:** Low

---

## Category E: Form Field Mistakes

### E1. Case-Sensitive Field Names
- **Root cause:** Using wrong case for field name (e.g., "Name" vs "name")
- **Symptom:** "No field with name X" error
- **Fix:** ALWAYS enumerate fields first with `getFields()` and `getName()`
- **Frequency:** High

### E2. Missing Fully Qualified Name
- **Root cause:** Using short name when field has hierarchical path
- **Symptom:** "No field with name X" error
- **Fix:** Use the full dot-separated path (e.g., "form.personal.name")
- **Frequency:** High

### E3. Duplicate Field Names
- **Root cause:** Creating two fields with the same name
- **Symptom:** Error thrown on second `createTextField()` (or similar)
- **Fix:** ALWAYS use unique names for every field
- **Frequency:** Medium

### E4. Field Appears Empty Until Clicked
- **Root cause:** Appearance stream not updated after setting text
- **Symptom:** Field looks empty but shows value when clicked/focused
- **Fix:** Call `form.updateFieldAppearances()` or `field.updateAppearances(font)` after setting values
- **Frequency:** Medium

---

## Category F: Color and Drawing Mistakes

### F1. Color Values 0-255 Instead of 0-1
- **Root cause:** Using CSS/HTML color range instead of PDF normalized range
- **Symptom:** All colors appear as white or unexpected values
- **Fix:** Divide by 255, or use direct 0.0–1.0 values
- **Frequency:** High

### F2. `drawEllipse` with `width`/`height`
- **Root cause:** Assuming same parameter names as `drawRectangle`
- **Symptom:** Ellipse renders with default size, custom dimensions ignored
- **Fix:** Use `xScale` and `yScale` parameters
- **Frequency:** Medium

### F3. `drawLine` with `borderWidth`
- **Root cause:** Assuming same parameter name as rectangles/circles
- **Symptom:** Line renders with default thickness, custom width ignored
- **Fix:** Use `thickness` parameter
- **Frequency:** Medium

---

## Category G: Save/Export Mistakes

### G1. Auto-Added Blank Page
- **Root cause:** Saving a document with zero pages
- **Symptom:** Output PDF has one unexpected blank page
- **Fix:** ALWAYS add at least one page before saving
- **Frequency:** Low

### G2. Unexpected Form Appearance Changes
- **Root cause:** `save()` auto-calls `updateFieldAppearances()` by default
- **Symptom:** Form field fonts/styles change unexpectedly on save
- **Fix:** Pass `{ updateFieldAppearances: false }` to `save()`
- **Frequency:** Medium

---

## Category H: Metadata Mistakes

### H1. Keywords Getter/Setter Asymmetry
- **Root cause:** `setKeywords()` takes `string[]` but `getKeywords()` returns `string`
- **Symptom:** Code expecting array from `getKeywords()` fails
- **Fix:** Split the returned string: `getKeywords()?.split(',') ?? []`
- **Frequency:** Low

---

## Category I: Rotation Mistakes

### I1. Non-90° Page Rotation
- **Root cause:** Using arbitrary angle for page rotation
- **Symptom:** Unexpected behavior or error
- **Fix:** ONLY use 0, 90, 180, or 270 for `page.setRotation(degrees(n))`
- **Note:** Drawing operations (`drawText`, `drawImage`, etc.) support arbitrary rotation angles via `rotate` option
- **Frequency:** Low

---

## Summary — Top 5 Mistakes by Frequency

| Rank | Mistake | Category | Section |
|------|---------|----------|---------|
| 1 | Forgotten `await` on embed/save | Async | A1–A4 |
| 2 | Y-coordinate reversed | Coordinates | B1 |
| 3 | Form field name mismatch | Forms | E1–E2 |
| 4 | Color values 0-255 vs 0-1 | Drawing | F1 |
| 5 | Direct page addition without `copyPages()` | Cross-doc | D1 |
