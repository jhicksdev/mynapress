---
title: Why MynaPress is written in Myna
date: 2026-10-24
tags: [myna, dogfooding]
---

MynaPress is a dogfooding test. Myna is a dense, brace-delimited language that transpiles
to Python and JavaScript; a static site generator exercises files, paths, dates, text
processing, templates, HTTP, and tests without needing a framework.

The useful output is not just a working site generator. It is the list of things that were
surprising, awkward, or impossible in Myna 0.3.0. Those findings live in `GAPS.md`, with
minimal reproductions, expected versus actual behaviour, and their upstream resolution.

The short version:

- Dataclasses, pattern matching, comprehensions, and error handling are pleasant and
  compact.
- The missing `.myna` module import mechanism was the biggest architectural tax. Myna now
  resolves sibling modules natively, and MynaPress uses ordinary relative imports.
- Compound runtime type checks (`x :? [s]`) used to produce invalid Python. They now work,
  so the content model uses them directly.
- Editable installs from the Myna source tree failed under setuptools' flat-layout
  discovery because of the `skills/` directory. The Myna packaging was fixed.

The first pass still shipped with the workarounds. The final code exercises the fixed
behaviour instead of hiding it, and `GAPS.md` keeps the historical reproductions.
