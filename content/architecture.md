---
title: Architecture
---

MynaPress keeps responsibilities in separate `.myna` modules:

```text
src/main.myna      entrypoint
src/cli.myna       argv parsing and command dispatch
src/config.myna    site.toml + validation
src/content.myna   discovery, front matter, Page model
src/markdown.myna  block and inline Markdown rendering
src/template.myna  tokenizer, expressions, filters, layouts
src/site.myna      assembly, collections, tags, feeds, sitemap
src/server.myna    http.server + polling watch
src/util.myna      slugs, dates, escaping, pagination helpers
src/mynashine/     vendored MynaShine highlighting core
```

## Module imports

`src/main.myna` is deliberately thin:

```myna
@target(py)
<sys
<.cli:{run_cli}

@main {
  sys.exit(run_cli(sys.argv[1:]))
}
```

Modules use normal Myna relative imports rooted at the entry file's directory:

```myna
# src/content.myna
<.util:{DotDict, parse_date, reading_time, slugify}
```

`myna run` resolves each `.myna` file lazily and caches it in `sys.modules`. That gives real
module namespaces and ordinary Python tooling. It also lets the test harness import each
module directly instead of executing everything into one shared dictionary.

The first MynaPress build used a custom exec-based loader because Myna 0.3.0 could not
resolve sibling `.myna` modules. Myna 0.4.0 shipped the native loader, so that workaround
is gone. The original gap reports and their upstream fixes are recorded in `GAPS.md`.

## Data flow

1. `config.myna` validates `site.toml`.
2. `content.myna` discovers Markdown and parses front matter into `Page` dataclasses.
3. `markdown.myna` renders each body to HTML, highlighting fenced blocks through the
   vendored MynaShine core.
4. `site.myna` filters drafts, sorts posts, groups tags, paginates, renders templates, and
   writes into a staged directory. It swaps that directory into place only after success,
   so a failed build leaves the previous output intact.
5. `server.myna` serves the output and rebuilds on file changes.

## Tests

`tests/run_tests.myna` installs a scoped finder for `src/`, imports the application
modules, discovers `test_*.myna` files, and runs every `test_*` function. It prints
`PASS`/`FAIL` with the assertion detail and exits non-zero on failure. Coverage includes
slugs and traversal, front matter, Markdown blocks and inlines, syntax highlighting,
escaping, template filters, layouts and inheritance, errors, dates, pagination, feeds,
fixture builds, drafts, MIME types, 404s, CLI cleanup, output path safety, and failed-build
preservation.
