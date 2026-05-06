# Lakon

Lakon = writing convention for LLM-readable documents.
Goal: reduce token cost while preserving intent + behavioral impact.
Benchmarks: -20% avg on structured rules | -39% on prose | cold-start 9/9 PASS.

## What's here

- `conventions.md` — Lakon rules (translation rules, symbols, layout)
- `skills/translate/` — translate any doc (docx/pdf/md/txt) to Lakon
- `skills/test/` — measure token gain, verify info loss, test comprehension + ambiguity

## Usage

1. Run `SETUP.md` install steps once.
2. Open this repo in Claude Code.
3. Translate a doc: tell Claude "translate path/to/file.pdf" (or "lakon this").
4. Test the output: "test lakon source=file.pdf translated=file_lakon.md".

## Conventions

All Lakon-translated files: English.
Source docs: any language — translate content to English as part of Lakon pass.
The `conventions.md` is the single source of truth for all translation decisions.
