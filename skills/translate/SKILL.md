---
name: translate
description: |
  Translate any source doc to Lakon + verify output via cold-start audit.
  Triggers: "translate", "lakon this", "traduis en lakon", file path provided.
version: "1.2"
trigger: explicit
executor: main-agent
---

## Input
- `path` — abs path to source doc (.docx | .pdf | .md | .txt)
- `output` — abs path for output file (default: `[source-dir]/[basename]_lakon.md`)
- `{root}` — repo root (contains `conventions.md`)

## Steps
1. Resolve `output` path (§ Input)
2. Extract source text (§ Extraction)
3. Apply Lakon (§ Translation) — keep extracted text as `src_text`, translated as `out_text`
4. Layout pass on `out_text` (§ Layout pass)
5. Estimate src tok on `src_text`:
   ```python
   import tiktoken; enc=tiktoken.get_encoding('cl100k_base'); print(len(enc.encode(src_text)))
   ```
6. Write output file (§ Output format)
7. Estimate output tok (same cmd on output file content)
8. Run audit subagent cold-start (§ Audit)
9. Output report (§ Deliverable)

## Extraction
`.md` | `.txt` → read directly.

`.docx`:
```python
from docx import Document
doc = Document(path)
text = '\n'.join(p.text for p in doc.paragraphs if p.text.strip())
```

`.pdf`:
```python
from pdfminer.high_level import extract_text
text = extract_text(path)
```

Other extension → signal unsupported format, exit.

## Translation
Apply Lakon per `{root}/conventions.md`. All rules apply:
- Verbatim floor: paths, cmds, IPs, config values, code blocks, URLs, identifiers — ¬ translate
- Private rule: strip `<private>...</private>` → semantic descriptor
- Layout: remove `---` horiz rules | `*italic*` prose | `**bold**` prose | HTML comments | emoji | blank lines between `##` header + first content line
- Symbols: apply stable symbol substitutions where unambiguous
- Inline prose enumeration: "A, B and C" → `A · B · C`
- ¬ root truncation; ¬ invented abbrevs
- ¬ inline-merge list items
- Source language ≠ English → translate content to English as part of Lakon pass

## Layout pass
Apply mechanically on `out_text` before write — strips layout violations that survive the translation pass.

```python
import re
out_text = re.sub(r'(^#{1,6} .+)\n\n', r'\1\n', out_text, flags=re.MULTILINE)
```

¬ touch YAML frontmatter.

## Output format
```yaml
---
source: [path]
translated_at: [YYYY-MM-DDTHH:MM:SSZ]   ← generate via: python3 -c "from datetime import datetime,timezone; print(datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%SZ'))"
skill: skills/translate
version: "1.2"
---
```
+ translated content

Filename: `[basename]_lakon.md`

## Audit
After write, spawn cold-start subagent.
Main agent substitutes `[CONVENTIONS_PATH]` = `{root}/conventions.md` and `[OUTPUT_PATH]` = output file path before passing prompt below.
Subagent has no prior context — instructions below are complete.

## SUBAGENT PROMPT (cold-start audit)
Read `[CONVENTIONS_PATH]` first, then read `[OUTPUT_PATH]`.

Run these checks on the translated file. Skip: code blocks, backtick spans, YAML frontmatter identifier fields.

CRITICAL:
- `neg-connector`: ¬ used as grammatical connector vs negation. Signal: ¬ followed by noun/clause it does not negate.
- `biconditional-misuse`: ⇔ for sufficient condition (→ correct). Signal: A ⇔ B where B ¬ implied by A.
- `verbatim-floor`: path, cmd, URL, UUID, hex modified (spacing, truncation, paraphrase).

MODERATE:
- `bold-label`: `**bold**` on prose/labels. Exception: numeric thresholds, measurements, config values.
- `italic-prose`: `*italic*` on prose outside code/backticks.
- `horiz-rule`: standalone `---` (¬ YAML delimiter).
- `blank-after-header`: blank line between `##` header + first content line.
- `non-english`: non-English prose outside verbatim content. Exception: source proper nouns.
- `inline-merge`: `- A, B` where A + B are distinct parallel entities.
- `multiline-cmd`: shell cmd spanning multiple lines outside fenced code block.

Report format:
```
[translate-audit][report]
[filepath] — N violations
  L[n] CRITICAL  [check-id]  [excerpt] → [proposed fix]
  L[n] MODERATE  [check-id]  [excerpt] → [proposed fix]
[filepath] — 0 violations ✓
Summary: [C] critical | [M] moderate
```

0 violations → continue. Violations present → include audit report in § Deliverable.

## Deliverable
`[translate][report]`

```
[translate][report]
Source:     [path] ([format])
Output:     [output_path]
Tokens:     [src_tok] → [out_tok] (-X%)
Audit:      [0 violations ✓ | C critical + M moderate — see below]

[audit report if violations]
```
