---
title: Command line
slug: cli
---

```bash
myna run src/main.myna build [--config site.toml] [--src content] [--out build] [--drafts]
myna run src/main.myna serve [--port 8000] [--watch] [--config site.toml] [--src content] [--out build] [--drafts]
myna run src/main.myna new "Post title" [--config site.toml]
myna run src/main.myna clean [--config site.toml]
```

## `build`

Discovers Markdown, renders pages and posts, copies static files, builds tag pages and
pagination, and writes feeds when `base_url` is configured. A full rebuild is performed
every time; see [Limitations](/limitations/).

- `--config PATH` — config file (default `site.toml`).
- `--src PATH` — override `content_dir`. Relative paths are resolved from the current
  directory.
- `--out PATH` — override `output_dir`. Relative paths are resolved from the current
  directory.
- `--drafts` — include pages whose front matter says `draft: true`.

Output ends with a one-line summary such as:

```text
Built 5 page(s), 3 post(s), 3 tag page(s), 1 asset(s) in 0.01s
```

Content errors print `error: path/to/file.md:line: message` and exit 1. They do not dump a
Python traceback.

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

Removes the configured output directory and `.mynapress-cache.json` if present. It refuses
to delete the site root or `/`.

## Exit codes

- `0` — success.
- `1` — build, config, content, template, server, or cleanup error.
- `2` — unknown command.

Invalid flags and missing values also exit 1 with an actionable message.
