# <img src="favicon.svg" alt="MynaPress logo" width="32" height="32"> MynaPress

[![CI](https://github.com/jhicksdev/mynapress/actions/workflows/ci.yml/badge.svg)](https://github.com/jhicksdev/mynapress/actions/workflows/ci.yml)

A small, dependency-free static site generator written **entirely in Myna**.

MynaPress takes Markdown with front matter and produces a fast, self-contained HTML site
with collections, tags, pagination, RSS, a sitemap, a dev server, and watch mode. It has
zero runtime dependencies beyond [`myna-lang`](https://pypi.org/project/myna-lang/) and the
Python standard library.

It is also a dogfooding experiment: the project exists to test whether Myna is ready for
real work. The friction found along the way is documented in [`GAPS.md`](GAPS.md), not
hidden.

## Status

v0.1 targets Python 3.11+ only. Application files declare `@target(py)`; the vendored
MynaShine core also declares `@target(py, js)` so it can be shared with its upstream
project. There is no pretend JavaScript/Node support for MynaPress itself. Builds are full
rebuilds, but each build is staged in a temporary directory and swapped into place only on
success; see [Known limitations](#known-limitations).

The documentation site in `content/` is built by MynaPress itself. Generated HTML is not
committed; CI builds it. It is published to
[GitHub Pages](https://jhicksdev.github.io/mynapress/) from the `pages.yml` workflow.

## Requirements and installation

- Python 3.11 or newer.
- Myna 0.5.0+ with multi-file `.myna` module resolution (`myna run` must support sibling
  imports such as `<.util:{slugify}`). The vendored highlighter is developed and tested
  against 0.5.0.

```bash
uv tool install myna-lang==0.5.0
# or: pipx install myna-lang==0.5.0
```

Clone this repository and run from its root:

```bash
git clone https://github.com/jhicksdev/mynapress.git
cd mynapress
myna run src/main.myna build
```

MynaPress does not need to be installed as a package. If you are working from a local Myna
checkout, install it editable once:

```bash
uv venv .venv
uv pip install --python .venv/bin/python -e /path/to/myna
source .venv/bin/activate
myna run src/main.myna build
```

The `bin/mynapress` wrapper is a developer convenience. It prefers `myna` on `PATH`, then
the repository `.venv`, then an installed `python3 -m myna`. For a local Myna checkout,
set `MYNA_SRC=/path/to/myna`; if no Myna installation is found, the wrapper exits with a
clear error.

## Quick start

```bash
# build the bundled documentation site into ./build
myna run src/main.myna build

# build the fixture site with an explicit config
myna run src/main.myna build --config fixtures/site/site.toml

# serve and rebuild on changes
myna run src/main.myna serve --watch

# create content
myna run src/main.myna new "My first post"
```

A minimal site needs `site.toml`, `content/`, and `templates/`:

```text
site.toml
content/
  index.md
templates/
  page.html
assets/
  style.css
```

```toml
# site.toml
title = "My Site"
description = "Notes and things."
base_url = "https://example.com"   # optional; enables RSS/sitemap/robots
base_path = "/docs"                # optional; for subpath hosting
paginate = 10
```

```html
<!-- templates/page.html -->
<!doctype html>
<html lang="{{ site.language }}">
<head><meta charset="utf-8"><title>{{ page.title }}</title></head>
<body>
  <h1>{{ page.title }}</h1>
  {{ content | safe }}
</body>
</html>
```

## CLI

```bash
myna run src/main.myna build [--config site.toml] [--src content] [--out build] [--drafts]
myna run src/main.myna serve [--port 8000] [--watch] [--config site.toml] [--src content] [--out build] [--drafts]
myna run src/main.myna new "Post title" [--config site.toml]
myna run src/main.myna clean [--config site.toml]
```

- `build` prints a concise summary: pages, posts, tag pages, assets, warnings, duration.
  It builds into a temporary sibling directory and swaps it into place only after a
  successful build, so a failed build preserves the previous output.
- `serve` builds first, serves `--out` with `http.server`, uses stdlib MIME types, returns
  404 for unknown paths, and `--watch` polls for changes and rebuilds.
- `new` writes `content/YYYY-MM-DD-<slug>.md` with front matter and avoids overwriting.
- `clean` removes a MynaPress output directory inside the site root and
  `.mynapress-cache.json`. It refuses output outside the site root and directories that
  were not created by MynaPress.
- Content and template errors exit non-zero with `error: file:line: message` where a line
  is available; config errors identify the config file. Raw content errors are not dumped
  as tracebacks.
- An explicitly supplied `--config` path must exist. If `site.toml` is absent and no
  `--config` was supplied, defaults are used.
- Unknown commands exit 2.

## Content model

Markdown files are discovered recursively in `content/`. Only `.md` is read. Nested folders
are allowed for organisation; URLs are currently flat (`/<slug>/`, or `/` for the root
`index.md`).

### Front matter: supported YAML subset

Front matter starts with a first line of exactly `---` and ends at `---` or `...`.

```markdown
---
title: "A quoted title"
date: 2026-10-24
slug: custom-slug
tags: [myna, static-sites]
draft: false
layout: post.html
type: post
custom_field: 42
---
```

Supported:

- `key: value` lines, one key per line.
- Single- or double-quoted strings with `\"`, `\'`, `\\`, `\n`, `\t` escapes.
- Integers, booleans (`true`/`false`), and `null` / `none` / `~`.
- Inline lists `[a, "b c"]`; `tags: myna, tests` is accepted as a convenience.
- Blank lines and `#` comments inside front matter.
- Arbitrary extra keys, exposed as `page.extra.<key>` in templates.

Not supported: block mappings, nested YAML, multi-line scalars, anchors. Duplicate keys,
malformed lines, missing closing delimiters, invalid dates, and missing `title` are errors
with file and line information.

### Fields

| Field | Type | Meaning |
| :--- | :--- | :--- |
| `title` | string | Required. |
| `date` | `YYYY-MM-DD` | Makes the item a post by default. |
| `slug` | string | Defaults to title, then filename. Sanitised. |
| `tags` | list | Creates `/tags/<tag>/` pages from posts. |
| `draft` | boolean | Excluded unless `--drafts`. |
| `layout` | string | Template filename; defaults to `post.html` or `page.html`. |
| `type` | `page` or `post` | Defaults to `post` when dated, otherwise `page`. An explicit `type: post` requires `date`. |

Derived template values include `page.slug`, `page.url`, `page.reading_time`, `page.extra`,
and `page.source`.

Slugs are lowercase, ASCII-folded, hyphenated, and bounded. Path traversal is sanitised:
`../../etc/passwd` → `etc-passwd`, `..` → `untitled`. Output paths independently reject
`..` segments.

## Templates

Templates are in `templates/` (configurable). Syntax at a glance:

```html
{{ page.title }}
{{ page.title | escape }}
{{ content | safe }}
{{ page.date | date("%B %d, %Y") }}
{{ page.url | url }}

{% if page.tags %}tags{% elif page.type == "post" %}post{% else %}page{% endif %}

{% for post in posts %}
  <a href="{{ post.url | url }}">{{ post.title }}</a>
  {% if not loop.last %},{% endif %}
{% endfor %}

{% include "partials/footer.html" %}
{# a comment #}
```

- Auto-escaping is on. `safe` opts out; `escape` explicitly escapes.
- Filters compose left-to-right: `{{ "Hello World" | lower | slug }}`.
- Filters: `escape`, `safe`, `date`, `upper`, `lower`, `slug`, `url`, `join`, `default`,
  `length`, `truncate`. `url` prefixes site-root paths with `site.base_path`; pass an
  explicit base (`{{ page.url | url("/docs") }}`) to override it.
- Conditionals support `{% if %}`, `{% elif %}`, `{% else %}`, `{% endif %}`.
- Loops expose `loop.index`, `loop.index0`, `loop.first`, `loop.last`, `loop.length`.
- Layout inheritance supports `{% extends "base.html" %}` and named
  `{% block name %}...{% endblock %}` regions.
- Expression syntax is a small Python-like dialect (comparisons, `and`/`or`/`not`,
  arithmetic, indexing, dotted lookup). `true`/`false`/`none` are accepted. Builtins are
  disabled. Missing names/attributes render empty.
- Template errors name the template file and line.

The site data available to every template includes `site`, `page`, `content`, `posts`,
`all_posts`, `page_number`, `total_pages`, `paginator`, `prev_url`, `next_url`, `tag`,
`tags`, and `now`. Posts are newest-first; the homepage's `posts` is the current page when
pagination is enabled.

## Markdown subset

Implemented:

- ATX headings `#`–`######`, paragraphs, hard breaks.
- Emphasis/strong with `*`, `**`, `_`, `__`, and `***`.
- Inline code, fenced code (``` and ~~~, optional language class), indented code.
- Fenced code with a language class is highlighted at build time by the vendored
  [MynaShine](#syntax-highlighting) highlighter.
- Blockquotes, nested ordered/unordered lists, horizontal rules.
- Links, images, scheme and email autolinks.
- Raw HTML blocks/inline passthrough by design.
- Text is HTML-escaped by default; code is always escaped and never interpreted.

Out of scope: tables, footnotes, definition lists, task lists, smart punctuation, heading
IDs, and full CommonMark edge cases. Raw HTML is trusted input; add your own sanitisation
if content is untrusted.

### Syntax highlighting

Fenced code blocks with an info string are highlighted by
[MynaShine](https://github.com/jhicksdev/mynashine), which is vendored under
`src/mynashine/` so MynaPress keeps its zero-dependency promise. Supported languages are
Myna (`myna`, `mn`), Python, JavaScript, TOML, YAML, Bash/shell, HTML/XML, Markdown, and
MynaPress templates. Unknown languages and malformed snippets degrade to escaped text;
highlighting never fails a build.

```myna
@main {
  p("highlighted")
}
```

becomes a `<pre class="hl"><code class="language-myna">` block with `hl-*` token spans.
Unlabelled and indented code blocks remain plain escaped `<pre><code>`.

The stylesheet is the `mynashine` theme block between `/* mynashine:start */` and
`/* mynashine:end */` in `assets/style.css`. Sites using a different stylesheet need the
same block; generate it with `mynashine css --theme mynapress` or copy it from this
repository. See [`src/mynashine/VENDORED.md`](src/mynashine/VENDORED.md) for the sync
procedure when MynaShine changes.

## Configuration

`site.toml` is parsed with stdlib `tomllib` and validated.

| Key | Type | Default |
| :--- | :--- | :--- |
| `title` | string | required if the file exists |
| `description` | string | `""` |
| `base_url` | string | `""`; must be `http://` or `https://` when set |
| `base_path` | string | `""`; URL path prefix for subpath hosting, e.g. `"/docs"` |
| `author` | string | `""` |
| `language` | string | `"en"` |
| `paginate` | integer | `10`; `0` disables pagination |
| `content_dir` | string | `"content"` |
| `output_dir` | string | `"build"` |
| `templates_dir` | string | `"templates"` |
| `static_dir` | string | `"assets"` |

Relative paths in `site.toml` resolve from the config file's directory. CLI `--src`/`--out`
resolve from the current directory. Unknown keys, wrong types, a missing title, a bad
`base_url` or `base_path`, negative pagination, and TOML errors fail with a clear message.
If `site.toml` is absent and no `--config` was supplied, defaults are used; an explicit
`--config` path that does not exist is an error.

When `base_path` is set, templates should prefix internal links with the `url` filter
(`{{ post.url | url }}`), and MynaPress prefixes root-relative Markdown links and images
automatically. The dev server serves the site under the same prefix. Absolute,
protocol-relative, fragment, and relative URLs are left unchanged.

Output directories are protected:

- The output may not be the site root, contain the site root, or overlap `content_dir`,
  `templates_dir`, or `static_dir`.
- Builds are staged in a temporary sibling and swapped into place only after success. A
  content or template error leaves the previous output untouched.
- A non-empty existing output directory must contain `.mynapress-output`, the marker
  written by MynaPress. Build may initialize an empty directory, but refuses to replace a
  non-empty unmarked one; `clean` removes only marked output.
- `clean` only removes output directories inside the site root.

Assembly behaviour:

- Static files keep their directory name (`assets/style.css` → `build/assets/style.css`).
- `favicon.ico`, `favicon.png`, or `favicon.svg` at the site root is copied to the output.
- `content/index.md` renders `/`; a nested `index.md` gets a normal slug URL. Otherwise a
  homepage is generated from `index.html` or a minimal built-in fallback.
- Posts generate tag pages and optional `/page/2/` pagination.
- With `base_url`, MynaPress emits RSS 2.0 (`rss.xml`), `sitemap.xml`, and `robots.txt` with
  absolute URLs.

## Development

All application and test code is `.myna`. There is no Python product code; `bin/mynapress`
is a shell convenience wrapper and the CI workflow is YAML.

```bash
# tests (Myna test harness)
myna run tests/run_tests.myna

# build both fixture sites
myna run src/main.myna build --config fixtures/site/site.toml
myna run src/main.myna build --config fixtures/blog/site.toml

# documentation site
myna run src/main.myna build
```

CI runs the tests and builds the fixture and documentation sites on Python 3.11 and 3.12.
The `pages.yml` workflow runs the tests, builds the docs with the configured
`base_url`/`base_path`, and deploys `build/` to GitHub Pages on pushes to `main`.

Tests cover slugify (Unicode, punctuation, traversal), front matter (quoted values, lists,
malformed input, unknown keys), Markdown block/inline forms and escaping, syntax
highlighting, template filters and layouts, multi-level inheritance, missing variables and
malformed tags, dates, pagination, config validation, explicit missing config, fixture
builds, drafts, tags, expected output, output path safety, failed-build preservation, MIME
types, 404s, and CLI cleanup.

### Architecture

`src/main.myna` is a thin entrypoint:

```myna
@target(py)
<sys
<.cli:{run_cli}

@main {
  sys.exit(run_cli(sys.argv[1:]))
}
```

Each responsibility lives in its own module, and modules use ordinary Myna relative
imports:

```myna
# src/content.myna
<.util:{DotDict, parse_date, reading_time, slugify}
```

`myna run` resolves those imports lazily from `src/`, so modules have real namespaces,
normal Python tooling works, and the tests can import or inspect them independently. The
loader workaround used during the original Myna 0.3.0 evaluation has been removed now that
Myna 0.4.0 supports sibling `.myna` modules; the history and the four original
reproductions are in `GAPS.md`.

`src/mynashine/` is a vendored copy of MynaShine's portable highlighting core. It is the
only third-party source in the repository, is MIT-licensed like MynaPress, and keeps
build-time highlighting dependency-free. The CLI, filesystem, demo, and site-postprocessor
modules are intentionally not vendored; see
[`src/mynashine/VENDORED.md`](src/mynashine/VENDORED.md).

## Known limitations

- Full rebuilds only; no incremental cache. Stale output is worse than a slow build.
- `clean` refuses output directories outside the site root. External output can still be
  built, but remove it manually.
- Flat post URLs; nested source folders do not affect URLs.
- Python target only; no Node target.
- Small YAML subset and CommonMark-ish Markdown; no tables, footnotes, or smart
  punctuation.
- Syntax highlighting covers fenced blocks with a language class only. Unlabelled and
  indented code stay plain, and the vendored highlighter must be re-synced when MynaShine
  changes.
- The highlighter CSS is a stylesheet block, not an automatic per-site asset; custom
  stylesheets must include it.
- Raw HTML is trusted and passed through.
- Tag pages are not paginated.
- No feeds/sitemap unless `base_url` is configured.
- Requires Myna 0.5.0+ for native `.myna` module resolution and the vendored highlighter.
- No committed generated HTML. CI builds the docs.

More detail is in the generated docs under `content/limitations.md` and in `GAPS.md`.

## License

MIT. See [LICENSE](LICENSE).
