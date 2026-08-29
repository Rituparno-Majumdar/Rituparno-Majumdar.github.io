# stubcheck — Stop Shipping Functions That Do Nothing

AI code completion fills in method signatures fast — but it fills them with
`pass`, `...`, or just a docstring when the implementation is forgotten.
The function exists, type-checks, and runs. It returns `None` to every
caller. The failure surfaces much later as a confusing `NoneType` error or a
silent downstream no-op.

```python
class OrderHandler(BaseHandler):
    def on_created(self, order):
        pass              # ← AI stubbed this, you shipped it

    def total(self, items):
        ...               # ← Ellipsis stub, returns None
```

`handler.on_created(order)` runs without error. The order is never
persisted. Nobody notices until the audit.

`stubcheck` walks the AST, finds every function whose body is a **silent
no-op** (`pass`-only or `...`-only, optionally with a docstring), and flags
it so you can fill in the real logic before it ships. `@abstractmethod`
functions and `.pyi` type-stub files are excluded — those stubs are correct.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/stubcheck.git
cd stubcheck
pip install -e .

# Scan everything under src/
stubcheck src/

# CI gate — fail the build on stub functions
stubcheck . --check

# Only the most dangerous: body is ONLY pass/ellipsis, no docstring
stubcheck src/ --min-severity high

# Machine-readable output
stubcheck . --json

# Include test files too (stubs are sometimes legit in tests)
stubcheck . --include-test
```

## Real Example

```python
# handlers.py
class Cache:
    def warm(self, keys):
        pass  # never implemented

    def evict(self, key):
        """TODO: evict key from cache"""
        ...
```

```bash
$ stubcheck handlers.py
file:line:col → kind|detail
handlers.py:3:5 → bare|warm() — body is only pass/ellipsis; silent no-op returning None — unimplemented logic shipped
handlers.py:6:5 → docstring|evict() — body is only a docstring + pass/ellipsis; documented as pending but returns None at runtime

2 no-op stub function(s) return None.
```

Fill in the implementations:

```python
def warm(self, keys):
    for k in keys:
        self._cache[k] = self.fetch(k)

def evict(self, key):
    del self._cache[key]
```

## Severity Model

| Kind | Example | Severity | Rationale |
|---|---|---|---|
| `bare` | `def f(): pass` (no docstring) | high | Completely opaque — no logic and no documentation of intent |
| `docstring` | `def f(): """todo"""` + `pass` | medium | Intent is documented, but the body still returns None silently |

**Default** `--min-severity medium` reports both. Use `--min-severity high`
in CI to gate only on bare stubs (no docstring = highest risk). Use
`--min-severity low` to report everything (same as medium for now).

## What Is NOT Flagged

- Functions decorated with `@abstractmethod` or `@abc.abstractmethod` —
  these are the documented *contract* that subclasses must implement.
- `.pyi` type-stub files — an entire file language for declaring signatures.
- Functions whose body contains any real statement (`return`, `raise`,
  assignments, calls, nested defs) — even a trailing redundant `pass` after
  real code is not a silent stub.
- Test files by default — pass-through stubs are sometimes used as test
  doubles. Pass `--include-test` to audit them anyway.

## CI Mode (Exit Codes)

- `0` — no silent no-op stubs found (CI green)
- `1` — at least one finding with `--check` (CI red)
- `2` — invalid arguments

```yaml
- name: Fail on stub functions
  run: pipx run stubcheck src/ --check
```

## Noise & Scope

Directories skipped by default (`.git`, `__pycache__`, `venv`, `node_modules`,
`.ruff_cache`, `.pytest_cache`, etc.) — pass `--hidden` to include them.

## Why Not Just Use ruff or flake8?

Neither ruff nor flake8 (nor pylint by default) flag a **function body that
is only `pass`/`...`** as a distinct, low-noise finding. `pass` is valid
Python, so it slips through every mainstream linter's default rules. pylint
has a scattered set of *empty-body* checks, but they require heavy
configuration and produce noise on abstract methods and type stubs.

`stubcheck` is a single-purpose, zero-dependency AST scan: it understands
that `@abstractmethod` stubs and `.pyi` files are legitimate, and flags only
the **silent** no-ops that return `None` in production logic.

## Why Zero Dependencies?

Every check is pure static analysis on the standard `ast` module — no parser
dependencies, no vendored tooling. Pipe `--json` into `jq`, and keep function
bodies honest on codebases of any size.

Pair it with `assertcheck` (validation that vanishes under `-O`), `broadcheck`
(silent error-swallowing), `mutablecheck` (mutable defaults), `shadowcheck`
(builtin shadowing), `globalcheck` (module-global mutation), `cccheck` (God
functions), `dupcheck` (copy-paste), and `todolint` (markers) for the complete
agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
