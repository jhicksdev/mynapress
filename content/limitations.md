---
title: Known limitations
slug: limitations
---

MynaPress v0.2 is deliberately small. These are the known limitations, stated plainly.

- **Full rebuilds only.** There is no incremental cache. A 500-page site is expected to
  build in a couple of seconds, but every build rewrites everything.
- **Protected output locations.** Build refuses to replace a non-empty unmarked directory,
  `clean` removes only marked output, and `clean` only works inside the site root. External
  output can be built but must be removed manually.
- **Flat post URLs.** Posts live at `/<slug>/`, not `/YYYY/MM/<slug>/`. Nested source
  folders are allowed but do not affect URLs.
- **One site per invocation.** No multi-site builds, themes marketplace, plugins, search
  index, i18n, image optimisation, or hot-reload websockets.
- **Python target only.** All files declare `@target(py)`. A Node target is a future
  stretch goal; nothing pretends cross-target support exists.
- **Small YAML subset.** Front matter supports scalars, booleans, integers, quoted
  strings, and inline lists only — no block mappings, multi-line strings, or anchors.
- **CommonMark-ish, not CommonMark.** Footnotes, definition lists, and smart punctuation
  are out of scope. See [Markdown](/markdown/).
- **Fenced code needs a language to be highlighted.** Blocks with an info string use the
  vendored MynaShine highlighter; unlabelled and indented blocks stay plain. See
  [Markdown](/markdown/).
- **Raw HTML is trusted.** It passes through by design. Sanitise untrusted content
  yourself.
- **Template expressions are a small Python-like dialect** evaluated without builtins.
  Complex expression logic belongs in the site builder, not a template.
- **Tag pages are not paginated.** They list all posts for a tag.
- **No feed unless `base_url` is set.** Absolute URLs are required for a valid feed and
  sitemap, so those files are omitted when no base URL is configured.
- **Requires Myna 0.5.0+.** MynaPress uses native `.myna` module imports and a vendored
  MynaShine core, so it needs a Myna build with sibling module resolution. Older
  single-file 0.3.0 builds cannot run it.
- **No committed build output.** The documentation site is generated in CI (or locally
  with `myna run src/main.myna build`); generated HTML is not committed.

If you need one of these for a real site, open an issue with the use case. The project is
small on purpose, not because it wants to stay limited.
