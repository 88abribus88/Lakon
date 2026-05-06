---
type: meta
version: 1.3
role: shared core
---

## language
All files using Lakon: English.

## Lakon
Translation language: any source doc → LLM-readable form (prompts, rules, memory docs, ingested docs).
Intent + behavioral impact preserved across form change. ¬ size reduction alone.
Subagent prompt → first step: Read `conventions.md` from repo root.

## translation rules
- Elide function words when context holds meaning
- Symbolic substitution: `→` `|` `+` `-` ; `=` if both sides ≤3 words
- Synonymic substitution to shorter equivalent ("mentioned" → "cited")
- Stable abbrevs: see list below
- ¬ root truncation
- ¬ invented abbrevs outside stable list

## untranslatable (verbatim floor)
¬ translate: paths, cmds, IPs, config values, YAML frontmatter, code blocks + backtick spans,
URLs + identifiers (UUID, SIREN, hash).
Trigger phrases (user-typed activation signals in skill `description` field): verbatim floor — ¬ translate.

## private
`<private>...</private>` → strip content; replace inline with semantic descriptor
  ex: `<private>sk-abc123</private>` → "credential configured"
¬ verbatim floor — content destroyed.

## layout
Keep: `## headers`, code blocks, list markers on hierarchical/comparative structures.
Remove: `---` horizontal rules (≠ YAML delimiters), `*italic*` on prose,
        HTML maintenance comments, `**bold**` always (¬ semantic value for LLM; +2 tok overhead — tested),
        emoji always (¬ semantic value; variable tok cost; use numeric/symbolic equivalent).
Multi-line cmds → fenced code block (copy-paste safety).
¬ inline-merge list items.

## stable symbols (extends + - = → | ·)
Quick ref: ¬ = not · → = if-then/leads-to · | = or · · = separator · = = biconditional (≤3 words each side)
↑  increases / crosses threshold upward
↓  decreases / crosses threshold downward
⇔  if and only if (strict biconditional — sufficient condition only → use →)
¬  not / do not — dense conditions, ¬ connector
∴  therefore / as a consequence — logical chains
∧  and (logical) — clarity only, no tok gain
·  inline separator (entry label · description)

## expression patterns
actor ambiguity → SUBJECT: action — explicit when non-default; default = agent
scope ambiguity → SCOPE: action — condition/context always left of what it governs
exception → ~> case: action — exception to adjacent rule; ¬ sequence/process rules
priority → // priority action — absolute priority; often combined with ~>
validation gate → GATE[SUBJECT:signal] → scope — explicit conditional barrier; ¬ implicit "requires"

## imperative weight (skill prompts)
¬ use on ordinary rules.

Scale (↑ constraint):
show-through → ¬ rule; surface via examples | context
soft         → `prefer [x]` — conditional guidance
standard     → direct statement `[rule]`
strong       → `always [rule]` — absolute obligation; extends temporal scope
prohibition  → `¬ [rule]`
absolute     → `¬ ever [rule]` — extends `¬` + temporal scope

`## HARD CONSTRAINTS` — groups strong + absolute rules; priority anchor
`## behavior` — signals behavioral rules section

## stable abbrevs
- tok
- occ
- sh (short-term)
- lg (long-term)
- sys
- cfg
- cmd
- ref
- vrs
- sid
- ts

## shared core
Only non-inferable content. Inferable = derivable from source doc, codebase, or model knowledge.
Write: rules + usage constraints. ¬ rationale, expected gains, validation data.
Check: if removing it ¬ changes agent output → remove.
