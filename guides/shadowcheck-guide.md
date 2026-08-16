# shadowcheck — Stop Shipping Builtin-Shadowed Code

The nastiest Python bug is the one that **compiles**. Shadowing happens when a
program rebinds a name the interpreter already owns: `list = [1, 2]` in one module,
or a scraped script that names a variable `id`, or a function that takes a parameter
called `sum`. The program runs — until it `TypeError`s three files away from the
binding, and a human spends an hour wondering why `list` stopped being a type.

LLMs are fluent generators of this exact smell: toy and prototype code reuses
`list`, `id`, `input`, `file`, and `type` as natural variable names, and models
trained on such examples reproduce the habit. `shadowcheck` makes this entire
failure class visible before the code ships.

`shadowcheck` scans every **binding** in your Python files — assignments, function
and class definitions, parameters, `for`/`with` targets, catch clauses, imports,
comprehensions, walrus expressions, and `match` patterns — and reports any name that
collides with a Python builtin. Zero dependencies.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/shadowcheck.git
cd shadowcheck
pip install -e .

# Scan everything under src/
shadowcheck src/

# CI gate — fail if any shadowing exists
shadowcheck . --check

# Tolerate deliberate shadowing of specific names
shadowcheck src/ --allow id,type

# Machine-readable output
shadowcheck . --json
```

## Real Example

```python
list = [1, 2, 3]        # shadows builtin list

def id():               # shadows builtin id
    return 1

def process(sum):       # shadows builtin sum inside the function
    return sum(list)
```

```bash
$ shadowcheck example.py
FILE          LINE  COL  NAME    KIND
example.py    1     1    list    assign
example.py    3     5    id      def
example.py    6     12   sum     param
```

Note `open` in `with open(f) as file:` is **not** flagged — that's a use (load) of
the builtin, not a rebinding. `shadowcheck` only reports where a name gets bound and
stops being the builtin from that point on.

## CI Mode (Exit Codes)

- `0` — no shadowed builtins (CI green)
- `1` — at least one shadow found (CI red)
- `2` — invalid arguments (bad `--allow` name, missing path)

```yaml
- name: Fail on shadowed builtins
  run: pipx run shadowcheck src/ --check
```

## What Gets Flagged

| Binding kind | Example |
|--------------|---------|
| assignment | `list = [...]`, destructuring `id, type = t` |
| definition | `def id():`, `class type:`, `async def open():` |
| parameter | `def handler(sum):`, lambda args |
| loop / with | `for dict in rows:`, `with open(f) as type:` |
| catch clause | `except KeyError as filter:` |
| import alias | `import json as filter`, `from os import path as open` |
| comprehension | `[len for len in range(3)]` |
| walrus | `if (id := 5) > 0:` |
| match pattern | `case int() as sum:` |

## Why Zero Dependencies?

Shadowing detection is pure static analysis on the AST — nothing warrants a
framework. `shadowcheck` is a single-file Python CLI on the standard library:
`pipx run` it in any CI, pipe `--json` into `jq`, and keep PRs honest.

Pair it with `cccheck` (God functions), `dupcheck` (copy-paste), and `todolint`
(unfinished markers) for a complete agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*