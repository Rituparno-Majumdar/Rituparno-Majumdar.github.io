# globalcheck — Stop Shipping Functions That Mutate the Module

The quietest bug is the one that looks like a design decision: a helper
function that reaches up and mutates a module-level object instead of
returning a value.

```python
results = []            # module-level state

def collect(item):       # silently mutates the global
    results.append(item)
```

This is one of the most common patterns AI-generated code emits — models
hoist a "results bucket" to module scope so every helper can share it, and
"keep the function signatures small." It compiles. Tests pass. Then:

- state leaks between calls and test runs,
- output ordering depends on call order, not data,
- the function is untestable in isolation and uncomposable,
- adding concurrency turns silent shared state into a race.

`globalcheck` makes the whole class visible again. It walks the AST, tracks
which names are bound at module scope, and reports every function that
mutates one of them — while ignoring local lists, function parameters,
imports used as libraries, and module-level setup code.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/globalcheck.git
cd globalcheck
pip install -e .

# Scan everything under src/
globalcheck src/

# CI gate — fail the build on hidden shared state
globalcheck . --check

# Skip functions that legitimately populate globals (registries, caches)
globalcheck src/ --allow populate,register

# Only the explicit `global` statements
globalcheck src/ --min-severity high

# Machine-readable output
globalcheck . --json
```

## Real Example

```python
cache = {}

def get_or_compute(key, fn):   # classic AI smell: memoize by mutating a global
    if key not in cache:
        cache[key] = fn(key)
    return cache[key]
```

```bash
$ globalcheck memo.py
FILE                 LINE  COL  KIND    DETAIL
memo.py              4     5    index   cache[...] mutated
```

## Severity Model

| Kind | Example | Severity | Rationale |
|------|---------|----------|-----------|
| `global` | `global counter` inside a function | high | Explicit global mutation — scope-jumping state, always intentional |
| `mutate` | `results.append(x)`, `stats.update(...)`, `seen.add(x)` | medium | In-place method call on a module-global object |
| `index` | `cache[key] = v`, `del cache[k]` | medium | Subscript write to a module-global mapping |
| `attr` | `server.port = 8080` | low | Attribute patch on a module-global object |

Never flagged: local lists, parameters that shadow a global, `import os` +
`os.getcwd()` (imports are modules, not state), `os.environ["X"] = ...`
(indirection through an attribute), and module-level mutation that executes
during import.

## CI Mode (Exit Codes)

- `0` — no module-state mutations (CI green)
- `1` — at least one finding with `--check` (CI red)
- `2` — invalid arguments

```yaml
- name: Fail on hidden shared state
  run: pipx run globalcheck src/ --check
```

## Why Zero Dependencies?

Every check is pure static analysis on the standard `ast` module — no parser
dependencies, no vendored tooling. `pipx run` it in CI, pipe `--json` into
`jq`, and keep functions honest on codebases of any size.

Pair it with `mutablecheck` (default-argument aliasing), `broadcheck` (silent
error-swallowing), `shadowcheck` (builtin shadowing), `cccheck` (God
functions), `dupcheck` (copy-paste), and `todolint` (markers) for the complete
agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*