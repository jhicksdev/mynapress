---
title: Configuration
---

`site.toml` is parsed with Python's standard-library `tomllib`. MynaPress validates keys,
types, and values and fails with a clear message instead of silently ignoring mistakes.

```toml
title = "My Site"
description = "Notes and things."
base_url = "https://example.com"
author = "Ada"
language = "en"
paginate = 10
content_dir = "content"
output_dir = "build"
templates_dir = "templates"
static_dir = "assets"
```

| Key | Type | Default | Notes |
| :--- | :--- | :--- | :--- |
| `title` | string | required if the file exists | Site title. |
| `description` | string | `""` | Used by feeds and available in templates. |
| `base_url` | string | `""` | Must start with `http://` or `https://` when set. Enables RSS, sitemap, and robots. |
| `author` | string | `""` | Available as `site.author`. |
| `language` | string | `en` | HTML language and RSS language. |
| `paginate` | integer | `10` | Posts per homepage page. `0` disables pagination. |
| `content_dir` | string | `content` | Markdown source directory. |
| `output_dir` | string | `build` | Replaced atomically on every successful build. |
| `templates_dir` | string | `templates` | Template source directory. |
| `static_dir` | string | `assets` | Copied into the output as a directory of the same basename. |

Relative paths in `site.toml` are resolved relative to the config file's directory. CLI
`--src` and `--out` overrides are resolved from the current working directory. Unknown
keys are errors. Wrong types, a missing `title`, a bad `base_url`, negative `paginate`, and
TOML syntax errors are errors.

If `site.toml` does not exist and no `--config` was supplied, MynaPress uses defaults
(`title = "MynaPress"`, current directory, `content/`, `templates/`, `assets/`, `build/`,
pagination 10). An explicitly passed `--config` path must exist.

## Output safety

MynaPress treats the output directory as disposable but protects everything else:

- `output_dir` may not be the site root, contain the site root, or overlap
  `content_dir`, `templates_dir`, or `static_dir`.
- Builds are written to a temporary sibling directory and swapped into place only after a
  successful build. A content or template error leaves the previous output untouched.
- A non-empty existing output directory must contain `.mynapress-output`, the marker
  MynaPress writes on every successful build. Build may initialize an empty directory, but
  refuses to replace a non-empty unmarked one; `clean` removes only marked output.
- `clean` only removes output inside the site root. You can build to an external path, but
  remove it manually.

The marker is an implementation detail; it is safe to delete the whole output directory,
but do not commit generated output.

## Generated output

- Every content page and post at its URL.
- `/` from the root `content/index.md` or a generated homepage.
- `/page/2/`, `/page/3/`, ... when pagination is enabled.
- `/tags/<tag>/` for every tag used by a post.
- Static files preserved under `build/<static basename>/`.
- `favicon.ico`, `favicon.png`, or `favicon.svg` copied to the output root if present.
- `rss.xml`, `sitemap.xml`, and `robots.txt` when `base_url` is set.

## Caching

There is no incremental cache in v0.1. Every build is a full rebuild. This is a deliberate
correctness choice: stale cache output is worse than a fast rebuild. `clean` still removes
`.mynapress-cache.json` if an older build created one.
