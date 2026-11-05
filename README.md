# Dev Best Practices

Team notes on development best practices and tips, published as an MkDocs site.

```bash
uv run mkdocs serve   # preview at http://127.0.0.1:8000
uv run mkdocs build --strict   # build into site/, fail on broken links
```

Notes live in `docs/`. See [docs/contributing.md](docs/contributing.md) for the layout and a note template.

Every push to `main` publishes the site to GitHub Pages (`.github/workflows/deploy.yml`).
