# Myna gaps found while building MynaPress

Reproductions are against Myna 0.3.0 at `/home/joseph/Projects/myna`.

## G1. Editable install fails from the source tree

**Severity:** painful (local development only)

```bash
python3 -m pip install -e /home/joseph/Projects/myna
```

**Expected:** installs `myna-lang` and the `myna` console script.

**Actual:** setuptools flat-layout discovery sees both `myna/` and `skills/` and aborts:

```text
error: Multiple top-level packages discovered in a flat-layout: ['myna', 'skills'].
```

A plain `uv pip install -e` fails the same way. This host also lacks `pip`/`ensurepip`.

**Workaround:** run Myna from its source tree with `PYTHONPATH=/home/joseph/Projects/myna`.
MynaPress documents this convention. CI installs the released wheel instead.

## G2. No import mechanism for `.myna` application modules

**Severity:** painful (blocks the suggested module layout)

```myna
# main.myna
@target(py)
<.util:{slugify}
p(slugify("Hello"))
```

**Expected:** either the compiler bundles/executes `util.myna`, or `myna run` supports a
source-relative module graph.

**Actual:** the compiler emits Python `from .util import slugify`. `myna run` executes the
compiled code with `__name__ = "__main__"` and no package context, so Python cannot resolve
the import. Even a plain `import util` would look for `util.py`, never `util.myna`.
There is no documented loader hook.

Reproduction:

```bash
cd /tmp/opencode/relpkg   # main.myna above; helper.py is irrelevant
PYTHONPATH=/home/joseph/Projects/myna python3 -m myna run main.myna
# ImportError: attempted relative import with no known parent package
```

**Workaround:** MynaPress keeps each responsibility in a separate `.myna` file but loads
them through a small Myna bootstrap (`src/main.myna`) that compiles each source with
`myna_to_python` and `exec`s it into one shared namespace. Tests use the same technique.
This is a real architectural tax: modules share one global namespace and cannot be
imported by ordinary Python tooling.

## G3. `dataclass` creation fails when `exec` is used with an unregistered module

**Severity:** painful (only for loaders, REPLs, and test harnesses)

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

Python's `dataclasses` resolves annotations through `sys.modules[cls.__module__]`; the
module is not registered. This is fallout from the custom loader required by G2.

**Workaround:** MynaPress creates a real `types.ModuleType("mynapress_runtime")`, registers
it in `sys.modules`, and executes all compiled modules into its `__dict__`.

## G4. Compound types in `:?` compile to invalid Python

**Severity:** painful

```myna
@target(py)
value = ["a"]
? value :? [s] {
  p("list")
}
```

**Expected:** `isinstance(value, list[str])` (or a runtime-checkable equivalent) works.

**Actual:** Myna emits `isinstance(value, [s])`, which raises `NameError: name 's' is not
defined`. The same shape appears for `{s:A}`.

**Workaround:** use Python's `isinstance(value, list)` / `isinstance(value, dict)` from Myna
code. This is less precise but works at runtime.
