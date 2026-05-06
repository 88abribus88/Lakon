---
name: test-prompt
description: |
  Behavioral equivalence test: classic prompt vs Lakon-translated prompt.
  Tests: token gain (A) + behavioral equivalence via cold-start subagents (E).
  Triggers: "test-prompt", "test prompt lakon", "behavioral equivalence test".
version: "1.0"
trigger: explicit
executor: main-agent
---

## Input
- `source` — abs path to classic (prose) prompt (.md | .txt)
- `translated` — abs path to Lakon-translated prompt (default: `[source-dir]/[basename]_lakon.md`)
- `scenarios` — list of test scenarios (optional — auto-generated if absent)
- `{root}` — repo root (contains `conventions.md`)

`source` required. `translated` absent → derive from `source` basename.

## Steps
1. Verify both files exist; exit with error if absent
2. Read both files; identify agent role + 3-5 key behavioral rules from source (§ Rule extraction)
3. Test A — Token gain (§ Test A)
4. Design scenarios if absent (§ Scenario design)
5. Test E — Behavioral equivalence (§ Test E)
6. Output report (§ Deliverable)

## Rule extraction
From source, identify:
- Agent role (identity)
- 3-5 key behavioral rules — prioritize: unconditional rules · tone constraints · structural rules (checkpoints, voice switching, output format)
- Any "formulation type" or quoted output scripts → flag as behavioral anchors to verify in Test E

## Test A — Token gain
```python
import tiktoken
enc = tiktoken.get_encoding('cl100k_base')
src = len(enc.encode(open('SOURCE_PATH').read()))
out = len(enc.encode(open('TRANSLATED_PATH').read()))
print(f"src={src} out={out} gain={round((1 - out/src)*100, 1)}%")
```
Record: `src_tok`, `out_tok`, `gain_pct`.
Benchmark: -20% to -45%. ~> already-lean prompts may fall below -20% — ¬ flag if behavioral equivalence passes.

## Scenario design
Design N_scenes scenarios:
- N_scenes = 2 (default) | 3 if agent has ≥3 distinct behavioral modes or voices
- Cover:
  - Typical use case — tests standard behavior + tone
  - Edge case — vague/ambiguous input that requires unconditional rule enforcement (HARD CONSTRAINTS equivalent)
  - ~> ≥3 modes: add 1 scenario per additional mode

For each scenario: one user message ≤3 sentences. ¬ telegraph expected behavior.

## Test E — Behavioral equivalence

For each scenario, spawn 4 subagents in parallel:
- Classic-1 · Classic-2: source prompt as system context + scenario
- Lakon-1 · Lakon-2: translated prompt as system context + scenario

Subagent instruction: "Read [PROMPT_PATH]. This is your system prompt — follow it exactly. Respond to: [scenario]. Produce only the agent response."

For each scenario, compare Classic vs Lakon across:
- Tone compliance (per agent's declared tone)
- Key rule compliance (check against rules extracted in § Rule extraction)
- Structural behavior (voice labeling, checkpoint phrases, output format)
- Unconditional rule enforcement (rules that must fire regardless of input signal)

Verdict per scenario:
- PASS: all key rules complied · tone consistent · same structural behavior
- WARN: minor divergence (1 non-critical rule missed · phrasing difference)
- FAIL: critical divergence (critical rule violated · wrong tone · opposite conclusion)

If WARN | FAIL → identify root cause:
- formulation-pattern: quoted output script removed → phrasing lost
- hard-constraint: unconditional rule not front-loaded → ¬ fires on weak signal
- compression-loss: content removed as "inferable" that was not
- other: [describe]

Overall verdict: PASS = all PASS | WARN = ≥1 WARN + 0 FAIL | FAIL = ≥1 FAIL

## Deliverable
`[test-prompt][report]`

```
[test-prompt][report]
Source:     [source] ([src_tok] tok)
Translated: [translated] ([out_tok] tok)

A — Token gain:  [gain_pct]%  [✓ in range | ⚠ outside benchmark]

E — Behavioral equivalence:  [PASS | WARN | FAIL]
  S1 [scenario summary]: [PASS | WARN | FAIL]
    Classic:  [key behaviors observed]
    Lakon:    [key behaviors observed]
    Delta:    [none | description]
  S2 [scenario summary]: [PASS | WARN | FAIL]
    ...

Root causes (if any): [formulation-pattern | hard-constraint | compression-loss | other]

Overall: [PASS | WARN | FAIL]
```
