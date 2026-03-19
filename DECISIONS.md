# DECISIONS

Architectural and process decisions with rationale. Each decision is numbered and immutable once recorded. New decisions may supersede old ones but old ones are never deleted.

---

## D-001: 7-Phase Research-First Methodology
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need a structured approach to build high-quality skills
**Decision**: Adopt the 7-phase methodology proven in the ERPNext, Blender-Bonsai, and Tauri 2 Skill Packages
**Rationale**: ERPNext project successfully produced 28 domain skills, Blender-Bonsai produced 73 skills, Tauri 2 produced 27 skills with this approach. Research-first prevents hallucinated content.
**Reference**: https://github.com/OpenAEC-Foundation/ERPNext_Anthropic_Claude_Development_Skill_Package/blob/main/WAY_OF_WORK.md

## D-002: Single Technology Package
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: pdf-lib is one technology — a JavaScript library for PDF creation/modification
**Decision**: No per-technology separation needed. All skills share the `pdflib-` prefix under a single `skills/source/` tree.
**Rationale**: pdf-lib is a single library. Splitting into sub-packages would add complexity without benefit. The TypeScript API is the only interface.

## D-003: English-Only Skills
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Team works primarily in Dutch, skills target international audience
**Decision**: ALL skill content in English only
**Rationale**: Skills are instructions for Claude, not end-user documentation. Claude reads English and responds in any language. Bilingual skills double maintenance with zero functional benefit. Proven in ERPNext, Blender, and Tauri projects.

## D-004: Claude Code Agent Tool for Orchestration
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to produce skills efficiently. Windows environment, no oa-cli available.
**Decision**: Use Claude Code Agent tool for parallel execution instead of oa-cli
**Rationale**: Windows environment does not support oa-cli (requires WSL/Linux). Claude Code Agent tool provides native parallelism within the Claude Code session. Simpler setup, no tmux/fcntl dependencies.

## D-005: MIT License
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to choose open-source license
**Decision**: MIT License
**Rationale**: Most permissive, maximizes adoption. Consistent with OpenAEC Foundation philosophy.

## D-006: ROADMAP.md as Single Source of Truth
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to track project status across multiple sessions and agents
**Decision**: ROADMAP.md is the ONLY place where project status is tracked
**Rationale**: Multiple status locations cause drift and confusion. Single source prevents "which is current?" questions. Proven in ERPNext, Blender, and Tauri projects.

## D-007: pdf-lib 1.x Only
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: pdf-lib has one major version (1.x) that is actively maintained
**Decision**: All code targets pdf-lib 1.x exclusively (latest 1.17.x). No pre-1.0 APIs.
**Rationale**: pdf-lib 1.x is the stable, production version. The API surface is well-defined. There is no v2 yet. All skills target the current stable release.

## D-008: Merge pdflib-core-setup into pdflib-core-architecture
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Phase 3 masterplan refinement — evaluating raw skill inventory
**Decision**: Merge standalone setup skill into architecture skill
**Rationale**: Setup is too thin standalone (just npm install + 2 imports). Architecture skill already covers key types and async design — adding installation/imports is natural.

## D-009: Merge pdflib-impl-metadata into pdflib-syntax-document
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Phase 3 masterplan refinement — evaluating raw skill inventory
**Decision**: Merge standalone metadata skill into document lifecycle skill
**Rationale**: Metadata is just 8 getters + 9 setters on PDFDocument. Not enough for a standalone skill. Document lifecycle skill already covers PDFDocument creation/load/save — metadata fits naturally.

## D-010: Reorder batches — forms + form-filling together
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Phase 3 masterplan refinement — batch planning
**Decision**: Place syntax-forms and impl-form-filling in the same batch
**Rationale**: Forms syntax and form-filling implementation are tightly coupled. Doing them in the same batch lets the impl skill reference the syntax skill directly.

## D-011: Embed Phase 4 into Phase 5
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Comprehensive vooronderzoek (9,251 words) made standalone topic research unnecessary
**Decision**: Skip separate Phase 4 — agents receive vooronderzoek directly during Phase 5
**Rationale**: When Phase 2 research is thorough enough, separate per-skill research adds overhead without new information. Agents extract what they need from the vooronderzoek. Documented in LESSONS.md L-006.

## D-012: WebFetch for Source Verification
**Date**: 2026-03-19
**Status**: ACTIVE
**Context**: Need to ensure all code examples match current pdf-lib API
**Decision**: Use WebFetch to verify code against official documentation during research and skill creation
**Rationale**: Training data may be outdated. WebFetch ensures latest API signatures are used. Prevents hallucinated API methods.
