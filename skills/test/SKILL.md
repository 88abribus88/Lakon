---
name: test
description: |
  Lakon test protocol: token gain, cold-start info loss, comprehension, ambiguity.
  Triggers: "test lakon", "run lakon test", "protocole test lakon".
version: "1.0"
trigger: explicit
executor: main-agent
---

## Input

- `source` — abs path to original doc (pre-translation)
- `translated` — abs path to Lakon-translated `.md` (default: `[source-dir]/[basename]_lakon.md`)
- `{root}` — repo root (contains `conventions.md`)

`source` required. `translated` absent → derive from `source` basename.

## Steps

1. Verify both files exist; exit with error if absent
2. Test A — Token gain (§ Test A)
3. Test B — Cold-start info loss (§ Test B)
4. Test C — Comprehension (§ Test C)
5. Test D — Ambiguity (§ Test D)
6. Output rapport (§ Livrable)

## Test A — Token gain

```python
import tiktoken
enc = tiktoken.get_encoding('cl100k_base')
src = len(enc.encode(open('SOURCE_PATH').read()))
out = len(enc.encode(open('TRANSLATED_PATH').read()))
print(f"src={src} out={out} gain={round((1 - out/src)*100, 1)}%")
```

Record: `src_tok`, `out_tok`, `gain_pct`.
Benchmark: Lakon sweet spot = -20% rules | -39% prose. Signal if gain < -5% (possible over-compression) or gain > -45% (unexpected).

## Test B — Cold-start info loss

Generate N questions from source content.
N calibration:
- Source ≤ 500 tok → 5 questions
- 500–2000 tok → 10 questions
- 2000–5000 tok → 15 questions
- > 5000 tok → 20 questions

Question types (distribute evenly):
- Factual recall: specific value, date, name, threshold cited in source
- Procedural: "what are the steps to X?"
- Conditional: "under what condition does Y apply?"
- Prohibition: "what does the doc say ¬ to do?"

Spawn cold-start subagent. Provide:
- Translated file content only (¬ source, ¬ context)
- The N questions
- Instruction: answer each question from the document only; if absent → "NOT FOUND"

Score: correct / N. PASS ≥ 90%. WARN 75–89%. FAIL < 75%.
Correct = answer matches source ground truth (assess yourself after subagent responds).

## Test C — Comprehension

Select 1 representative task derivable from source content.
Task must require understanding ≥ 3 distinct facts from the source.

Spawn 2 subagents in parallel:
- Agent-Src: receives source doc + task
- Agent-Tr: receives translated doc + task

Compare outputs:
- Same conclusions reached? (yes/partial/no)
- Same key facts cited? (count matching facts / total)
- Different behavior? Signal any divergence.

Verdict: PASS = same conclusions + ≥ 80% facts matched | WARN = same conclusions, 60–79% facts | FAIL = different conclusions.

## Test D — Ambiguity

Select 2 rules or instructions from translated doc (prefer conditional/prohibition rules).
Spawn 3 subagents per rule (6 total), each cold-start.
Give each subagent: translated doc + scenario requiring applying the rule.

Assess behavioral consistency across 3 agents per rule:
- All 3 same behavior → PASS (100%)
- 2/3 same → WARN (67%)
- All different → FAIL (33%)

Overall ambiguity score = avg across 2 rules.
Benchmark: L0 prose avg 44% consistency (lower = more ambiguous). Lakon target ≥ 65%.

## Livrable

`[test][rapport]`

```
[test][rapport]
Source:   [source] ([src_tok] tok)
Lakon:    [translated] ([out_tok] tok)

A — Token gain:     [gain_pct]%  [✓ in range | ⚠ outside benchmark]
B — Info loss:      [score]/[N] ([PASS|WARN|FAIL])
C — Comprehension:  [verdict] — [matching_facts]/[total] facts | [same|diff] conclusions
D — Ambiguity:      [score]% consistency ([PASS|WARN|FAIL])
    R1: [rule excerpt] → [3/3 | 2/3 | 1/3]
    R2: [rule excerpt] → [3/3 | 2/3 | 1/3]

[any notable findings or divergences]
```
