# pdf-lib Claude Skill Package

![Claude Code Ready](https://img.shields.io/badge/Claude_Code-Ready-blue?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCAyNCAyNCI+PHBhdGggZD0iTTEyIDJDNi40OCAyIDIgNi40OCAyIDEyczQuNDggMTAgMTAgMTAgMTAtNC40OCAxMC0xMFMxNy41MiAyIDEyIDJ6IiBmaWxsPSIjZmZmIi8+PC9zdmc+)
![pdf-lib 1.x](https://img.shields.io/badge/pdf--lib-1.x-FF0000?style=flat-square)
![TypeScript](https://img.shields.io/badge/TypeScript-Ready-3178C6?style=flat-square&logo=typescript&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)

**Deterministic Claude AI skills for pdf-lib PDF document creation and modification — TypeScript coverage.**

Built on the [Agent Skills](https://agentskills.org) open standard.

---

## Why This Exists

Without skills, Claude generates incorrect pdf-lib code:

```typescript
// Wrong — top-left origin assumption, missing await
const doc = PDFDocument.create();
page.drawText('Hello', { x: 50, y: 50 }); // y=50 is near BOTTOM, not top
```

With this skill package, Claude produces correct pdf-lib code:

```typescript
// Correct — proper async/await, bottom-left origin understood
const doc = await PDFDocument.create();
const page = doc.addPage(PageSizes.A4);
const { height } = page.getSize();
page.drawText('Hello', { x: 50, y: height - 50, size: 12 }); // Correct: top of page
```

---

## Current Progress

**Phase 1: Setup + Raw Masterplan** — 50% complete

| Phase | Status |
|-------|--------|
| 1. Setup + Raw Masterplan | IN PROGRESS |
| 2. Deep Research | NOT STARTED |
| 3. Masterplan Refinement | NOT STARTED |
| 4. Topic-Specific Research | NOT STARTED |
| 5. Skill Creation | NOT STARTED |
| 6. Validation | NOT STARTED |
| 7. Publication | NOT STARTED |

## Skill Categories

| Category | Description |
|----------|-------------|
| `syntax/` | API syntax, method signatures, type patterns |
| `impl/` | Step-by-step development workflows and integration guides |
| `errors/` | Error diagnosis, debugging patterns, anti-patterns |
| `core/` | Cross-cutting architecture, coordinate system, byte handling |
| `agents/` | Intelligent orchestration for PDF generation and review |

## Installation

### Claude Code

```bash
# Option 1: Clone the full package
git clone https://github.com/OpenAEC-Foundation/pdf-lib-Claude-Skill-Package.git
cp -r pdf-lib-Claude-Skill-Package/skills/source/ ~/.claude/skills/pdflib/

# Option 2: Add as git submodule
git submodule add https://github.com/OpenAEC-Foundation/pdf-lib-Claude-Skill-Package.git .claude/skills/pdflib
```

### Claude.ai (Web)

Upload individual SKILL.md files as project knowledge.

## Version Compatibility

| Technology | Versions | Notes |
|------------|----------|-------|
| pdf-lib | **1.x** | Primary target (1.17.x latest) |
| TypeScript | 4.x / 5.x | Type safety |
| Node.js | 14+ | Runtime support |
| Browsers | Modern | Full browser support |
| Deno | 1.x+ | Supported runtime |

## Methodology

This package is developed using the **7-phase research-first methodology**, proven across multiple skill packages:

1. **Setup + Raw Masterplan** — Project structure and governance files
2. **Deep Research** — Comprehensive source analysis of pdf-lib documentation, source code, and community resources
3. **Masterplan Refinement** — Skill inventory refinement based on research findings
4. **Topic-Specific Research** — Deep-dive per skill topic
5. **Skill Creation** — Deterministic skill files following Agent Skills standard
6. **Validation** — Correctness, completeness, and consistency checks
7. **Publication** — GitHub release and documentation

## Documentation

| Document | Purpose |
|----------|---------|
| [ROADMAP.md](ROADMAP.md) | Project status (single source of truth) |
| [REQUIREMENTS.md](REQUIREMENTS.md) | Quality guarantees and per-area requirements |
| [DECISIONS.md](DECISIONS.md) | Architectural decisions with rationale |
| [SOURCES.md](SOURCES.md) | Official reference URLs and verification rules |
| [WAY_OF_WORK.md](WAY_OF_WORK.md) | 7-phase development methodology |
| [LESSONS.md](LESSONS.md) | Lessons learned during development |
| [CHANGELOG.md](CHANGELOG.md) | Version history |

## Related Projects

| Project | Description |
|---------|-------------|
| [PDFjs Skill Package](https://github.com/OpenAEC-Foundation/PDFjs-Claude-Skill-Package) | Skills for PDF.js (Mozilla) PDF rendering/viewing |
| [ERPNext Skill Package](https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package) | 28 skills for ERPNext/Frappe development |
| [Tauri 2 Skill Package](https://github.com/OpenAEC-Foundation/Tauri-2-Claude-Skill-Package) | 27 skills for Tauri 2 desktop applications |
| [Blender-Bonsai Skill Package](https://github.com/OpenAEC-Foundation/Blender-Bonsai-ifcOpenshell-Sverchok-Claude-Skill-Package) | 73 skills for Blender, Bonsai, IfcOpenShell & Sverchok |
| [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) | Parent organization |

## License

[MIT](LICENSE)

---

Part of the [OpenAEC Foundation](https://github.com/OpenAEC-Foundation) ecosystem.
