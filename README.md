# pdf-lib Claude Skill Package

![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)
![Skills: 17](https://img.shields.io/badge/Skills-17-blue?style=flat-square)
![pdf-lib 1.x](https://img.shields.io/badge/pdf--lib-1.x-FF0000?style=flat-square)
![TypeScript](https://img.shields.io/badge/TypeScript-Ready-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Claude Code Ready](https://img.shields.io/badge/Claude_Code-Ready-blue?style=flat-square)

Deterministic skill package that teaches Claude to write correct, version-aware pdf-lib code for creating and modifying PDF documents.

Built on the [Agent Skills](https://agentskills.org) open standard.

---

## What This Does

Skill packages are structured knowledge files that give Claude deterministic, verified guidance for specific technologies. Instead of relying on training data that may be outdated or imprecise, Claude reads these skills at runtime and produces correct code every time.

**Without skills**, Claude generates incorrect pdf-lib code:

```typescript
// Wrong -- top-left origin assumption, missing await
const doc = PDFDocument.create();
page.drawText('Hello', { x: 50, y: 50 }); // y=50 is near BOTTOM, not top
```

**With this skill package**, Claude produces correct pdf-lib code:

```typescript
// Correct -- proper async/await, bottom-left origin understood
const doc = await PDFDocument.create();
const page = doc.addPage(PageSizes.A4);
const { height } = page.getSize();
page.drawText('Hello', { x: 50, y: height - 50, size: 12 }); // Correct: top of page
```

---

## Installation

### Claude Code

```bash
# Clone into your Claude skills directory
git clone https://github.com/OpenAEC-Foundation/pdf-lib-Claude-Skill-Package.git ~/.claude/skills/pdf-lib
```

Alternative: add as a git submodule in your project:

```bash
git submodule add https://github.com/OpenAEC-Foundation/pdf-lib-Claude-Skill-Package.git .claude/skills/pdf-lib
```

### Claude.ai (Web)

Upload individual `SKILL.md` files from `skills/source/` as project knowledge.

---

## Skills Overview

### Core (1 skill)

| Skill | Description |
|-------|-------------|
| `pdflib-core-architecture` | Architecture overview, key types, async-first design, coordinate system, installation and setup |

### Syntax (7 skills)

| Skill | Description |
|-------|-------------|
| `pdflib-syntax-document` | PDFDocument create, load, save, copy; load/save options; metadata; attachments |
| `pdflib-syntax-pages` | Add, insert, remove pages; dimensions; rotation; PageSizes enum; page boxes; transformations |
| `pdflib-syntax-text` | drawText with all options; multiline text; page-level defaults; text measuring |
| `pdflib-syntax-fonts` | Standard fonts, custom font embedding, fontkit registration, WinAnsi encoding, unicode support |
| `pdflib-syntax-images` | Embed PNG/JPG; PDFImage properties; scaling; drawImage with all options |
| `pdflib-syntax-drawing` | Rectangles, circles, lines, SVG paths; color constructors (rgb, cmyk, grayscale); blend modes |
| `pdflib-syntax-forms` | PDFForm API; all field types (text, checkbox, radio, dropdown, button); flatten; XFA handling |

### Implementation (4 skills)

| Skill | Description |
|-------|-------------|
| `pdflib-impl-form-filling` | Complete form filling workflow; unicode support; flattening; font decisions for forms |
| `pdflib-impl-merging` | Copy pages between documents; merge, split, extract, reorder pages; known limitations |
| `pdflib-impl-pdf-generation` | End-to-end PDF generation; multi-page documents; headers/footers; table layouts; invoices |
| `pdflib-impl-page-layout` | Coordinate system strategies; margins; multi-column layout; text wrapping; content flow |

### Error Handling (3 skills)

| Skill | Description |
|-------|-------------|
| `pdflib-errors-fonts` | WinAnsi encoding errors; missing fontkit; font embedding failures; diacritics issues |
| `pdflib-errors-loading` | Encrypted PDFs; malformed PDFs; ignoreEncryption; restricted document handling |
| `pdflib-errors-common` | Forgotten await; coordinate confusion; unsupported formats; color values 0-1 vs 0-255 |

### Agents (2 skills)

| Skill | Description |
|-------|-------------|
| `pdflib-agents-review` | Validation checklist for generated pdf-lib code; async audit; anti-pattern detection |
| `pdflib-agents-project-scaffolder` | Generate complete pdf-lib project; TypeScript setup; Node.js and browser boilerplate |

---

## Quick Start

Once installed, Claude automatically uses the relevant skills when you ask it to work with pdf-lib. For example:

**Prompt**: "Create a PDF invoice with company logo, header, line items table, and total"

Claude will reference `pdflib-impl-pdf-generation`, `pdflib-syntax-text`, `pdflib-syntax-images`, and `pdflib-impl-page-layout` to produce correct, production-ready code:

```typescript
import { PDFDocument, StandardFonts, rgb, PageSizes } from 'pdf-lib';
import * as fs from 'fs';

const doc = await PDFDocument.create();
const page = doc.addPage(PageSizes.A4);
const { width, height } = page.getSize();
const font = await doc.embedFont(StandardFonts.Helvetica);
const boldFont = await doc.embedFont(StandardFonts.HelveticaBold);

// Logo
const logoBytes = fs.readFileSync('logo.png');
const logo = await doc.embedPng(logoBytes);
const logoDims = logo.scaleToFit(120, 60);
page.drawImage(logo, { x: 50, y: height - 80, ...logoDims });

// Title
page.drawText('INVOICE', {
  x: width - 200, y: height - 60,
  font: boldFont, size: 24, color: rgb(0.2, 0.2, 0.2),
});

// ... line items, totals, footer

const pdfBytes = await doc.save();
fs.writeFileSync('invoice.pdf', pdfBytes);
```

---

## Technology Coverage

| Area | What Is Covered |
|------|-----------------|
| Document lifecycle | Create, load (with options), save, saveAsBase64, copy |
| Pages | Add, insert, remove, resize, rotate; 55 standard page sizes |
| Text | Drawing, measuring, multiline, centering, page-level defaults |
| Fonts | 14 standard fonts, custom TTF/OTF embedding, fontkit, unicode |
| Images | PNG and JPG embedding, scaling, aspect ratio preservation |
| Drawing | Rectangles, circles, ellipses, lines, SVG paths; colors; blend modes |
| Forms | All field types, form filling, flattening, XFA handling |
| Merging | Copy pages, merge documents, split, extract, reorder |
| Page layout | Coordinate system, margins, columns, text wrapping, content flow |
| Error handling | Font errors, loading errors, common pitfalls with solutions |

**Version**: pdf-lib 1.x (1.17.x latest) | TypeScript 4.x/5.x | Node.js 14+ | Modern browsers | Deno 1.x+

---

## Project Structure

```
pdf-lib-Claude-Skill-Package/
├── CLAUDE.md                          # Project instructions for Claude
├── ROADMAP.md                         # Project status (source of truth)
├── REQUIREMENTS.md                    # Quality guarantees
├── DECISIONS.md                       # Architectural decisions
├── SOURCES.md                         # Official reference URLs
├── WAY_OF_WORK.md                     # Development methodology
├── LESSONS.md                         # Lessons learned
├── CHANGELOG.md                       # Version history
├── LICENSE                            # MIT License
├── docs/
│   ├── masterplan/                    # Execution plan
│   └── research/                      # Research documents and fragments
└── skills/
    └── source/
        ├── pdflib-core/               # Architecture and setup (1 skill)
        ├── pdflib-syntax/             # API syntax and patterns (7 skills)
        ├── pdflib-impl/               # Implementation workflows (4 skills)
        ├── pdflib-errors/             # Error handling and debugging (3 skills)
        └── pdflib-agents/             # Intelligent orchestration (2 skills)
```

---

## Contributing

Contributions are welcome. Please follow these guidelines:

1. All skill content must be in English
2. Use deterministic language (ALWAYS/NEVER, not "you might consider")
3. Every code example must be TypeScript with proper type annotations
4. Verify all API references against the [official pdf-lib documentation](https://pdf-lib.js.org/)
5. SKILL.md files must stay under 500 lines
6. Follow the existing skill structure (YAML frontmatter, Quick Reference, Decision Trees, Patterns)

---

## Related Projects

| Project | Skills | Description |
|---------|--------|-------------|
| [ERPNext Skill Package](https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package) | 28 | ERPNext/Frappe development |
| [Tauri 2 Skill Package](https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package) | 27 | Tauri 2 desktop applications |
| [Blender-Bonsai Skill Package](https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package) | 73 | Blender, Bonsai, IfcOpenShell, Sverchok |
| [PDFjs Skill Package](https://github.com/OpenAEC-Foundation/PDFjs-Claude-Skill-Package) | -- | PDF.js (Mozilla) rendering/viewing |

---

## License

[MIT](LICENSE)

---

Built by the [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) with [Claude Code](https://claude.ai/claude-code).
