# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A knowledge base of development best practices and tips (backend/Django, frontend, GitHub, security),
published as an MkDocs site with the Material theme to https://humen-dev.github.io/dev-best-practices/.
There is no application code: the content is Markdown in `docs/`.

Notes are written in **English**, even when the request comes in Ukrainian.

## Commands

```bash
uv run mkdocs serve            # live preview at http://127.0.0.1:8000
uv run mkdocs build --strict   # what CI runs; fails on broken links and anchors
```

Set `NO_MKDOCS_2_WARNING=1` to silence the MkDocs 2.0 / Zensical banner.

Python is pinned to 3.13 in `.python-version`: on 3.15 `pyyaml` has no wheel and the install fails.
`mkdocs<2` is pinned on purpose, because MkDocs 2.0 drops support for Material.
Zensical (Material's successor, reads the same `mkdocs.yml`) is the expected migration path.

## How the site is put together

- **No `nav:` in `mkdocs.yml`.** The menu is built from the `docs/` folder tree, and
  `navigation.indexes` makes each folder's `index.md` its landing page.
  A new folder therefore needs an `index.md`, and the parent `index.md` should link to the new page or folder.
- `validation.anchors: warn` plus `--strict` makes the build fail on links to missing headings,
  so cross-page links like `commit-email.md#name` must match a real heading slug.
- Enabled Markdown extensions: admonitions (`!!! tip`), content tabs (`=== "Tab"`),
  collapsible blocks, task lists (`- [ ]`), code highlighting.

## Writing a note

Follow the template in `docs/contributing.md`: TL;DR → Problem → Solution → Why it works →
Trade-offs → Pitfalls. File names are lowercase and hyphen-separated.

- **Trade-offs are the point of this repo.** Where there is more than one way to do something,
  compare the options in an `Option | Pros | Cons` table and end with a `!!! tip "Recommendation"`.
  Don't write bare step-by-step instructions.
- Check setting names, version numbers and behaviour against official docs before writing them.
  Quote UI setting names exactly as GitHub/Django/etc. show them, in bold.
- `docs/github/privacy/` is the reference example of a finished section.

## Deploy and Git workflow

`.github/workflows/deploy.yml` builds with `--strict` on every PR, and on push to `main`
runs `mkdocs gh-deploy` to the `gh-pages` branch, which GitHub Pages serves.
Merging to `main` publishes, so changes go through a PR from a branch (`docs/<topic>`).
