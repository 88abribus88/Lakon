# Changelog

## conventions.md

### v1.4 — 2026-05-06
- Added `## file formats`: skills, prompts, memory docs → .md | .txt only; ¬ binary formats

### v1.3 — 2026-05-06
- `## tronc commun` renamed → `## shared core` (non-english header)
- Added to `## Lakon`: scope extended to all document types (¬ prompts/rules only)

### v1.2 — 2026-04-30
- Added `## expression patterns`: 5 structural patterns (actor explicit, scope-left, exception `~>`, priority `//`, validation gate `GATE[...]`)
- Added `## imperative weight`: 6-level scale (show-through → absolute)
- `**bold**` always removed — tested +2 tok overhead per occurrence
- `⇔` clarified: strict biconditional only; sufficient condition → use `→`

### v1.1 — 2026-04-27
- Initial stable version
- Verbatim floor formalized
- Stable symbol set: `→ | + - · ¬ ↑ ↓ ⇔ ∴ ∧`
- Stable abbrev list: tok, occ, sh, lg, sys, cfg, cmd, ref, vrs, sid, ts

## skills/translate

### v1.1 — 2026-05-06
- Layout rule: strip blank lines between `##` header + first content line
- Translation rule: inline prose enumeration "A, B and C" → `A · B · C`

### v1.0 — 2026-05-06
- Initial release

## skills/test

### v1.0 — 2026-05-06
- Initial release

## prompts/translate-system.md

### v1.0 — 2026-05-06
- Initial release — full Lakon convention set for use without Claude Code (API / Claude.ai Projects)
