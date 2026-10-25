---
title: MynaPress
type: page
layout: index.html
---

MynaPress turns a folder of Markdown files into a fast, self-contained HTML site.
It has **zero runtime dependencies** beyond Myna and the Python standard library, and
every line of application and test code is `.myna`.

It exists for two reasons: to be a genuinely useful small static site generator, and to
find out what building a real project in Myna feels like. The friction we found is
recorded honestly in `GAPS.md`.

```bash
myna run src/main.myna build
myna run src/main.myna serve --watch
```

Start with [Getting started](/getting-started/), or read the [CLI](/cli/),
[content model](/content-model/), [template syntax](/templates/),
[Markdown subset](/markdown/), and [configuration](/configuration/) references.
