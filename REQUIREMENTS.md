# REQUIREMENTS

## What This Skill Package Must Achieve

### Primary Goal
Enable Claude to write correct, version-aware pdf-lib code for creating and modifying PDF documents — without hallucinating APIs.

### What Claude Should Do After Loading Skills
1. Recognize pdf-lib context from user requests (PDF creation, modification, form filling)
2. Select the correct skill(s) automatically based on the request
3. Write correct TypeScript/JavaScript code using pdf-lib 1.x APIs
4. Avoid known anti-patterns and common AI mistakes
5. Follow best practices documented in the skill references

### Quality Guarantees
| Guarantee | Description |
|-----------|-------------|
| Version-correct | Code MUST target pdf-lib 1.x |
| API-accurate | All method signatures verified against official docs |
| Type-safe | TypeScript examples with proper type annotations |
| Anti-pattern-free | Known mistakes are explicitly documented and avoided |
| Deterministic | Skills use ALWAYS/NEVER language, not suggestions |
| Self-contained | Each skill works independently without requiring other skills |

---

## Per-Area Requirements

### 1. PDFDocument API
| Requirement | Detail |
|-------------|--------|
| Creation | `PDFDocument.create()` for new documents |
| Loading | `PDFDocument.load()` with options (ignoreEncryption, updateMetadata) |
| Saving | `pdfDoc.save()` and `pdfDoc.saveAsBase64()` variants |
| Metadata | Title, author, subject, keywords, creator, producer |
| Critical | Proper async/await usage — all operations are async |

### 2. Page Operations
| Requirement | Detail |
|-------------|--------|
| Adding | `pdfDoc.addPage()` with standard sizes and custom dimensions |
| Getting | `pdfDoc.getPages()`, `pdfDoc.getPage(index)` |
| Copying | `pdfDoc.copyPages()` for cross-document page operations |
| Embedding | Page embedding from other documents |
| Dimensions | `page.getSize()`, `page.setSize()`, standard page sizes |
| Critical | Coordinate system — origin at bottom-left, not top-left |

### 3. Text Operations
| Requirement | Detail |
|-------------|--------|
| Drawing | `page.drawText()` with position, font, size, color |
| Fonts | Standard fonts (Helvetica, TimesRoman, Courier), custom font embedding |
| Font embedding | `pdfDoc.embedFont()` for TTF/OTF fonts |
| Unicode | Font subsetting and unicode character support |
| Text measuring | `font.widthOfTextAtSize()`, `font.heightAtSize()` |
| Critical | Standard fonts do NOT support unicode — ALWAYS embed custom fonts for non-ASCII |

### 4. Image Operations
| Requirement | Detail |
|-------------|--------|
| Embedding | `pdfDoc.embedPng()`, `pdfDoc.embedJpg()` |
| Drawing | `page.drawImage()` with position, dimensions, rotation |
| Scaling | Maintaining aspect ratio, fitting to page |
| Critical | Only PNG and JPG are supported — no SVG, no GIF, no WebP |

### 5. Form Fields
| Requirement | Detail |
|-------------|--------|
| Reading | `pdfDoc.getForm()`, getting fields by name |
| Field types | Text fields, checkboxes, radio buttons, dropdowns, buttons |
| Filling | Setting field values, checking boxes, selecting options |
| Flattening | `form.flatten()` to make fields non-editable |
| Appearance | Custom field appearances, font settings |
| Critical | ALWAYS call `form.flatten()` after filling if the PDF should not be editable |

### 6. Drawing Operations
| Requirement | Detail |
|-------------|--------|
| Shapes | Rectangles, circles, ellipses, lines |
| Colors | `rgb()`, `cmyk()`, `grayscale()` color constructors |
| Operators | Line width, dash patterns, opacity |
| SVG paths | `page.drawSvgPath()` for complex shapes |
| Critical | All coordinates use bottom-left origin |

### 7. Document Manipulation
| Requirement | Detail |
|-------------|--------|
| Merging | Copying pages between documents |
| Splitting | Extracting page ranges into new documents |
| Encryption | Loading encrypted PDFs, encryption options |
| Attachments | File attachments to PDF documents |
| Critical | Cross-document operations require `copyPages()` — NEVER directly add pages from another document |

---

## Critical Requirements (apply to ALL skills)

- All code MUST work with pdf-lib 1.x (latest 1.17.x)
- All TypeScript MUST include proper type imports from `pdf-lib`
- Async/await MUST be used correctly — all pdf-lib operations are async
- Coordinate system MUST be documented (bottom-left origin)
- Code examples MUST be verified against official documentation

---

## Structural Requirements

### Skill Format
- SKILL.md < 500 lines (heavy content in references/)
- YAML frontmatter with name and description (including trigger words)
- English-only content
- Deterministic language (ALWAYS/NEVER, imperative)

### Skill Categories
| Category | Purpose | Must Include |
|----------|---------|--------------|
| syntax/ | How to write it | Method signatures, code patterns, type annotations |
| impl/ | How to build it | Decision trees, workflows, step-by-step |
| errors/ | How to handle failures | Error patterns, diagnostics, recovery |
| core/ | Cross-cutting | API overview, architecture, concepts |
| agents/ | Orchestration | Validation checklists, auto-detection |

---

## Research Requirements (before creating any skill)

1. Official documentation MUST be consulted and referenced
2. Source code MUST be checked for accuracy when docs are ambiguous
3. Anti-patterns MUST be identified from real issues (GitHub issues)
4. Code examples MUST be verified (not hallucinated)
5. Version accuracy MUST be confirmed via WebFetch (D-012)

---

## Non-Requirements (explicitly out of scope)

- No PDF.js coverage (separate package — PDFjs-Claude-Skill-Package)
- No server-side PDF rendering (pdf-lib is a creation/modification library)
- No PDF viewer implementation (pdf-lib does not render PDFs)
- No comparison guides with other PDF libraries (jsPDF, pdfmake, etc.)
- No browser-specific DOM integration tutorials
