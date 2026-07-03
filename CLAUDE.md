# CLAUDE.md — ai-governance-ceiling-simulator
# Single-file browser HTML app — no npm, no build step, no test runner

---

## Project Context

**Type:** Single-page browser app (pure HTML + embedded CSS/JS, no dependencies)
**Entry point:** `index.html`
**External resources:** Chart.js CDN only

---

## Linting

Standard lint run (4 gates — full check):

```bash
python C:/Dev2/Tools/lint.py -p "*.html" -I
```

This app uses browser globals (`console`, `document`, `alert`, `fetch`, etc.) — always use `-I` (include defaults).
Skip JS gates for pure HTML structure checking:

```bash
python C:/Dev2/Tools/lint.py -p "*.html" --html-only
```

### lint.py参数说明 (for other repos)

| Flag | Purpose |
|---|---|
| `-p "*.html"` | Files to lint (glob or path) |
| `-I` | Add browser globals to generated ESLint config |
| `--html-only` | Skip ESLint + JS parse gates |
| `-F` | Fail on first file error |
| `-v` | Print tool output on failure |

---

## Verification Gates

Any file modification requires these three gates:

1. **Python parse** — `python -m py_compile <path> && echo OK`
2. **lint.py** — all 4 gates pass
3. **Git diff** — only intended files changed

---

## Commit Policy

- Never overwrite existing files — increment filenames or add version suffixes
- Keep a CHANGELOG entry for each functional change
- Run lint before committing
- Do not push — Albert pushes manually