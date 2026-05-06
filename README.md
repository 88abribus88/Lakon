# Lakon

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

## The core idea: translation, not compression

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
| Structured rules / skill files | -20 to -44% | 9/9 PASS |
| Technical spec (RFC-style) | -24% | PASS |
| Markdown with code blocks | -42% | PASS |

Cold-start threshold: PASS = 90%+ of questions answered correctly from translated doc alone.

The benchmark sweet spot is **-20% on rules, -39% on prose** — consistent across model sizes (Haiku → Opus).

---

## Ambiguity

A common concern: does denser notation introduce ambiguity?

We tested this with a 16-agent protocol: agents were given the same rules at different notation levels (L0 prose → L3 structured) and asked to apply them to identical scenarios. Behavioral consistency was measured across agents.

Key finding: **notation change alone does not resolve ambiguity rooted in missing information** — it only helps when the ambiguity is structural (actor unclear, scope unclear, condition order wrong). Lakon's expression patterns (explicit actor, scope-left, exception marker `~>`, validation gate `GATE[...]`) target exactly those structural sources.

Target: ≥ 65% behavioral consistency across agents (vs. ~44% for plain prose).

---

## Getting started

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

---

## Repository structure

```
lakon/
  conventions.md          — Lakon rules (the single source of truth)
  skills/
    translate/SKILL.md    — translate any doc to Lakon + audit
    test/SKILL.md         — 4-protocol validation suite
  examples/               — translated samples (README, RFC, Wikipedia, CRM)
  SETUP.md                — install instructions
```

---

## Examples

The `examples/` directory contains real translated documents:

| Source | Type | Gain |
|---|---|---|
| `requests-readme.md` | Python library README (markdown) | -42% |
| `rfc9110-excerpt.txt` | HTTP semantics RFC (formal spec) | -24% |
| `wikipedia-attention.txt` | Wikipedia article (technical prose) | -24% |
| `crm-wikipedia.txt` | CRM knowledge base article (long prose) | -41% |

Each `_lakon.md` file can be compared directly against its source.

---

## License

MIT — see `LICENSE`.
