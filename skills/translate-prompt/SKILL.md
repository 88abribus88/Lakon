---
name: translate-prompt
description: |
  Translate a prompt to Lakon + architecturally restructure it.
  Two-pass: Lakon translation → architectural restructuring.
  Triggers: "translate-prompt", "prompt to lakon", file path provided with translate-prompt intent.
version: "1.0"
trigger: explicit
executor: main-agent
---

## Input
- `path` — abs path to source prompt (.md | .txt)
- `output` — abs path for output file (default: `[source-dir]/[basename]_lakon.md`)
- `{root}` — repo root (contains `conventions.md`)

## Steps
1. Resolve `output` path (§ Input)
2. Read source prompt
3. GATE[agent: source contains behavioral rules or identity definition] → proceed; ¬ behavioral prompt → signal, recommend `skills/translate` instead, exit
4. Pass 1 — Lakon (§ Pass 1)
5. Pass 2 — Architecture (§ Pass 2)
6. Estimate tok: src + output
   ```python
   import tiktoken; enc=tiktoken.get_encoding('cl100k_base'); print(len(enc.encode(text)))
   ```
7. Write output (§ Output format)
8. Run audit subagent cold-start (§ Audit)
9. Output rapport (§ Deliverable)

## Pass 1
Apply full Lakon per `{root}/conventions.md`. All rules apply — same as `skills/translate § Translation`.

## Pass 2
Restructure the Pass 1 output. Apply all transformations below. ¬ add content absent from source.

### Identity
Prompt defines a role or behavioral identity → extract as `## Identity` section.
Format: declarative sentence per axiom. Pattern: `[Role]: [what it does]. ¬ [what it is not].`
Example: `Transformer: text → English Lakon. ¬ assistant. ¬ agent. ¬ interlocutor.`

### Absolute isolation
Prompt specifies how to treat input (material to transform, ¬ message to respond to) → extract as `## Absolute isolation`.
Format: scenario → action pairs. Pattern: `Input [scenario] → [action], ¬ [forbidden action].`
One line per scenario. Final line: `¬ greet · ¬ confirm · ¬ explain · ¬ comment · ¬ add.`

### HARD CONSTRAINTS
Move all strong + absolute rules → `## HARD CONSTRAINTS` block.
Signals: "always", "never", "must", "character-for-character", unconditional scope, violation = semantic error.
Group under labeled sub-headers if ≥2 distinct domains (e.g. verbatim floor · symbol safety · output language).

### Principle
Prompt contains meta-statement on nature of the transformation → extract as `## Principle`.
1 sentence max. Contrast with related-but-wrong approach.
Example: `Translator preserves intent across form change. Compressor reduces size. Lakon = former.`

### Pre-emit check
Prompt contains review/validation step → convert to `## Pre-emit check (silent — ¬ output)`.
Format: `[ ] [check in imperative voice]` — one atomic check per line.
Final line: `Fail → correct + re-check → emit.`

### Section order
Normalize to:
1. `## Identity`
2. `## Absolute isolation`
3. `## HARD CONSTRAINTS`
4. Core rules sections (from source, Lakon-translated)
5. `## Principle`
6. `## Pre-emit check`
7. `## Output`

## Output format
```yaml
---
source: [path]
translated_at: [YYYY-MM-DDTHH:MM:SSZ]   ← python3 -c "from datetime import datetime,timezone; print(datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%SZ'))"
skill: skills/translate-prompt
version: "1.0"
---
```
+ restructured content

Filename: `[basename]_lakon.md`

## Audit
After write, spawn cold-start subagent.
Main agent substitutes `[CONVENTIONS_PATH]` = `{root}/conventions.md` and `[OUTPUT_PATH]` = output file path before passing prompt below.
Subagent has no prior context — instructions below are complete.

## SUBAGENT PROMPT (cold-start audit)
Read `[CONVENTIONS_PATH]` first, then read `[OUTPUT_PATH]`.

Run checks on the translated file. Skip: code blocks, backtick spans, YAML frontmatter identifier fields.

CRITICAL:
- `neg-connector`: ¬ used as grammatical connector vs negation. Signal: ¬ followed by noun/clause it does not negate.
- `biconditional-misuse`: ⇔ for sufficient condition (→ correct). Signal: A ⇔ B where B ¬ implied by A.
- `verbatim-floor`: path, cmd, URL, UUID, hex modified (spacing, truncation, paraphrase).
- `hard-constraints-missing`: strong/absolute rules ¬ in `## HARD CONSTRAINTS` block.
- `language-output`: output language rule ¬ in `## HARD CONSTRAINTS` (must be unconditional).

MODERATE:
- `bold-label`: `**bold**` on prose/labels. Exception: numeric thresholds, measurements, config values.
- `italic-prose`: `*italic*` on prose outside code/backticks.
- `horiz-rule`: standalone `---` (¬ YAML delimiter).
- `blank-after-header`: blank line between `##` header + first content line.
- `non-english`: non-English prose outside verbatim content. Exception: source proper nouns.
- `identity-missing`: `## Identity` absent from output.
- `isolation-missing`: `## Absolute isolation` absent from output.
- `preemit-format`: pre-emit checks ¬ in `[ ]` checkbox format.

Report format:
```
[translate-prompt-audit][report]
[filepath] — N violations
  L[n] CRITICAL  [check-id]  [excerpt] → [proposed fix]
  L[n] MODERATE  [check-id]  [excerpt] → [proposed fix]
[filepath] — 0 violations ✓
Summary: [C] critical | [M] moderate
```

0 violations → continue. Violations → include report in § Deliverable.

## Deliverable
`[translate-prompt][rapport]`

```
[translate-prompt][rapport]
Source:     [path] ([format])
Output:     [output_path]
Tokens:     [src_tok] → [out_tok] (-X%)
Audit:      [0 violations ✓ | C critical + M moderate — see below]

[audit report if violations]
```
