# Lessons Learned

Observations and findings captured during skill package development.

---

## L-001: Single Technology Simplifies Package Structure

- **Date**: 2026-03-19
- **Context**: pdf-lib is one library with a single TypeScript API surface.
- **Finding**: No need for per-technology separation. `skills/source/pdflib-{category}/` is sufficient. The entire package can use a flat category structure without layered technology namespacing.

---

## L-002: Bottom-Left Coordinate System is the Primary Pitfall

- **Date**: 2026-03-19
- **Context**: pdf-lib uses the PDF specification's coordinate system with origin at bottom-left.
- **Finding**: Every drawing-related skill MUST document that coordinates start at bottom-left, not top-left like HTML/CSS. This is the most common source of confusion and incorrect positioning. Claude frequently assumes top-left origin without explicit instruction.

---

## L-003: Async-First API Design

- **Date**: 2026-03-19
- **Context**: All pdf-lib operations (create, load, save, embed) are asynchronous.
- **Finding**: Every code example MUST use async/await. Forgetting to await operations is a common mistake that leads to undefined values and silent failures. Skills must emphasize this pattern consistently.
