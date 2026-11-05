# Contributing

## Where to put a note

One topic per file, inside the section it belongs to:

```
docs/
├── backend/
│   └── django/
│       └── orm-n-plus-one.md
├── frontend/
├── github/
└── security/
```

- File names are lowercase and hyphen-separated: `orm-n-plus-one.md`.
- A new folder needs an `index.md` that says what the folder covers.
- The site menu is built from the folder tree, so you do not edit `mkdocs.yml`.

## Note template

````markdown
# <Title: what to do, e.g. "Use select_related for foreign keys">

**TL;DR:** one or two sentences with the rule.

## Problem

What goes wrong without it. Show the bad code or setting.

## Solution

```python
# the good version
```

## Why it works

Short explanation. Link to official docs.

## Pitfalls

!!! warning
    Cases where the rule does not apply.
````

## Preview locally

```bash
uv run mkdocs serve
```

Open <http://127.0.0.1:8000>. The page reloads when you save a file.
