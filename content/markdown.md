---
title: Markdown subset
slug: markdown
---

MynaPress implements a CommonMark-ish subset. It is intentionally smaller than full
CommonMark, and the supported behaviour is tested in `tests/test_markdown.myna`.

## Blocks

- ATX headings `#` through `######`.
- Paragraphs; soft line breaks become newlines in the HTML source.
- Hard line breaks from two trailing spaces or a trailing backslash.
- Fenced code blocks with ``` or ~~~, including an optional language class.
- Indented code blocks (four spaces or a tab).
- Blockquotes with `>` and nested block content.
- Ordered and unordered lists, including nested lists. Tight single-paragraph items are
  emitted without `<p>` wrappers.
- Horizontal rules (`***`, `---`, `___`).
- Raw HTML blocks for common block tags, HTML comments, and doctypes.
- Raw HTML passthrough is **by design**; treat content as trusted input.

## Inline

- Emphasis and strong with `*`, `**`, `_`, `__`, and combined `***`.
- Inline code spans with backticks; code content is escaped, never interpreted.
- Links `[text](url "title")` and images `![alt](src "title")`.
- Autolinks `<https://example.com>` and `<me@example.com>`.
- Inline raw HTML tags and comments pass through.
- Backslash escaping of punctuation.
- Text is HTML-escaped by default, including `&`, `<`, `>`, quotes, and apostrophes.

## Not supported

- Tables, footnotes, definition lists, task lists, smart punctuation, heading IDs, and
  syntax highlighting.
- Full CommonMark edge cases (lazy list continuation, complex emphasis flanking, entity
  preservation, HTML block type 6/7 subtleties). The renderer aims for predictable output,
  not spec conformance.

## Safety notes

Raw HTML blocks and inline tags pass through. Markdown text is escaped. Code blocks and
code spans are always escaped. If you accept untrusted Markdown, add your own sanitisation
step; MynaPress does not sanitise raw HTML.
