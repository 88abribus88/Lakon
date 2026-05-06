# Setup

## Requirements

- Claude Code (CLI) — [install](https://claude.ai/code)
- Python 3.8+
- Linux | macOS | Windows

## Install

### Linux / macOS

```bash
pip install tiktoken python-docx pdfminer.six
```

### Windows

```powershell
pip install tiktoken python-docx pdfminer.six
```

If `pip` is not found, use:
```powershell
python -m pip install tiktoken python-docx pdfminer.six
```

If `python3` is not recognized on Windows, use `python` instead — both refer to the same installation.

## Optional — richer PDF extraction (tables, columns)

```bash
pip install pymupdf
```

## Verify

### Linux / macOS
```bash
python3 -c "import tiktoken, docx; from pdfminer.high_level import extract_text; print('OK')"
```

### Windows
```powershell
python -c "import tiktoken, docx; from pdfminer.high_level import extract_text; print('OK')"
```

## Skills

Invoke via Claude Code trigger phrases:

| Skill | Trigger | Input |
|---|---|---|
| translate | "translate", "lakon this", file path | `path=` source doc |
| test | "test lakon", "run lakon test" | `source=` + `translated=` paths |

## Notes

- Supported formats: `.md`, `.txt`, `.docx`, `.pdf`
- Output file written next to source: `[filename]_lakon.md`
- All skills are cross-platform — no OS-specific behavior
