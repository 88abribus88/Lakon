---
name: translate
description: |
  Translate any source doc to Lakon + verify output via cold-start audit.
  Triggers: "translate", "lakon this", "traduis en lakon", file path provided.
version: "1.0"
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
3. Estimate src tok: `python3 -c "import tiktoken; enc=tiktoken.get_encoding('cl100k_base'); txt=open('[path]').read(); print(len(enc.encode(txt)))"`
4. Apply Lakon (§ Translation)
5. Write output file (§ Output format)
6. Estimate output tok (same cmd on output file)
7. Run audit subagent cold-start (§ Audit)
8. Output rapport (§ Livrable)

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
- Layout: remove `---` horiz rules | `*italic*` prose | `**bold**` prose | HTML comments | emoji
- Symbols: apply stable symbol substitutions where unambiguous
- ¬ root truncation; ¬ invented abbrevs
- ¬ inline-merge list items
- Source language ≠ English → translate content to English as part of Lakon pass

## Output format

```yaml
---
source: [path]
translated_at: [YYYY-MM-DDTHH:MM:SSZ]
skill: skills/translate
version: "1.0"
---
```
+ translated content

Filename: `[basename]_lakon.md`

## Audit

After write, spawn cold-start subagent.
Subagent receives: output file path + `{root}/conventions.md` path.
Subagent has no prior context — instructions below are complete.

---
SUBAGENT PROMPT (cold-start audit):

Read `conventions.md` first, then read the translated file.

Run these checks on the translated file. Skip: code blocks, backtick spans, YAML frontmatter identifier fields.

CRITICAL:
- `neg-connector`: ¬ used as grammatical connector vs negation. Signal: ¬ followed by noun/clause it does not negate.
- `biconditional-misuse`: ⇔ for sufficient condition (→ correct). Signal: A ⇔ B where B ¬ implied by A.
- `verbatim-floor`: path, cmd, URL, UUID, hex modified (spacing, truncation, paraphrase).

MODERATE:
- `bold-label`: `**bold**` on prose/labels. Exception: numeric thresholds, measurements, config values.
- `italic-prose`: `*italic*` on prose outside code/backticks.
- `horiz-rule`: standalone `---` (¬ YAML delimiter).
- `non-english`: non-English prose outside verbatim content. Exception: source proper nouns.
- `inline-merge`: `- A, B` where A + B are distinct parallel entities.
- `multiline-cmd`: shell cmd spanning multiple lines outside fenced code block.

Report format:
```
[translate-audit][rapport]
[filepath] — N violations
  L[n] CRITICAL  [check-id]  [excerpt] → [proposed fix]
  L[n] MODERATE  [check-id]  [excerpt] → [proposed fix]
[filepath] — 0 violations ✓
Summary: [C] critical | [M] moderate
```
---

0 violations → continue. Violations present → include audit rapport in § Livrable.

## Livrable

`[translate][rapport]`

```
[translate][rapport]
Source:     [path] ([format])
Output:     [output_path]
Tokens:     [src_tok] → [out_tok] (-X%)
Audit:      [0 violations ✓ | C critical + M moderate — see below]

[audit rapport if violations]
```
