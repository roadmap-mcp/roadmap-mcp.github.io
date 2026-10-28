# roadmap-doc

The documentation website for Roadmap, built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and published to
GitHub Pages by `.github/workflows/pages.yml` on every push to `main`.

## Write and preview

```bash
python -m venv .venv
.venv/Scripts/pip install -r requirements.txt     # .venv/bin/pip on Linux/macOS
.venv/Scripts/mkdocs serve                         # http://127.0.0.1:8000
.venv/Scripts/mkdocs build --strict                # what CI runs
```

Pages live in `docs/`; the navigation is the `nav:` list in `mkdocs.yml`.
Diagrams are Mermaid code blocks (```` ```mermaid ````).

## Publishing

1. Push this repository to GitHub.
2. Settings → Pages → *Build and deployment* → Source: **GitHub Actions**.
3. Set `site_url` in `mkdocs.yml` to the Pages address.

## Keeping it in step

The source of truth for internals is the Roadmap repository's `docs/` folder
(`docs/knowledge-base.md` for the Knowledge Base). This site is the user-facing
version of it: when a feature there changes, update the matching page here.
