---
title: Getting started
---

## Requirements

- Python 3.11 or newer.
- Myna 0.3.0. Install the language with `pipx install myna-lang` (or
  `pip install myna-lang` in a virtual environment).
- MynaPress itself. Clone this repository and run it from its root. No package install is
  needed for MynaPress: the source lives in `src/` and `bin/mynapress` is a convenience
  wrapper.

If Myna is only available as a source checkout, put it on `PYTHONPATH` for every command:

```bash
export PYTHONPATH=/home/joseph/Projects/myna
```

This repository uses that convention in development and in the commands below when Myna
is not installed globally.

## A minimal site

```text
site.toml
content/
  index.md
  about.md
templates/
  page.html
assets/
  style.css
```

`site.toml`:

```toml
title = "My Site"
description = "Notes and things."
author = "Ada"
language = "en"
paginate = 10
```

`content/index.md`:

```markdown
---
title: Home
---

Welcome to **my site**.
```

`content/about.md`:

```markdown
---
title: About
---

This is the about page.
```

`templates/page.html`:

```html
<!doctype html>
<html lang="{{ site.language }}">
<head><meta charset="utf-8"><title>{{ page.title }}</title></head>
<body>
  <h1>{{ page.title }}</h1>
  {{ content | safe }}
</body>
</html>
```

Build it:

```bash
myna run src/main.myna build
```

The output is written to `build/` by default. Open `build/index.html`, or run the local
server:

```bash
myna run src/main.myna serve --watch
```

`serve` builds first, serves `build/` on `http://localhost:8000/`, returns a real 404 for
unknown paths, and rebuilds when a content, template, static, favicon, or config file
changes.

## Creating a post

```bash
myna run src/main.myna new "A first post"
```

This writes `content/YYYY-MM-DD-a-first-post.md` with front matter. Add a `date` and it
becomes a post automatically; posts are collected newest-first on the generated homepage
and participate in tag pages, pagination, and RSS.

## Project layout

| Path | Purpose |
| :--- | :--- |
| `src/main.myna` | Entrypoint and small Myna module loader. |
| `src/util.myna` | Slugs, dates, escaping, hashing, pagination helpers. |
| `src/config.myna` | `site.toml` loading and validation. |
| `src/content.myna` | Discovery, front matter, `Page` model, URLs. |
| `src/markdown.myna` | Markdown block and inline renderer. |
| `src/template.myna` | Template tokenizer, expressions, filters, layouts. |
| `src/site.myna` | Assembly, collections, tags, pagination, RSS, sitemap. |
| `src/server.myna` | `http.server` wrapper and polling watch mode. |
| `src/cli.myna` | Argument parsing and user-facing command output. |
| `tests/` | Myna test harness and unit/end-to-end tests. |
| `GAPS.md` | Language gaps found while building this project. |
