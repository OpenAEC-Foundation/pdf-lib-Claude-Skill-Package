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

---

## L-004: WinAnsi Encoding is the #1 Error Source

- **Date**: 2026-03-19
- **Context**: Research revealed dozens of open GitHub issues related to standard font unicode limitations.
- **Finding**: Standard fonts (Helvetica, TimesRoman, Courier) use WinAnsi encoding which ONLY supports Latin-1 characters. Any non-Latin text (Cyrillic, CJK, Arabic, emoji) throws "WinAnsi cannot encode" errors. The ONLY fix is using custom fonts via @pdf-lib/fontkit. This must be the most prominent warning in font-related skills.

---

## L-005: Single-Technology Packages Complete Faster

- **Date**: 2026-03-19
- **Context**: pdf-lib is a single library vs multi-technology packages like Blender-Bonsai (73 skills).
- **Finding**: With 17 skills instead of 27-73, the entire package (Phase 1-7) was completed in a single session. The 7-phase methodology scales well — fewer skills means fewer batches and faster iteration. Embedding Phase 4 (topic research) into Phase 5 (creation) also saves time when the vooronderzoek is comprehensive enough.

---

## L-006: Comprehensive Vooronderzoek Eliminates Phase 4

- **Date**: 2026-03-19
- **Context**: Phase 4 (topic-specific research) was planned as separate per-skill research.
- **Finding**: When the Phase 2 vooronderzoek is thorough enough (9,251 words covering all API areas), Phase 4 can be embedded into Phase 5. Agents receive the vooronderzoek directly and extract what they need. This saved an entire phase of agent spawning.

---

## L-007: Agent Rate Limits Require Retry Strategy

- **Date**: 2026-03-19
- **Context**: 2 out of 3 agents in Batch 3 hit rate limits mid-execution.
- **Finding**: When running multiple agent batches in rapid succession, rate limits can block individual agents. Strategy: simply retry the failed agents in the next batch alongside remaining work. No data is lost — the agents that completed are fine.
