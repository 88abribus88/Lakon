# Lakon

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Status: beta](https://img.shields.io/badge/status-beta-blue.svg)]()
[![Claude Code](https://img.shields.io/badge/runs%20on-Claude%20Code-blueviolet.svg)](https://claude.ai/code)

**Token-efficient writing conventions for LLM-readable documents.**

Lakon is a translation language — a set of conventions for rewriting documents so that LLMs can read them with fewer tokens, without losing any meaning.

---

## The problem

LLMs process text as tokens. Every token in a prompt, knowledge base, or rule file has a cost: latency, money, and context window pressure. When you feed an LLM a 5,000-token document, a significant portion of those tokens carry no semantic value for the model — they exist for human readers.

Standard solutions have tradeoffs:
- **RAG / chunking** — splits documents, loses cross-chunk context
- **Summarization** — lossy by definition, collapses nuance
- **Prompt compression** (LLMLingua, etc.) — automated, opaque, unreliable on technical content

Lakon takes a different approach: **authoring-time translation**.

---

## When it's worth it

Translating a document to Lakon costs tokens upfront — the translation call itself has a price. Whether that investment pays off depends entirely on how often the translated document is used.

**High-value targets** are documents read on every request or loaded in every session: system prompts, agent instructions, knowledge bases, rule files. The translation cost is paid once; the saving compounds with every use. A system prompt translated at -30% and called 1,000 times returns that saving 1,000 times over.

**Low-value targets** are documents read once and discarded: one-off reports, single-use context fed into a single conversation. For these, the translation cost may exceed the saving. Lakon is not worth applying to content that won't be reused.

**The context window argument**

Token reduction is not only about cost — it is about conversation longevity. Every token saved in a persistent document is a token freed for conversation history. Lakon-translated knowledge bases and system prompts leave more room for the exchange to breathe: longer sessions before the context window fills, less premature truncation, less pressure to summarize and lose history. The goal is not just cheaper calls — it is conversations that last longer with less noise.

## The core idea: LLM-native readability

The starting point of Lakon is not token reduction — it is readability for LLMs.

LLMs do not read the way humans do. They generate from context, leveraging patterns absorbed during training. Verbose prose, decorative formatting, and connective tissue that human readers need to track meaning — LLMs infer all of this from context. Stripping it does not degrade comprehension; it reduces noise.

But this only works if what replaces it is already within the model's native competency. Lakon does not invent a new notation. Every symbol and abbreviation it uses is one LLMs already understand from training data: `→` as causal connector, `¬` as logical negation, `↑`/`↓` for increase/decrease. The translated document is immediately interpretable by any LLM — no prior exposure to Lakon required.

This is the constraint that separates Lakon from arbitrary compression. A scheme that invents new abbreviations or drops content might reduce tokens while degrading comprehension. Lakon maps to existing LLM competencies: what it removes, models infer; what it substitutes, models already know.

**Why English**

Lakon outputs English regardless of the source language. This is not an arbitrary choice — it follows directly from the LLM-native readability principle.

LLM training data is heavily English-dominant. This means context inference is most reliable in English: models fill gaps, resolve ellipsis, and interpret dense notation with the highest confidence there. The same Lakon-compressed sentence in French may require more tokens for an LLM to process safely, because the contextual shortcuts Lakon relies on are less densely reinforced.

English is also linguistically more compact than most European languages: shorter average word length, less inflection, fewer obligatory articles and particles. A French sentence translated to English before Lakon compression typically yields a larger token reduction than the same sentence compressed in French.

The consequence: **Lakon is more effective in English**, and translating to English is part of the pass for any source language. This is not a limitation — it is the mechanism, a direct reflection of where LLM training data is currently concentrated.

This may evolve. As models are trained on larger and more balanced multilingual corpora, the gap between languages will narrow. A future model trained equally on French and English may infer context from French prose as reliably as from English. At that point, the case for English as the mandatory output language weakens — and Lakon should follow the data.

## Translation, not compression

The distinction matters. A compressor reduces size. A translator preserves intent and behavioral impact across a change of form.

Lakon treats LLM-readable documents the way a human translator treats text: the unit of translation is the group (not the word), meaning is transferred as a package including presuppositions and implicatures, and anything that cannot be translated without loss — paths, commands, URLs, identifiers — is left verbatim.

This is grounded in translation theory (Nida's formal/dynamic equivalence), controlled natural language research (ASD-STE100), and precision communication principles (BLUF/ICAO). The key insight is that **ambiguity lives at junctions between groups, not within them** — so symbolic substitution at junctions reduces ambiguity as well as token count.

---

## How it works

Lakon applies five categories of transformation:

**1. Function word elision**
Words that carry no meaning when context already holds it are dropped.
> "Before starting, you must ensure that the following are in place:" → "Ensure:"

**2. Symbolic substitution**
Common logical and causal connectors are replaced with single-token symbols.

| Symbol | Replaces | Token gain |
|---|---|---|
| `→` | "leads to", "if … then", "results in" | +1–2 |
| `¬` | "not", "do not" | 0 (logical clarity) |
| `↑` / `↓` | "increases" / "decreases" | +1–2 |
| `⇔` | "if and only if" | +1 |
| `·` | inline separator | neutral |

**3. Layout cleanup**
Markdown formatting that adds tokens without LLM-readable value is removed:
- `**bold**` on prose and labels (≈ +2 tok each, tested)
- `*italic*` on prose
- `---` horizontal rules
- Emoji
- HTML comments

Code blocks, headers (`##`), and hierarchical list markers are preserved — they carry structural meaning.

**4. Verbatim floor**
Paths, commands, URLs, config values, identifiers (UUID, hash, IP) are never translated. They are analogous to proper nouns: touching them risks silent semantic breakage.

**5. Synonymic substitution**
Longer words replaced by shorter exact equivalents where the substitution is universally inferrable.
> "mentioned" → "cited" | "regarding" → "on" | "utilize" → "use"

---

## Results

Tested across 5 document types with cold-start verification (fresh agent, no context, answer questions from translated doc only):

| Document type | Token gain | Cold-start |
|---|---|---|
| Prose (knowledge base, Wikipedia) | -38 to -41% | PASS |
| Structured rules / skill files (French source) | -20 to -44% | 9/9 PASS |
| Technical spec (RFC-style) | -24% | PASS |
| Markdown with code blocks | -42% | PASS |

Cold-start threshold: PASS = 90%+ of questions answered correctly from translated doc alone.

The benchmark sweet spot is **-20% on rules, -39% on prose** — consistent across model sizes (Haiku → Opus).

The structured rules test used French-language source documents. Lakon translated content to English as part of the pass — the cold-start agent answered questions in English from the translated output only. This validates the cross-language use case: French (or other language) knowledge bases and rule files can be translated to English Lakon in a single pass, with no loss of information.

---

## Ambiguity

A common concern: does denser notation introduce ambiguity?

We tested this with a 16-agent protocol: agents were given the same rules at different notation levels (L0 prose → L3 structured) and asked to apply them to identical scenarios. Behavioral consistency was measured across agents.

Key finding: **notation change alone does not resolve ambiguity rooted in missing information** — it only helps when the ambiguity is structural (actor unclear, scope unclear, condition order wrong). Lakon's expression patterns (explicit actor, scope-left, exception marker `~>`, validation gate `GATE[...]`) target exactly those structural sources.

Target: ≥ 65% behavioral consistency across agents (vs. ~44% for plain prose).

---

## Getting started

There are two ways to use Lakon depending on your setup.

### With Claude Code

Claude Code automates the full workflow: extraction, translation, token counting, and cold-start audit.

**1. Install dependencies**

```bash
pip install tiktoken python-docx pdfminer.six
```

See `SETUP.md` for full instructions and optional dependencies.

**2. Open in Claude Code**

```bash
cd lakon
claude
```

**3. Translate a document**

Tell Claude: `translate path/to/your-document.pdf`

Or: `lakon this` with a file path.

Supported formats: `.md`, `.txt`, `.docx`, `.pdf`

Output: `[filename]_lakon.md` in the same directory, with a token gain report and automatic audit.

**4. Run the test protocol**

```
test lakon source=original.md translated=original_lakon.md
```

Runs 4 tests: token gain measurement, cold-start info loss verification, comprehension comparison, and ambiguity consistency test.

### Without Claude Code

If you are using Claude.ai Projects, the API, or any Claude interface without Claude Code, use the system prompt in `prompts/translate-system.md`.

Copy the prompt block into your project system prompt (or API `system` field), then send documents directly. Claude will translate them to Lakon following the full convention set.

The prompt is itself written in Lakon — token-efficient by design. It covers all edge cases: verbatim floor, symbol safety (¬ and ⇔ misuse risks), non-inferable content filtering, layout cleanup, and language normalization.

**What to translate**

The two highest-value targets are:

**Prompts and agent instructions.** System prompts, rule files, and behavioral instructions are read by the model on every request. Every token saved there is saved on every call. Lakon is particularly effective here because instructions are dense with connectives, conditionals, and labels — exactly the structure it targets. A system prompt translated to Lakon typically runs 20–44% shorter with no behavioral change.

**Context documents.** Knowledge bases, product specs, policy documents, and reference material fed into a context window accumulate fast. A 5,000-token knowledge base at -40% saves 2,000 tokens of context pressure on every conversation that loads it — freeing space for conversation history, examples, or additional documents. Unlike summarization, Lakon is lossless: the translated document contains everything the original did.

---

## Repository structure

```
lakon/
  conventions.md          — Lakon rules (the single source of truth)
  prompts/
    translate-system.md   — system prompt for use without Claude Code
  skills/
    translate/SKILL.md    — translate any doc to Lakon + audit
    test/SKILL.md         — 4-protocol validation suite
  examples/               — translated samples (README, RFC, Wikipedia, CRM)
  SETUP.md                — install instructions
```

---

## Examples

The `examples/` directory contains real translated documents:

| Source | Type | Source language | Gain |
|---|---|---|---|
| `requests-readme.md` | Python library README (markdown) | English | -42% |
| `rfc9110-excerpt.txt` | HTTP semantics RFC (formal spec) | English | -24% |
| `wikipedia-attention.txt` | Wikipedia article (technical prose) | English | -24% |
| `crm-wikipedia.txt` | CRM knowledge base article (long prose) | English | -41% |
| `test-input.md` | Configuration guide + rule set | French | -40% |

Each `_lakon.md` file can be compared directly against its source.

---

## Name

*Lakon* (Λάκων) — a Laconian, an inhabitant of Lacedaemon (Sparta).

The Laconians were famous throughout the ancient world for their terseness. Laconic speech: say exactly what is needed, nothing more. The word "laconic" derives directly from them.

The most cited example: Philip II of Macedon sent a message to Sparta — *"If I enter Laconia, I will level Sparta to the ground."* The Spartan reply: *"If."*

That is the spirit of this system. Every token that carries no meaning is a token wasted.

---

## License

MIT — see `LICENSE`. The name "Lakon" is subject to additional terms — see `NOTICE`.
