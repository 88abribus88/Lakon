# Contributing

Lakon is in active testing. The best contribution right now is running the protocol and reporting findings.

There are two testing paths depending on your setup. Both produce useful data — they are complementary, not hierarchical.

## With Claude Code (quantitative)

Claude Code runs the full protocol automatically: extraction, translation, token counting, and cold-start audit.

1. Follow `SETUP.md` to install dependencies.
2. Open the repo in Claude Code.
3. Pick a document from your own work (knowledge base, spec, rule file, guide).
4. Run: `translate path/to/your-doc.md`
5. Run: `test lakon source=your-doc.md translated=your-doc_lakon.md`
6. Report your results (see below).

This produces exact token counts (tiktoken cl100k_base), a structured cold-start score, and a reproducible audit trail.

## Without Claude Code (qualitative)

If you are using Claude.ai Projects or the API without Claude Code, copy the system prompt from `prompts/translate-system.md` into your project and translate documents directly.

You can contribute:
- Convention edge cases — situations where the rules gave unclear or unexpected output
- Translation quality observations — a sentence or block that translated poorly or produced ambiguity
- Comprehension gaps — a case where you could tell the translated version lost something the original had

You do not need to run the token protocol. Qualitative findings on real documents in real use contexts are among the most valuable inputs at this stage.

## What to report

Open an issue with any of the following:

- **Token gain** — source tok, output tok, doc type
- **Cold-start failure** — a question the translated doc couldn't answer that the source could
- **Comprehension gap** — a task where the agent behaved differently with the translated doc
- **Ambiguity** — a rule or instruction that translated agents interpreted differently
- **Convention edge case** — a situation where the rules in `conventions.md` gave unclear guidance

Use the issue templates in `.github/ISSUE_TEMPLATE/` — [Finding / Feedback](.github/ISSUE_TEMPLATE/finding.md) or [Convention change proposal](.github/ISSUE_TEMPLATE/convention-change.md).

## Proposing changes to conventions

Changes to `conventions.md` require evidence:

1. Identify the rule or gap.
2. Show a concrete example: source text → current translation → proposed translation.
3. Provide a behavioral test: what agent output changes with the proposed rule?
4. Open an issue or PR with the above.

¬ propose changes based on aesthetic preference alone — every rule must have a behavioral rationale.

## Pull requests

PRs welcome for:
- New example documents (source + translated pair in `examples/`)
- Bug fixes in the translate or test skill
- Convention changes with evidence (see above)

¬ PRs that add prose explanations already covered by `conventions.md`.

## Questions

Open an issue with the `question` label.
