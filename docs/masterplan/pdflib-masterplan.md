# pdf-lib Skill Package — Definitive Masterplan

## Status

Phase 3 complete. Finalized from raw masterplan after vooronderzoek review.
Date: 2026-03-19

---

## Decisions Made During Refinement

| # | Decision | Rationale |
|---|----------|-----------|
| D-01 | **Merged** `pdflib-core-setup` into `pdflib-core-architecture` | Setup is too thin standalone (just npm install + 2 imports). Architecture skill already covers key types and async design — adding installation/imports is natural. |
| D-02 | **Merged** `pdflib-impl-metadata` into `pdflib-syntax-document` | Metadata is just 8 getters + 9 setters on PDFDocument. Not enough for a standalone skill. Document lifecycle skill already covers PDFDocument creation/load/save — metadata fits naturally. |
| D-03 | **Reordered** batches: forms + form-filling together | Forms syntax and form-filling implementation are tightly coupled. Doing them in the same batch lets the impl skill reference the syntax skill directly. |

**Result**: 19 raw skills → **17 definitive skills** (2 merges, 0 additions, 0 removals).

---

## Definitive Skill Inventory (17 skills)

### pdflib-core/ (1 skill)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `pdflib-core-architecture` | Architecture overview; key types (PDFDocument, PDFPage, PDFFont, PDFImage, PDFForm); async-first design; coordinate system (bottom-left); PDF points; platform support; installation & setup; imports; fontkit registration | `PDFDocument`, `PDFPage`, `PDFFont`, `PDFImage`, `PDFForm`, `StandardFonts`, `rgb()`, `degrees()`, `PageSizes` | Vooronderzoek §1, §2 | M | None |

### pdflib-syntax/ (7 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `pdflib-syntax-document` | PDFDocument create/load/save; load options (ignoreEncryption, parseSpeed, throwOnInvalidObject, updateMetadata, capNumbers); save options (useObjectStreams, addDefaultPage, objectsPerTick, updateFieldAppearances); saveAsBase64; copy(); metadata getters/setters (title, author, subject, keywords, creator, producer, dates, language); attachments; addJavaScript | `PDFDocument.create()`, `.load()`, `.save()`, `.saveAsBase64()`, `.copy()`, `.attach()`, `.addJavaScript()`, metadata setters/getters | Vooronderzoek §3 | L | core-architecture |
| `pdflib-syntax-pages` | addPage/insertPage/removePage; getPage/getPages/getPageCount/getPageIndices; dimensions (getSize/setSize/getWidth/setWidth/getHeight/setHeight); rotation; PageSizes enum (55 sizes); page boxes (media/crop/bleed/trim/art); content transformations (scale/scaleContent/scaleAnnotations/translateContent); position cursor (moveTo/moveUp/moveDown) | `addPage()`, `insertPage()`, `removePage()`, `getPage()`, `getPages()`, `PageSizes`, box methods, transformation methods | Vooronderzoek §4 | M | syntax-document |
| `pdflib-syntax-text` | drawText with all options (x, y, font, size, color, opacity, rotate, xSkew, ySkew, lineHeight, maxWidth, wordBreaks, blendMode); multiline text (\n); page-level defaults (setFont/setFontSize/setFontColor/setLineHeight); text centering pattern; text measuring (widthOfTextAtSize, heightAtSize, sizeAtHeight) | `page.drawText()`, `page.setFont()`, `page.setFontSize()`, `font.widthOfTextAtSize()`, `font.heightAtSize()` | Vooronderzoek §5 | M | syntax-pages, syntax-fonts |
| `pdflib-syntax-fonts` | StandardFonts enum (14 fonts); embedFont() for standard + custom; embedStandardFont() (sync); fontkit registration; custom font embedding (TTF/OTF); EmbedFontOptions (subset, customName, features); WinAnsi encoding limitation; unicode support via custom fonts; font compatibility matrix; getCharacterSet(); encodeText() | `embedFont()`, `embedStandardFont()`, `StandardFonts`, `registerFontkit()`, `@pdf-lib/fontkit` | Vooronderzoek §6 | M | core-architecture |
| `pdflib-syntax-images` | embedPng/embedJpg (string/Uint8Array/ArrayBuffer); PDFImage properties (width, height); scale(factor); scaleToFit(w, h); size(); drawImage with all options (x, y, width, height, rotate, xSkew, ySkew, opacity, blendMode); aspect ratio preservation patterns | `embedPng()`, `embedJpg()`, `page.drawImage()`, `image.scale()`, `image.scaleToFit()` | Vooronderzoek §7 | M | syntax-pages |
| `pdflib-syntax-drawing` | drawRectangle, drawSquare, drawCircle, drawEllipse, drawLine, drawSvgPath, drawPage; color constructors (rgb, cmyk, grayscale — all 0-1 range); rotation helpers (degrees, radians); all shape options; border options; blend modes; complete drawing options reference table | All `page.draw*()` methods, `rgb()`, `cmyk()`, `grayscale()`, `degrees()`, `radians()`, `BlendMode` | Vooronderzoek §8 | M | syntax-pages |
| `pdflib-syntax-forms` | PDFForm API (getForm, getFields, getField, getFieldMaybe); all field type getters/creators; PDFTextField complete API; PDFCheckBox complete API; PDFRadioGroup complete API; PDFDropdown complete API; PDFOptionList complete API; PDFButton complete API; PDFSignature; form.flatten(); form.updateFieldAppearances(); XFA handling (hasXFA, deleteXFA); field properties (readOnly, required, exported) | `getForm()`, all field getters/creators, field manipulation methods, `form.flatten()` | Vooronderzoek §9 | L | syntax-document |

### pdflib-impl/ (4 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `pdflib-impl-form-filling` | Complete form filling workflow; filling existing forms step-by-step; creating new forms; unicode/custom font support for forms; form.updateFieldAppearances(font); flattening workflow; selective flattening workaround; when to flatten decision tree; font decision tree for forms | `form.flatten()`, `field.setText()`, `field.check()`, `field.select()`, `form.updateFieldAppearances()` | Vooronderzoek §9 | L | syntax-forms |
| `pdflib-impl-merging` | copyPages workflow; merge multiple documents pattern; page extraction (single, range, all); splitting into individual pages; page insertion/reordering; cross-document operation rules; known limitations (form fields lost, blank pages, file size bloat) | `copyPages()`, `addPage()`, `insertPage()`, `removePage()` | Vooronderzoek §10 | M | syntax-document, syntax-pages |
| `pdflib-impl-pdf-generation` | End-to-end PDF generation; multi-page documents; headers/footers pattern; page numbering; table-like layouts; invoice/report patterns; combining text + images + shapes | All drawing + text + image APIs combined | Vooronderzoek §5, §7, §8 | L | syntax-text, syntax-drawing, syntax-images |
| `pdflib-impl-page-layout` | Coordinate system strategies; top-down positioning helper (height - y); margins pattern; content area calculation; multi-column layout; text wrapping with maxWidth; content flow across pages; centering patterns (horizontal + vertical) | Coordinate math, `getSize()`, `widthOfTextAtSize()` | Vooronderzoek §4, §5 | M | syntax-pages, syntax-text |

### pdflib-errors/ (3 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `pdflib-errors-fonts` | WinAnsi encoding error; standard font unicode limitation; missing fontkit registration; font embedding failures; font not applied to form fields; diacritics failures; font compatibility matrix; GitHub issues catalog | Font error patterns, fontkit errors | Vooronderzoek §11 | M | syntax-fonts |
| `pdflib-errors-loading` | Loading encrypted PDFs; ignoreEncryption usage; malformed PDFs; throwOnInvalidObject; encrypted PDF limitations (no decryption engine); restricted document handling; updateMetadata side effects | `load()` options, `isEncrypted` | Vooronderzoek §11 | M | syntax-document |
| `pdflib-errors-common` | Forgotten await (async errors); coordinate system confusion (bottom-left vs top-left); unsupported image formats; cross-document page errors; form field not found (case-sensitive names); duplicate field names; color values 0-1 not 0-255; copy() limitations; save() auto-behaviors | All error patterns | Vooronderzoek §11 | M | All syntax skills |

### pdflib-agents/ (2 skills)

| Name | Scope | Key APIs | Research Input | Complexity | Dependencies |
|------|-------|----------|----------------|------------|-------------|
| `pdflib-agents-review` | Validation checklist for generated pdf-lib code; async/await audit; coordinate system check; font handling verification; import completeness; anti-pattern detection; form handling validation | All validation rules | Vooronderzoek §11 | M | ALL syntax + impl skills |
| `pdflib-agents-project-scaffolder` | Generate complete pdf-lib project; TypeScript setup; fontkit configuration; PDF generation boilerplate; Node.js + browser variants | All scaffolding patterns | Vooronderzoek §1, §2, §3 | L | ALL core + syntax skills |

---

## Batch Execution Plan (DEFINITIVE)

| Batch | Skills | Count | Dependencies | Notes |
|-------|--------|-------|-------------|-------|
| 1 | `core-architecture`, `syntax-document`, `syntax-fonts` | 3 | None | Foundation: architecture + document lifecycle + font system |
| 2 | `syntax-pages`, `syntax-text`, `syntax-images` | 3 | Batch 1 | Core syntax: pages + text + images |
| 3 | `syntax-drawing`, `syntax-forms`, `impl-form-filling` | 3 | Batch 1-2 | Drawing + forms + form workflow |
| 4 | `impl-merging`, `impl-pdf-generation`, `impl-page-layout` | 3 | Batch 1-3 | Implementation patterns |
| 5 | `errors-fonts`, `errors-loading`, `errors-common` | 3 | Batch 1-4 | Error handling |
| 6 | `agents-review`, `agents-project-scaffolder` | 2 | ALL above | Agent skills last |

**Total**: 17 skills across 6 batches.

---

## Per-Skill Agent Prompts

### Constants

```
PROJECT_ROOT = C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package
RESEARCH_FILE = C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\docs\research\vooronderzoek-pdflib.md
REQUIREMENTS_FILE = C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\REQUIREMENTS.md
REFERENCE_SKILL = C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md
```

---

### Batch 1

#### Prompt: pdflib-core-architecture

```
## Task: Create the pdflib-core-architecture skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-core\pdflib-core-architecture\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (API signatures for PDFDocument, PDFPage, PDFFont, PDFImage, PDFForm)
3. references/examples.md (working code examples)
4. references/anti-patterns.md (what NOT to do)

### Reference Format
Read and follow the structure of:
C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md

### YAML Frontmatter
---
name: pdflib-core-architecture
description: "Guides pdf-lib architecture including PDFDocument lifecycle, async-first API design, bottom-left coordinate system, key types, installation, and project setup. Activates when creating PDFs with pdf-lib, understanding pdf-lib project structure, or setting up a new pdf-lib project."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- pdf-lib 1.x architecture: pure JavaScript, no native dependencies, cross-platform
- Key types hierarchy: PDFDocument, PDFPage, PDFFont, PDFImage, PDFForm, PDFField
- Async-first design: which operations are async (embed, load, save) vs sync (draw methods)
- Coordinate system: bottom-left origin, Y increases upward, units in PDF points (1/72 inch)
- Installation & setup: npm install, imports, CDN, Deno, TypeScript support
- fontkit registration for custom fonts
- Platform support: Node.js, Browser, Deno, React Native

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 1: Architecture Overview
- Section 2: Installation & Setup

### Quality Rules
- English only
- SKILL.md < 500 lines; heavy content goes in references/
- Use ALWAYS/NEVER deterministic language, not "you should" or "consider"
- All code examples must be TypeScript with proper imports
- Include version annotations (pdf-lib 1.x)
- Include Critical Warnings section with NEVER rules
```

#### Prompt: pdflib-syntax-document

```
## Task: Create the pdflib-syntax-document skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-document\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (complete PDFDocument API: create, load, save, metadata, attach, addJavaScript)
3. references/examples.md (working code examples for all operations)
4. references/anti-patterns.md (common PDFDocument mistakes)

### Reference Format
Read and follow the structure of:
C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md

### YAML Frontmatter
---
name: pdflib-syntax-document
description: "Guides PDFDocument creation, loading, saving, metadata, and attachments in pdf-lib. Activates when creating new PDFs, loading existing PDFs, saving PDF documents, setting PDF metadata, or attaching files to PDFs."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- PDFDocument.create() with CreateOptions
- PDFDocument.load() with all LoadOptions (ignoreEncryption, parseSpeed, throwOnInvalidObject, updateMetadata, capNumbers)
- pdfDoc.save() with all SaveOptions (useObjectStreams, addDefaultPage, objectsPerTick, updateFieldAppearances)
- pdfDoc.saveAsBase64() with dataUri option
- pdfDoc.copy() and its limitations
- Document metadata: all getters (getTitle, getAuthor, etc.) and setters (setTitle with showInWindowTitleBar, setAuthor, etc.)
- pdfDoc.setLanguage()
- pdfDoc.attach() with AttachmentOptions
- pdfDoc.addJavaScript()
- PDFDocument properties (catalog, context, defaultWordBreaks, isEncrypted)

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 3: PDFDocument Lifecycle

### Quality Rules
- English only
- SKILL.md < 500 lines; heavy content goes in references/
- Use ALWAYS/NEVER deterministic language
- All code examples must use async/await correctly
- Include all method signatures with parameter types
- Include Critical Warnings (updateMetadata default behavior, copy() limitations)
```

#### Prompt: pdflib-syntax-fonts

```
## Task: Create the pdflib-syntax-fonts skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-fonts\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (PDFFont API, embedFont, embedStandardFont, StandardFonts enum)
3. references/examples.md (standard fonts, custom fonts, unicode, measuring)
4. references/anti-patterns.md (WinAnsi errors, missing fontkit, unicode failures)

### Reference Format
Read and follow the structure of:
C:\Users\Freek Heijting\Documents\GitHub\Tauri-2-Claude-Skill-Package\skills\source\tauri-core\tauri-core-architecture\SKILL.md

### YAML Frontmatter
---
name: pdflib-syntax-fonts
description: "Guides font embedding in pdf-lib including standard fonts, custom TTF/OTF fonts, fontkit registration, unicode support, font subsetting, and text measuring. Activates when embedding fonts, using custom fonts, handling unicode text, or measuring text dimensions in PDFs."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- StandardFonts enum: all 14 standard fonts with families
- embedFont() signature and all input types (StandardFonts, string, Uint8Array, ArrayBuffer)
- embedStandardFont() synchronous alternative
- EmbedFontOptions: subset, customName, features
- fontkit registration: @pdf-lib/fontkit installation and registerFontkit()
- WinAnsi encoding limitation: what characters fail with standard fonts
- Font compatibility matrix (Latin, accented, Cyrillic, CJK, Arabic, Emoji)
- PDFFont methods: widthOfTextAtSize, heightAtSize, sizeAtHeight, getCharacterSet, encodeText
- Font subsetting trade-offs

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 6: Font System

### Quality Rules
- English only
- SKILL.md < 500 lines; heavy content goes in references/
- Use ALWAYS/NEVER deterministic language
- CRITICAL: WinAnsi limitation must be prominently documented
- Include font compatibility matrix
- Include decision tree: when to use standard vs custom fonts
```

---

### Batch 2

#### Prompt: pdflib-syntax-pages

```
## Task: Create the pdflib-syntax-pages skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-pages\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (all page operation signatures)
3. references/examples.md (page manipulation examples)
4. references/anti-patterns.md (common page operation mistakes)

### YAML Frontmatter
---
name: pdflib-syntax-pages
description: "Guides page operations in pdf-lib including adding, inserting, removing, and getting pages, page dimensions, PageSizes enum, rotation, page boxes, and content transformations. Activates when adding pages, setting page sizes, rotating pages, or manipulating page dimensions."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- addPage (no args, [width,height], PDFPage), insertPage, removePage
- getPage, getPages, getPageCount, getPageIndices
- Dimensions: getSize, setSize, getWidth/setWidth, getHeight/setHeight
- PageSizes enum: all 55 sizes (A/B/C/RA/SRA series + NA sizes)
- Rotation: setRotation, getRotation (multiples of 90 only)
- Page boxes: media, crop, bleed, trim, art (get/set for each)
- Content transformations: scale, scaleContent, scaleAnnotations, translateContent
- Position cursor: moveTo, moveUp/Down/Left/Right, getPosition, resetPosition
- Page-level defaults: setFont, setFontSize, setFontColor, setLineHeight

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 4: Page Operations

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Include PageSizes reference table (at minimum A3-A5, Letter, Legal, Tabloid)
- Document coordinate system (bottom-left origin) in every relevant example
```

#### Prompt: pdflib-syntax-text

```
## Task: Create the pdflib-syntax-text skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-text\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (drawText options, text measurement methods)
3. references/examples.md (text positioning, multiline, centering, rotation)
4. references/anti-patterns.md (coordinate confusion, missing font, wrong color values)

### YAML Frontmatter
---
name: pdflib-syntax-text
description: "Guides text drawing in pdf-lib including drawText options, text positioning, multiline text, text wrapping, text measuring, and centering. Activates when drawing text on PDF pages, positioning text, wrapping text, measuring text width, or centering text."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- drawText() with all 13 options (x, y, font, size, color, opacity, rotate, xSkew, ySkew, lineHeight, maxWidth, wordBreaks, blendMode)
- Multiline text with \n and lineHeight
- Text wrapping with maxWidth and wordBreaks
- Page-level defaults (setFont, setFontSize, setFontColor, setLineHeight)
- Text measuring: widthOfTextAtSize, heightAtSize, sizeAtHeight
- Centering pattern (horizontal centering calculation)
- Color functions for text: rgb(), cmyk(), grayscale()
- Rotation helpers: degrees(), radians()

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 5: Text Operations

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- ALWAYS show bottom-left coordinate system in positioning examples
- Include centering example
- Include multiline example
```

#### Prompt: pdflib-syntax-images

```
## Task: Create the pdflib-syntax-images skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-images\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (embedPng, embedJpg, PDFImage API, drawImage options)
3. references/examples.md (embedding, scaling, aspect ratio, positioning)
4. references/anti-patterns.md (unsupported formats, aspect ratio distortion)

### YAML Frontmatter
---
name: pdflib-syntax-images
description: "Guides image embedding and drawing in pdf-lib including PNG and JPG embedding, image scaling, aspect ratio preservation, and drawImage options. Activates when embedding images in PDFs, drawing images on pages, scaling images, or working with image dimensions."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- embedPng() and embedJpg() signatures and input types
- PDFImage properties: width, height, ref
- PDFImage methods: scale(factor), scaleToFit(width, height), size()
- drawImage() with all 9 options (x, y, width, height, rotate, xSkew, ySkew, opacity, blendMode)
- Aspect ratio preservation: 3 patterns (scale, scaleToFit, manual calculation)
- Supported formats: ONLY PNG and JPG — NO GIF, BMP, TIFF, WebP, SVG
- embedPdf/embedPage/embedPages for page-as-image operations
- drawPage() for drawing embedded pages

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 7: Image Operations

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- CRITICAL: prominently document that ONLY PNG and JPG are supported
- Include all 3 aspect ratio preservation patterns
- Include anti-pattern for image stretching
```

---

### Batch 3

#### Prompt: pdflib-syntax-drawing

```
## Task: Create the pdflib-syntax-drawing skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-drawing\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (all draw* method signatures with complete option types)
3. references/examples.md (shapes, colors, SVG paths, styled drawing)
4. references/anti-patterns.md (color value range, coordinate confusion, ellipse xScale/yScale vs width/height)

### YAML Frontmatter
---
name: pdflib-syntax-drawing
description: "Guides drawing shapes in pdf-lib including rectangles, squares, circles, ellipses, lines, and SVG paths, plus color constructors and drawing options. Activates when drawing shapes on PDF pages, using colors, creating borders, or drawing SVG paths."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- drawRectangle, drawSquare, drawCircle, drawEllipse, drawLine, drawSvgPath — all with complete options
- Color constructors: rgb(r,g,b), cmyk(c,m,y,k), grayscale(g) — ALL 0.0-1.0 range
- BlendMode enum values
- LineCapStyle, LineJoinStyle
- Border options: borderColor, borderWidth, borderOpacity, borderDashArray, borderDashPhase
- Rotation via degrees() and radians()
- Drawing options reference table (which options apply to which shapes)
- SVG path syntax for drawSvgPath

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 8: Drawing Operations

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- CRITICAL: document that color values are 0-1, NOT 0-255
- CRITICAL: drawEllipse uses xScale/yScale (not width/height)
- CRITICAL: drawLine uses thickness (not borderWidth)
- Include shape decision tree
- Include drawing options cross-reference table
```

#### Prompt: pdflib-syntax-forms

```
## Task: Create the pdflib-syntax-forms skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-syntax\pdflib-syntax-forms\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (PDFForm API + all field type APIs)
3. references/examples.md (field access, creation, property manipulation)
4. references/anti-patterns.md (field not found, duplicate names, XFA issues)

### YAML Frontmatter
---
name: pdflib-syntax-forms
description: "Guides PDF form field operations in pdf-lib including accessing forms, field types (text, checkbox, radio, dropdown, option list, button, signature), creating fields, field properties, and form flattening. Activates when working with PDF forms, reading form fields, creating form fields, or flattening forms."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- PDFForm: getForm(), getFields(), getField(), getFieldMaybe()
- All field type getters: getTextField, getCheckBox, getRadioGroup, getDropdown, getOptionList, getButton, getSignature
- All field type creators: createTextField, createCheckBox, createRadioGroup, createDropdown, createOptionList, createButton
- PDFTextField: setText/getText, setFontSize, setAlignment, multiline, scrolling, maxLength, combing, password, addToPage, updateAppearances
- PDFCheckBox: check/uncheck, isChecked, addToPage
- PDFRadioGroup: select, getSelected, getOptions, addOptionToPage, mutual exclusion
- PDFDropdown: select, getSelected, addOptions, getOptions, setOptions, multiselect, editing, sorting
- PDFOptionList: select, getSelected, addOptions, clear, multiselect, sorting
- PDFButton: setImage, setFontSize, addToPage (note: takes text label)
- Common field properties: readOnly, required, exported, getName
- form.flatten() and form.updateFieldAppearances()
- XFA: hasXFA(), deleteXFA()
- Field management: removeField, markFieldAsDirty/Clean

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 9: Form System

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Include field type decision tree
- CRITICAL: document case-sensitive field names
- CRITICAL: document that default font (Helvetica) only supports Latin
```

#### Prompt: pdflib-impl-form-filling

```
## Task: Create the pdflib-impl-form-filling skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-impl\pdflib-impl-form-filling\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (form filling method signatures)
3. references/examples.md (complete form filling workflows)
4. references/anti-patterns.md (form filling mistakes, unicode in forms)

### YAML Frontmatter
---
name: pdflib-impl-form-filling
description: "Guides complete form filling workflows in pdf-lib including reading existing forms, filling text fields, checking boxes, selecting options, custom font support for unicode, and form flattening. Activates when filling PDF forms, creating interactive forms, flattening form fields, or handling unicode in form fields."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Step 1: Load PDF and get form (form = pdfDoc.getForm())
- Step 2: Enumerate fields (getFields + getName + constructor.name)
- Step 3: Fill fields by type (setText, check, select, setImage)
- Step 4: Unicode/custom font support (registerFontkit + embedFont + updateFieldAppearances)
- Step 5: Flatten (form.flatten()) or save editable
- Creating new forms workflow (createTextField, addToPage, etc.)
- Font decision tree for forms (ASCII → default OK; diacritics/unicode → custom font required)
- When to flatten decision tree
- Selective flattening workaround (removeField before flatten)
- Known limitations: appearance streams, restricted documents

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 9: Form System (form filling, flattening, unicode subsections)

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Include complete step-by-step workflow
- Include font decision tree for forms
- CRITICAL: ALWAYS enumerate fields first before filling (case-sensitive names)
- CRITICAL: ALWAYS register fontkit for non-Latin text in forms
```

---

### Batch 4

#### Prompt: pdflib-impl-merging

```
## Task: Create the pdflib-impl-merging skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-impl\pdflib-impl-merging\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (copyPages, addPage, insertPage, removePage signatures)
3. references/examples.md (merge, split, extract, reorder patterns)
4. references/anti-patterns.md (direct page addition, form field loss, file size bloat)

### YAML Frontmatter
---
name: pdflib-impl-merging
description: "Guides PDF merging, splitting, and page manipulation in pdf-lib including copying pages between documents, extracting page ranges, splitting into individual pages, and reordering pages. Activates when merging PDFs, splitting PDFs, extracting pages, or combining multiple PDF documents."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- copyPages() workflow: load source → copyPages on destination → addPage each copied page
- Merge multiple documents pattern (loop)
- Extract single page, extract page range
- Split into individual pages
- Page reordering (copyPages + insertPage)
- Known limitations: form fields lost during copy (#1205, #1587), blank pages (#1579, #1767), file size bloat (#1338)
- Decision tree for document manipulation operations

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 10: Document Manipulation

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- CRITICAL: ALWAYS use copyPages() — NEVER add pages directly from another document
- Include complete merge pattern function
- Include known limitations section
```

#### Prompt: pdflib-impl-pdf-generation

```
## Task: Create the pdflib-impl-pdf-generation skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-impl\pdflib-impl-pdf-generation\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (key API methods used in generation workflows)
3. references/examples.md (complete generation examples: invoice, report, multi-page)
4. references/anti-patterns.md (generation mistakes)

### YAML Frontmatter
---
name: pdflib-impl-pdf-generation
description: "Guides end-to-end PDF generation workflows in pdf-lib including multi-page documents, headers and footers, page numbering, table-like layouts, and document templates. Activates when generating PDFs from scratch, creating invoices, building reports, or producing multi-page documents."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- End-to-end generation workflow: create → addPage → embed fonts → draw content → save
- Multi-page documents: adding pages dynamically, content overflow detection
- Headers and footers pattern: draw on every page
- Page numbering pattern
- Table-like layout pattern (rows, columns, borders)
- Invoice/report template pattern
- Combining text + images + shapes on a single page
- Save and output patterns (Uint8Array, Base64, data URI)

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 3: PDFDocument Lifecycle (create/save)
- Section 5: Text Operations
- Section 7: Image Operations
- Section 8: Drawing Operations

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Include at least one complete generation example (invoice or report)
- ALWAYS use bottom-left coordinate system correctly
- ALWAYS show proper async/await
- Include content overflow detection pattern
```

#### Prompt: pdflib-impl-page-layout

```
## Task: Create the pdflib-impl-page-layout skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-impl\pdflib-impl-page-layout\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (positioning and measurement API methods)
3. references/examples.md (layout patterns: margins, centering, columns, flow)
4. references/anti-patterns.md (coordinate mistakes, layout anti-patterns)

### YAML Frontmatter
---
name: pdflib-impl-page-layout
description: "Guides page layout strategies in pdf-lib including coordinate system helpers, margin management, text centering, multi-column layouts, content flow across pages, and top-down positioning patterns. Activates when positioning content on PDF pages, calculating layout, centering elements, or managing content flow."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Coordinate system strategies: top-down helper (y = height - offset)
- Margin management pattern (contentArea = { x: margin, y: margin, width: pageWidth - 2*margin, height: pageHeight - 2*margin })
- Horizontal centering: (pageWidth - elementWidth) / 2
- Vertical centering: (pageHeight - elementHeight) / 2
- Multi-column layout pattern
- Text wrapping with maxWidth
- Content flow across pages: track currentY, detect overflow, add new page
- cursorY pattern for sequential content placement

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 4: Page Operations (dimensions, position cursor)
- Section 5: Text Operations (measuring, centering)

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- CRITICAL: ALWAYS document bottom-left origin in every coordinate example
- Include reusable helper patterns
- Include content flow / page break detection pattern
```

---

### Batch 5

#### Prompt: pdflib-errors-fonts

```
## Task: Create the pdflib-errors-fonts skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-errors\pdflib-errors-fonts\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (font-related method signatures relevant to errors)
3. references/examples.md (error reproduction and fix examples)
4. references/anti-patterns.md (comprehensive font anti-pattern catalog)

### YAML Frontmatter
---
name: pdflib-errors-fonts
description: "Diagnoses and fixes font-related errors in pdf-lib including WinAnsi encoding failures, missing fontkit registration, unicode character support issues, font embedding failures, and form field font problems. Activates when encountering font errors, WinAnsi encoding errors, unicode rendering issues, or fontkit problems."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- WinAnsi encoding error: "WinAnsi cannot encode X (0xNNNN)" — cause, fix
- Missing fontkit registration error — cause, fix
- Standard fonts unicode limitation — what fails, what works
- Font not applied to form fields — cause, fix (updateFieldAppearances)
- Diacritics failures with standard fonts
- Font compatibility matrix (standard vs custom)
- GitHub issues catalog (linked to real issues)
- Decision tree: diagnosing font errors

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 6: Font System (WinAnsi limitation)
- Section 11: Anti-Patterns & Common Errors (font errors)

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Each error must include: error message, cause, broken code, fixed code
- Include font compatibility matrix
- Include diagnostic decision tree
```

#### Prompt: pdflib-errors-loading

```
## Task: Create the pdflib-errors-loading skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-errors\pdflib-errors-loading\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (load options and error-related signatures)
3. references/examples.md (error reproduction and fix examples)
4. references/anti-patterns.md (loading anti-patterns)

### YAML Frontmatter
---
name: pdflib-errors-loading
description: "Diagnoses and fixes PDF loading errors in pdf-lib including encrypted PDF handling, malformed PDF recovery, ignoreEncryption usage, and load option configuration. Activates when encountering PDF loading errors, encrypted PDF errors, or malformed PDF parsing failures."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Encrypted PDF errors and ignoreEncryption option
- pdf-lib encryption limitations (no decryption engine, no password support)
- Malformed PDF errors and throwOnInvalidObject option
- updateMetadata side effects (auto-sets producer, creator, dates)
- Restricted document handling for form filling
- isEncrypted property
- capNumbers option
- parseSpeed option
- Decision tree: loading error diagnosis

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 3: PDFDocument Lifecycle (load options)
- Section 11: Anti-Patterns (encrypted PDFs, restricted documents)

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Each error must include: error context, cause, fix
- CRITICAL: document that ignoreEncryption does NOT decrypt content
```

#### Prompt: pdflib-errors-common

```
## Task: Create the pdflib-errors-common skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-errors\pdflib-errors-common\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (relevant API signatures)
3. references/examples.md (error reproduction and fix examples)
4. references/anti-patterns.md (comprehensive common mistakes catalog)

### YAML Frontmatter
---
name: pdflib-errors-common
description: "Diagnoses and fixes common pdf-lib mistakes including forgotten await, coordinate system confusion, unsupported image formats, cross-document page errors, form field name issues, color value range errors, and save option side effects. Activates when debugging pdf-lib code, encountering unexpected behavior, or reviewing pdf-lib code for correctness."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Forgotten await (embedFont, embedPng, save return Promises)
- Coordinate system confusion (bottom-left vs top-left)
- Unsupported image formats (only PNG and JPG)
- Cross-document page addition without copyPages
- Form field not found (case-sensitive, fully qualified names)
- Duplicate field name errors
- Color values 0-1 not 0-255
- copy() doesn't copy acroforms/outlines
- save() auto-adds blank page if empty
- save() auto-updates field appearances
- setKeywords takes string[], getKeywords returns string
- Page rotation only multiples of 90
- drawEllipse uses xScale/yScale not width/height
- drawLine uses thickness not borderWidth

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 11: Anti-Patterns & Common Errors (all non-font, non-loading errors)

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Each mistake: broken code → explanation → fixed code
- Include quick diagnostic checklist
- Organized by frequency/severity
```

---

### Batch 6

#### Prompt: pdflib-agents-review

```
## Task: Create the pdflib-agents-review skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-agents\pdflib-agents-review\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (all validation-relevant API patterns)
3. references/examples.md (before/after code review examples)
4. references/anti-patterns.md (complete anti-pattern checklist)

### YAML Frontmatter
---
name: pdflib-agents-review
description: "Validates generated pdf-lib code for correctness including async/await audit, coordinate system verification, font handling checks, import completeness, form handling validation, and anti-pattern detection. Activates when reviewing pdf-lib code, validating PDF generation code, or checking code for common pdf-lib mistakes."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Async/await audit checklist (all async methods listed)
- Import completeness check (required imports for each operation type)
- Coordinate system verification (bottom-left origin used correctly)
- Font handling check (standard font limitations, fontkit for custom, form fonts)
- Image format check (only PNG/JPG)
- Cross-document operation check (copyPages used, not direct page addition)
- Form field validation (enumerate first, case-sensitive names, flatten when appropriate)
- Color value range check (0-1 not 0-255)
- Save options review (updateFieldAppearances, addDefaultPage)
- Complete validation checklist (ordered by severity)

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 11: Anti-Patterns & Common Errors (ALL entries)
- Section 12: API Quick Reference

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Structure as a validation CHECKLIST that Claude can follow step-by-step
- Each check item: what to verify, why it matters, how to fix
- Include severity levels (CRITICAL, WARNING, INFO)
```

#### Prompt: pdflib-agents-project-scaffolder

```
## Task: Create the pdflib-agents-project-scaffolder skill

### Output Directory
C:\Users\Freek Heijting\Documents\GitHub\pdf-lib-Claude-Skill-Package\skills\source\pdflib-agents\pdflib-agents-project-scaffolder\

### Files to Create
1. SKILL.md (main skill file, <500 lines)
2. references/methods.md (setup and configuration API)
3. references/examples.md (complete project templates)
4. references/anti-patterns.md (setup mistakes)

### YAML Frontmatter
---
name: pdflib-agents-project-scaffolder
description: "Generates complete pdf-lib project structures including TypeScript configuration, dependency installation, fontkit setup, PDF generation boilerplate, and platform-specific variants. Activates when creating a new pdf-lib project, scaffolding PDF generation code, or setting up pdf-lib in an existing project."
license: MIT
compatibility: "Designed for Claude Code. Requires pdf-lib 1.x with TypeScript/JavaScript."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

### Scope (EXACT — do not exceed)
- Node.js project template (package.json, tsconfig.json, main file)
- Browser project template (HTML + script setup)
- Dependencies: pdf-lib, @pdf-lib/fontkit (when needed), @types/node
- TypeScript configuration for pdf-lib
- Basic PDF generation boilerplate (create, add page, embed font, draw text, save)
- Custom font setup boilerplate (fontkit registration)
- Form filling boilerplate
- Document merging boilerplate
- File output patterns (Node.js fs.writeFile, browser download)

### Research Sections to Read
From vooronderzoek-pdflib.md:
- Section 1: Architecture Overview
- Section 2: Installation & Setup
- Section 3: PDFDocument Lifecycle

### Quality Rules
- English only, SKILL.md < 500 lines, ALWAYS/NEVER language
- Include complete, runnable project templates
- ALWAYS include proper TypeScript types
- ALWAYS include async/await
- Include both Node.js and browser variants
```

---

## Appendix: Skill Directory Structure

```
skills/source/
├── pdflib-core/
│   └── pdflib-core-architecture/
│       ├── SKILL.md
│       └── references/
│           ├── methods.md
│           ├── examples.md
│           └── anti-patterns.md
├── pdflib-syntax/
│   ├── pdflib-syntax-document/
│   ├── pdflib-syntax-pages/
│   ├── pdflib-syntax-text/
│   ├── pdflib-syntax-fonts/
│   ├── pdflib-syntax-images/
│   ├── pdflib-syntax-drawing/
│   └── pdflib-syntax-forms/
├── pdflib-impl/
│   ├── pdflib-impl-form-filling/
│   ├── pdflib-impl-merging/
│   ├── pdflib-impl-pdf-generation/
│   └── pdflib-impl-page-layout/
├── pdflib-errors/
│   ├── pdflib-errors-fonts/
│   ├── pdflib-errors-loading/
│   └── pdflib-errors-common/
└── pdflib-agents/
    ├── pdflib-agents-review/
    └── pdflib-agents-project-scaffolder/
```
