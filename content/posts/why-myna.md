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
minimal reproductions, expected versus actual behaviour, and the workaround used here.

The short version:

- Dataclasses, pattern matching, comprehensions, and error handling are pleasant and
  compact.
- The lack of a `.myna` module import mechanism is the biggest architectural tax. We use a
  small loader built on the public compiler.
- Compound runtime type checks (`x :? [s]`) produce invalid Python; `isinstance` is the
  workaround.
- Editable installs from the Myna source tree fail under setuptools' flat-layout
  discovery because of the `skills/` directory.

None of that is hidden. The project is meant to be a fair report from the field.
