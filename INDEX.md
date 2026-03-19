# pdf-lib Skill Package — Skill Index

## How to Use
Add skills to your project by copying the skill directory to your `.claude/skills/` folder or referencing them directly.

## Skills (17 total)

### Core (1 skill)
| Skill | Description | Path |
|-------|-------------|------|
| pdflib-core-architecture | pdf-lib architecture including PDFDocument lifecycle, async-first API design, bottom-left coordinate system, key types, installation, and project setup | `skills/source/pdflib-core/pdflib-core-architecture/` |

### Syntax (7 skills)
| Skill | Description | Path |
|-------|-------------|------|
| pdflib-syntax-document | PDFDocument creation, loading, saving, metadata, and attachments | `skills/source/pdflib-syntax/pdflib-syntax-document/` |
| pdflib-syntax-pages | Page operations including adding, inserting, removing, page dimensions, PageSizes enum, rotation, and page boxes | `skills/source/pdflib-syntax/pdflib-syntax-pages/` |
| pdflib-syntax-fonts | Font embedding including standard fonts, custom TTF/OTF fonts, fontkit registration, unicode support, and text measuring | `skills/source/pdflib-syntax/pdflib-syntax-fonts/` |
| pdflib-syntax-text | Text drawing including drawText options, text positioning, multiline text, wrapping, measuring, and centering | `skills/source/pdflib-syntax/pdflib-syntax-text/` |
| pdflib-syntax-images | Image embedding and drawing including PNG/JPG embedding, image scaling, aspect ratio preservation, and drawImage options | `skills/source/pdflib-syntax/pdflib-syntax-images/` |
| pdflib-syntax-drawing | Drawing shapes including rectangles, circles, ellipses, lines, SVG paths, color constructors, and drawing options | `skills/source/pdflib-syntax/pdflib-syntax-drawing/` |
| pdflib-syntax-forms | PDF form field operations including field types, creating fields, field properties, and form flattening | `skills/source/pdflib-syntax/pdflib-syntax-forms/` |

### Implementation (4 skills)
| Skill | Description | Path |
|-------|-------------|------|
| pdflib-impl-pdf-generation | End-to-end PDF generation workflows including multi-page documents, headers/footers, page numbering, and templates | `skills/source/pdflib-impl/pdflib-impl-pdf-generation/` |
| pdflib-impl-form-filling | Complete form filling workflows including reading forms, filling fields, custom font support, and flattening | `skills/source/pdflib-impl/pdflib-impl-form-filling/` |
| pdflib-impl-merging | PDF merging, splitting, and page manipulation including copying pages, extracting ranges, and reordering | `skills/source/pdflib-impl/pdflib-impl-merging/` |
| pdflib-impl-page-layout | Page layout strategies including coordinate helpers, margins, centering, multi-column layouts, and content flow | `skills/source/pdflib-impl/pdflib-impl-page-layout/` |

### Error Handling (3 skills)
| Skill | Description | Path |
|-------|-------------|------|
| pdflib-errors-fonts | Font-related error diagnosis including WinAnsi encoding failures, missing fontkit, unicode issues, and embedding errors | `skills/source/pdflib-errors/pdflib-errors-fonts/` |
| pdflib-errors-loading | PDF loading error diagnosis including encrypted PDF handling, malformed PDF recovery, and load options | `skills/source/pdflib-errors/pdflib-errors-loading/` |
| pdflib-errors-common | Common pdf-lib mistakes including forgotten await, coordinate confusion, unsupported formats, and anti-patterns | `skills/source/pdflib-errors/pdflib-errors-common/` |

### Agents (2 skills)
| Skill | Description | Path |
|-------|-------------|------|
| pdflib-agents-review | Validates pdf-lib code for correctness including async/await audit, coordinate verification, and anti-pattern detection | `skills/source/pdflib-agents/pdflib-agents-review/` |
| pdflib-agents-project-scaffolder | Generates complete pdf-lib project structures including TypeScript config, dependencies, fontkit setup, and boilerplate | `skills/source/pdflib-agents/pdflib-agents-project-scaffolder/` |
