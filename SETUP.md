# Setup

## Requirements

- Claude Code (CLI)
- Linux | macOS
- Python 3.8+

## Install

```bash
pip install tiktoken python-docx pdfminer.six
```

For `.docx` → text extraction: `python-docx`
For `.pdf` → text extraction: `pdfminer.six`
For `.md` / `.txt`: no extra dependency

Optional — richer PDF extraction (tables, columns):
```bash
pip install pymupdf
```

## Verify

```bash
python3 -c "import tiktoken, docx; from pdfminer.high_level import extract_text; print('OK')"
```

## Skills

Invoke via Claude Code slash commands or trigger phrases:

| Skill | Trigger | Input |
|---|---|---|
| translate | "translate", "lakon this", file path | `path=` source doc |
| test | "test lakon", "run lakon test" | `source=` + `translated=` paths |
