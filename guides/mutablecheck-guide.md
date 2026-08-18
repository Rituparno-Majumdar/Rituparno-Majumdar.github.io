# mutablecheck — Stop Shipping the Aliasing Footgun

The Python default-argument footgun is a bug that **never error**. `def f(items=[])`
evaluates the list **once**, at import time. Every call that appends to `items`
mutates the *same* shared list, so state leaks between calls — the output of one
call silently contaminates the next. No exception, no warning, just wrong data that
appears "random".

LLMs are fluent generators of this exact smell: the `=[]`, `={}`, and `=set()`
defaults show up constantly in scaffolded and generated code, and the model rarely
knows its own output is a runtime bug. `mutablecheck` makes this whole failure class
visible before the code ships.

`mutablecheck` scans every function and method signature in your Python files —
positional and keyword-only defaults — and flags any default that is a mutable
list/dict/set literal or a `list()`/`dict()`/`set()`/`bytearray()`/`defaultdict()`
constructor call. Zero dependencies.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/mutablecheck.git
cd mutablecheck
pip install -e .

# Scan everything under src/
mutablecheck src/

# CI gate — fail if any mutable default exists
mutablecheck . --check

# Tolerate deliberate mutable defaults in specific functions
mutablecheck src/ --allow cache_store

# Machine-readable output
mutablecheck . --json
```

## Real Example

```python
def cache_store(items=[]):      # mutable default — shared across calls
    items.append(last_seen)     # leaks state between calls
    return items

def index(tags={"a", "b"}):     # set literal — also shared
    return tags | incoming
```

```bash
$ mutablecheck example.py
file:line:col → func: arg (kind)
example.py:1:23 → cache_store: items (list literal)
example.py:4:16 → index: tags (set literal)

2 mutable default argument(s) found.
```

The immutable defaults — `None`, `0`, `""`, `(1, 2)` — are fine and never flagged.

## CI Mode (Exit Codes)

- `0` — no mutable defaults (CI green)
- `1` — at least one mutable default found (CI red)
- `2` — invalid arguments

```yaml
- name: Fail on mutable default arguments
  run: pipx run mutablecheck src/ --check
```

## What Gets Flagged

| Default | Kind | Why it's dangerous |
|---|---|---|
| `[]` | list literal | shared list, state leaks across calls |
| `{}` | dict literal | shared dict, keys accumulate |
| `set()` | set literal | shared set, membership confusion |
| `list()` | list() call | same alias problem via constructor |
| `bytearray()` | bytearray call | shared mutable buffer |
| `defaultdict(list)` | defaultdict call | shared default factory state |

## Why Zero Dependencies?

Mutable-default detection is pure static analysis on the AST — nothing warrants a
framework. `mutablecheck` is a single-file Python CLI on the standard library:
`pipx run` it in any CI, pipe `--json` into `jq`, and keep PRs honest.

Pair it with `shadowcheck` (builtin shadowing), `cccheck` (God functions),
`dupcheck` (copy-paste), and `todolint` (unfinished markers) for a complete
agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*