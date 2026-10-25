---
title: Content model
---

Content is discovered recursively from `content/` (or the configured `content_dir`). Only
`.md` files are read. Nested folders are fine for organisation, but current URLs are flat:
a post's URL is `/<slug>/` regardless of its source folder.

## Front matter

Front matter is delimited by a first line of exactly `---` and a later line of `---` (or
`...`). The body follows. This is deliberately **not full YAML**.

Supported value forms:

```markdown
---
title: "A quoted title"
date: 2026-10-24
slug: custom-slug
tags: [myna, static-sites]
draft: false
layout: post.html
type: post
arbitrary_key: 42
another_key: true
---
```

- Keys are `key: value` on one line.
- Quoted strings may use single or double quotes. `\"`, `\'`, `\\`, `\n`, and `\t` are
  unescaped.
- Integers (`-3`, `42`), booleans (`true`, `false`, case-insensitive), and `null` /
  `none` / `~` are recognised.
- Inline lists `[a, b]` are supported. Items may be quoted. `tags: myna, tests` is also
  accepted as a convenience.
- Blank lines and lines starting with `#` inside front matter are ignored.
- Duplicate keys, malformed lines, and missing closing delimiters are errors that name the
  source file and line.
- There is no block YAML, no nested mapping syntax, and no multi-line scalar.

Unknown keys are collected in `page.extra`. In templates use `{{ page.extra.arbitrary_key }}`.

## Fields

| Field | Type | Meaning |
| :--- | :--- | :--- |
| `title` | string | **Required.** |
| `date` | `YYYY-MM-DD` | Makes the file a post by default. Invalid dates are errors. |
| `slug` | string | Defaults to the title, then the filename. Always sanitised. |
| `tags` | list | Drives `/tags/<tag>/` pages. |
| `draft` | boolean | Excluded from builds unless `--drafts`. |
| `layout` | string | Template filename. Defaults to `post.html` for posts and `page.html` for pages. |
| `type` | `page` or `post` | Defaults to `post` when `date` is present, otherwise `page`. An explicit `type: post` requires `date`. |

## Derived values available in templates

- `page.slug` — lowercase, ASCII-folded, hyphenated.
- `page.url` — `/` for `index.md`, otherwise `/<slug>/`.
- `page.reading_time` — minutes at 200 words per minute, minimum 1.
- `page.extra` — unknown front matter fields.
- `page.source` — source filename for error messages.

Slugs are sanitised with Unicode NFKD folding, ASCII transliteration, lowercasing, and
replacement of every non-alphanumeric run with `-`. `../etc/passwd` becomes `etc-passwd`;
`..` becomes `untitled`. Output paths independently reject `..` segments, so a bad slug
cannot escape the build directory.

## Pages, posts, collections

Pages do not participate in post collections. Posts are sorted by `(date, slug)`, newest
first. The homepage receives `posts` (the current page of posts), `all_posts` (everything),
and paginator fields. Tag pages receive `tag` and that tag's posts.

If `content/index.md` exists it renders to `/`; otherwise MynaPress generates a simple
homepage from `index.html` if available, or a built-in list fallback. Posts also remain
available at their own URLs.
