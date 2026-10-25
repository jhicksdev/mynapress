# Myna gaps found while building MynaPress

Reproductions were originally filed against Myna 0.3.0. All four entries below were
re-checked against Myna `main` at `2ca0826` (2026-10-24). The fixes ship in the
`myna-lang` 0.4.0 package.

| Gap | Status |
| :-- | :-- |
| G1. Editable install fails | **Fixed** in `04a699c` |
| G2. No import mechanism for `.myna` modules | **Fixed** in `2ca0826` |
| G3. `dataclass` fails under `exec` | **Not a Myna defect** (CPython behaviour) |
| G4. Compound types in `:?` | **Fixed** in `04a699c` |

## G1. Editable install fails from the source tree — fixed

**Severity:** painful (local development only)

```bash
python3 -m pip install -e /home/joseph/Projects/myna
```

**Expected:** installs `myna-lang` and the `myna` console script.

**Actual (0.3.0):** setuptools flat-layout discovery saw both `myna/` and `skills/` and
aborted:

```text
error: Multiple top-level packages discovered in a flat-layout: ['myna', 'skills'].
```

**Resolution:** `pyproject.toml` now declares `packages = ["myna", "myna.backends",
"myna.frontends"]`. Verified with a clean `uv venv` + `uv pip install -e .`; the `myna`
console script works. The `PYTHONPATH=/home/joseph/Projects/myna` fallback is no longer
needed for local development.

## G2. No import mechanism for `.myna` application modules — fixed

**Severity:** painful (blocked the suggested module layout)

```myna
# main.myna
@target(py)
<.util:{slugify}
p(slugify("Hello"))
```

**Expected:** either the compiler bundles/executes `util.myna`, or `myna run` supports a
source-relative module graph.

**Actual (0.3.0):** the compiler emitted Python `from .util import slugify`, but `myna run`
executed the compiled code with `__name__ = "__main__"` and no package context, so Python
could not resolve the import. Even a plain `import util` looked for `util.py`, never
`util.myna`. There was no documented loader hook.

**Resolution:** `myna run` now installs a scoped import hook (`myna/loader.py`) rooted at
the entry file's directory:

- `<.util:{f}` → `util.myna` beside the entry file.
- `<.pkg.mod:{f}` → `pkg/mod.myna`; `pkg` may have `__init__.myna` or act as a namespace
  directory.
- Parent-relative imports (`<..util:{f}`) work from inside subpackages.
- Modules are compiled lazily on first import and cached through `sys.modules`.
- Absolute imports keep their normal Python meaning.
- Lex/parse errors in imported files now name the originating `.myna` path.

MynaPress can therefore drop its exec-into-one-namespace bootstrap and use ordinary
`.myna` modules, restoring module isolation and normal Python tooling.

**Remaining limits:** `myna compile` is still per-file (its relative imports expect a real
Python package), and `myna run -t js` is single-file. `myna bundle` covers the
single-artifact case for Python.

## G3. `dataclass` creation fails when `exec` is used with an unregistered module — not a Myna defect

**Severity:** informational (loaders, REPLs, test harnesses)

```myna
@target(py)
<types
<myna:{myna_to_python}
ns = types.ModuleType("demo").__dict__
exec(compile(myna_to_python("$$P(x:i)"), "demo", "exec"), ns)
```

**Expected:** `P` is created; a `$$` dataclass is just a module-level class.

**Actual:**

```text
AttributeError: 'NoneType' object has no attribute '__dict__'
```

**Analysis:** this reproduces identically in plain Python. `dataclasses` resolves string
annotations through `sys.modules.get(cls.__module__).__dict__`; the module is not
registered, so the lookup returns `None`. Registering the module first fixes it — which is
what MynaPress's loader already does. With the G2 fix, `myna run` registers every module it
loads, so the original failure no longer occurs in that path.

This was fallout from the custom loader G2 required, not a language defect.

## G4. Compound types in `:?` compile to invalid Python — fixed

**Severity:** painful

```myna
@target(py)
value = ["a"]
? value :? [s] {
  p("list")
}
```

**Expected:** a runtime check that `value` is a list.

**Actual (0.3.0):** Myna emitted `isinstance(value, [s])`, raising
`NameError: name 's' is not defined`. `{s:A}` and `{i}` failed the same way, and the JS
target emitted `x instanceof [s]`, which throws at runtime.

**Resolution:** `:?` now checks the runtime origin type — `[T]` → `list`,
`{K:V}` → `dict`, `{T}` → `set`, `Name[T]` → `Name` — in both backends. A tuple of types
remains a union (`x :? (i, s)` is `isinstance(x, (int, str))`), now documented in `SPEC.md`
and the Myna skill. Covered by Python and cross-target JS tests.
