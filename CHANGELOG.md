# Changelog

## conventions.md

### v1.6 — 2026-05-06
- `## stable abbrevs`: whitelist → qualitative rule + non-exhaustive examples (widely established in technical/programming domains; ¬ invent if recognizability uncertain)

### v1.5 — 2026-05-06
- Removed Stella-specific abbrevs from `## stable abbrevs`: sh · lg · sid · vrs

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
- Audit: added `blank-after-header` check (MODERATE)
- Audit: `---` SUBAGENT PROMPT delimiters → `##` header (conventions compliance)
- Output format: version field updated to `"1.1"`

### v1.0 — 2026-05-06
- Initial release

## skills/test

### v1.0 — 2026-05-06
- Initial release

## skills/translate-prompt

### v1.2 — 2026-05-06
- Added `## Layout pass` (step 6): mechanical blank-after-header strip via regex before write
  Fixes audit subagents reporting violations without applying fixes to the output file

### v1.1 — 2026-05-06
- Pass 1: formulation patterns (model response scripts) protected from non-inferable removal
  Signals: "Formulation type :", explicit quoted agent speech defining output phrasing
  Test: "removing this changes agent output?" → ¬ inferable → keep

### v1.0 — 2026-05-06
- Initial release — 2-pass skill: Lakon translation → architectural restructuring
- Pass 2 patterns: Identity · Absolute isolation · HARD CONSTRAINTS · Principle · Pre-emit check ([ ] format)
- Audit: 2 additional CRITICAL checks (hard-constraints-missing · language-output) + 3 MODERATE (identity-missing · isolation-missing · preemit-format)

## prompts/translate-system.md

### v1.0 — 2026-05-06
- Initial release — full Lakon convention set for use without Claude Code (API / Claude.ai Projects)
