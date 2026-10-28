---
title: Templates
---

Templates live in `templates/` (configurable). A page chooses a layout with `layout:`;
the default is `post.html` for posts and `page.html` for pages. If the requested layout is
missing, MynaPress falls back to `page.html`.

## Variables

```html
<h1>{{ page.title }}</h1>
{{ content | safe }}
```

`{{ expression }}` is HTML-escaped by default. `{{ expression | safe }}` opts out. Use it
for Markdown output (`content`) and other trusted HTML only.

Expressions support dotted lookup, indexing, literals, comparisons, `and`, `or`, `not`,
`in`, and normal arithmetic. `true`, `false`, and `none` are accepted aliases for
`True`, `False`, and `None`. Missing names and attributes render as an empty string, which
makes `default` useful:

```html
{{ page.extra.subtitle | default("Untitled") }}
```

The expression evaluator runs with Python builtins disabled. It is intentionally small;
it is not a general-purpose scripting language. Dotted lookup works for dataclass fields,
the `loop`, `tag`, and `paginator` helper objects, and `page.extra`.

## Filters

Filters compose left-to-right:

```html
{{ "Hello World" | lower | slug }}
```

| Filter | Behaviour |
| :--- | :--- |
| `escape` | HTML-escape explicitly and mark the result safe. |
| `safe` | Mark a value as trusted HTML; disables auto-escaping for it. |
| `date` | Format an ISO date. Default `%b %d, %Y`; pass a `strftime` format. |
| `upper` / `lower` | Change case. |
| `slug` | Slugify with the same rules as content slugs. |
| `url` | Prefix site-root paths with `site.base_path`; optional explicit base argument. |
| `join` | Join a list; optional separator, default `", "`. |
| `default` | Replace `None`, empty string, or empty list. |
| `length` | Length of a string, list, or mapping. |
| `truncate` | Truncate text; optional limit (default 80) and suffix (default `…`). |

Unknown filters are template errors that name the template file and line.

## Conditionals

```html
{% if page.extra.featured %}
  <strong>Featured</strong>
{% elif page.type == "post" %}
  <span>Post</span>
{% else %}
  <span>Page</span>
{% endif %}
```

## Loops

```html
{% for post in posts %}
  <a href="{{ post.url | url }}">{{ post.title }}</a>
{% endfor %}
```

Inside a loop, `loop.index` (1-based), `loop.index0`, `loop.first`, `loop.last`, and
`loop.length` are available. Loops may nest. Empty or missing collections render nothing.

Use the `url` filter for internal links when the site is served from a subpath. It is
harmless when `base_path` is empty, so it is the recommended form everywhere:

```html
<link rel="stylesheet" href="{{ '/assets/style.css' | url }}">
<a href="{{ '/' | url }}">Home</a>
```

## Includes

```html
{% include "partials/footer.html" %}
```

Paths are resolved under `templates/` and may not escape it. Recursive includes are
errors.

## Layout inheritance

Templates support one level (and technically chains) of `{% extends %}` plus named
`{% block %}` regions:

```html
{# child.html #}
{% extends "base.html" %}
{% block title %}About{% endblock %}
{% block content %}<p>About this site.</p>{% endblock %}
```

```html
{# base.html #}
<title>{% block title %}Default{% endblock %}</title>
<main>{% block content %}{% endblock %}</main>
```

A layout that extends another is rendered through its parent. Content outside blocks in a
child is ignored. Blocks without an override use their default content.

## Comments and errors

```html
{# This is not output. #}
```

Malformed tags, unknown filters, missing `endif`/`endfor`/`endblock`, missing includes, and
recursive includes all fail the build with `template.html:line: message`.
