---
title: Command line
slug: cli
---

```bash
mynapress build [--config site.toml] [--src content] [--out build] [--drafts]
mynapress serve [--port 8000] [--watch] [--config site.toml] [--src content] [--out build] [--drafts]
mynapress new "Post title" [--config site.toml]
mynapress clean [--config site.toml]
mynapress --version
```

From a source checkout, use `myna run src/main.myna <command> ...` instead of
`mynapress <command> ...`.

## `build`

Discovers Markdown, renders pages and posts, copies static files, builds tag pages and
pagination, and writes feeds when `base_url` is configured. A full rebuild is performed
every time; see [Limitations](/limitations/). The build is staged in a temporary sibling
directory and swapped into place only after success, so a failed build leaves the previous
output intact.

- `--config PATH` — config file (default `site.toml`). An explicitly passed path must
  exist; only an absent default `site.toml` falls back to defaults.
- `--src PATH` — override `content_dir`. Relative paths are resolved from the current
  directory.
- `--out PATH` — override `output_dir`. Relative paths are resolved from the current
  directory.
- `--drafts` — include pages whose front matter says `draft: true`.

The output directory may not be the site root, contain it, or overlap the content,
template, or static directories. A non-empty existing output directory must contain the
`.mynapress-output` marker written by MynaPress; otherwise the build refuses to replace it.
An empty directory may be initialized by the first build.

Output ends with a one-line summary such as:

```text
Built 5 page(s), 3 post(s), 3 tag page(s), 1 asset(s) in 0.01s
```

Content errors print `error: path/to/file.md:line: message` where a line is available and
exit 1. They do not dump a Python traceback.

## `serve`

Builds, then serves the output directory using Python's `http.server` on
`127.0.0.1:8000` by default. Static MIME types come from Python's `mimetypes`; unknown
paths return 404.

`--watch` polls content, templates, static files, favicon files, and `site.toml` once per
second. On a change it rebuilds and prints `[watch] change detected; rebuilding...`.
Polling is intentional and simple; there are no websockets.

## `new`

Creates a Markdown file with front matter and a date-based filename:

```text
content/2026-10-24-a-first-post.md
```

It refuses to overwrite an existing file and adds `-2`, `-3`, and so on instead.

## `clean`

Removes the configured output directory and `.mynapress-cache.json` if present. The output
must be inside the site root and must contain `.mynapress-output`; unmarked directories and
outputs outside the root are refused. External output can be built, but must be removed
manually.

## Exit codes

- `0` — success.
- `1` — build, config, content, template, server, or cleanup error.
- `2` — unknown command.

Invalid flags and missing values also exit 1 with an actionable message.

`--version` prints the MynaPress version and exits 0.
