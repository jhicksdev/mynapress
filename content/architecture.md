---
title: Architecture
---

MynaPress keeps responsibilities in separate `.myna` modules:

```text
src/main.myna      entrypoint + module loader
src/cli.myna       argv parsing and command dispatch
src/config.myna    site.toml + validation
src/content.myna   discovery, front matter, Page model
src/markdown.myna  block and inline Markdown rendering
src/template.myna  tokenizer, expressions, filters, layouts
src/site.myna      assembly, collections, tags, feeds, sitemap
src/server.myna    http.server + polling watch
src/util.myna      slugs, dates, escaping, pagination helpers
```

## Why a custom loader?

Myna 0.3.0 compiles relative imports to Python relative imports. `myna run` executes the
compiled code with `__main__` semantics and no package, and Python cannot import `.myna`
files. There is no source-relative module graph in the language.

Rather than put the whole application in one enormous file, `src/main.myna` compiles each
module with the public Myna compiler and executes it into one shared runtime namespace:

```myna
src = path.read_text(encoding="utf-8")
code = myna_to_python(src)
exec(compile(code, @str(path), "exec"), ns)
```

The tests use the same technique. This is a workaround for a real language gap, documented
in `GAPS.md`. It also means modules share one global namespace rather than proper module
objects.

## Data flow

1. `config.myna` validates `site.toml`.
2. `content.myna` discovers Markdown and parses front matter into `Page` dataclasses.
3. `markdown.myna` renders each body to HTML.
4. `site.myna` filters drafts, sorts posts, groups tags, paginates, renders templates, and
   writes output plus feeds.
5. `server.myna` serves the output and rebuilds on file changes.

## Tests

`tests/run_tests.myna` loads the application modules, discovers `test_*.myna` files, and
runs every `test_*` function. It prints `PASS`/`FAIL` with the assertion detail and exits
non-zero on failure. Coverage includes slugs and traversal, front matter, Markdown blocks
and inlines, escaping, template filters and layouts, errors, dates, pagination, feeds,
fixture builds, drafts, MIME types, 404s, and CLI cleanup.
