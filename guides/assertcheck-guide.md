# assertcheck — Stop Shipping Validation That Vanishes Under -O

Every Python application ships with `python -O`. Gunicorn, uWSGI, mod_wsgi,
and every production WSGI server sets optimized mode by default. In that
mode, **every `assert` statement is silently removed at compile time** — not
just disabled, but completely excised from the bytecode.

AI-generated code discovers `assert` as the quickest way to add validation:

```python
def withdraw(user_id, amount):
    assert user_id is not None, "user_id required"   # ← gone under -O
    assert amount > 0                                 # ← also gone
    if user_id < 1:
        raise ValueError("bad user_id")
    ...
```

In tests this passes. In production the checks evaporate, `None` user IDs and
negative amounts sail straight through, and the failure surfaces much later as
a confusing `TypeError` or worse, a silent data corruption.

`assertcheck` walks the AST, finds every `assert` in your source tree, and
classifies it so you can replace it with real `if/raise` logic before it
ships. Test files are skipped automatically — asserts belong in tests.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/assertcheck.git
cd assertcheck
pip install -e .

# Scan everything under src/
assertcheck src/

# CI gate — fail the build on assert-based validation
assertcheck . --check

# Only the most dangerous: bare asserts with no message
assertcheck src/ --min-severity high

# Machine-readable output
assertcheck . --json

# Include test files too (for auditing purposes)
assertcheck . --include-test
```

## Real Example

```python
# auth.py
def authenticate(token):
    assert token is not None          # ← bare: vanishes silently in prod
    assert len(token) > 16, "token too short"  # ← msg: stripped anyway
    if not token.startswith("Bearer "):
        raise InvalidTokenError("bad prefix")
    return token
```

```bash
$ assertcheck auth.py
file:line:col → kind|detail
auth.py:2:5 → bare|assert <condition> — no message, vanishes silently under -O
auth.py:3:5 → msg|assert ..., "token too short"

2 assert statement(s) that vanish under -O.
```

Replace both with:

```python
if token is None:
    raise ValueError("token is required")
if len(token) <= 16:
    raise ValueError("token too short")
```

## Severity Model

| Kind | Example | Severity | Rationale |
|---|---|---|---|
| `bare` | `assert x > 0` (no message) | high | Completely silent when stripped — failure is opaque |
| `msg` | `assert x > 0, "must be positive"` | medium | Message documents intent, but still stripped in prod |

**Default** `--min-severity medium` reports both. Use `--min-severity high`
for CI to gate only on bare asserts (no message = highest risk). Use
`--min-severity low` to report everything (same as medium for now).

## Test File Exclusion

Files matching any of these patterns are **skipped by default**:

- `test_*.py`, `*_test.py`, `conftest.py`
- Any `.py` file inside a `tests/` or `test/` directory

Pass `--include-test` to audit asserts in test code too.

## CI Mode (Exit Codes)

- `0` — no assert-based validation found (CI green)
- `1` — at least one finding with `--check` (CI red)
- `2` — invalid arguments

```yaml
- name: Fail on assert-based validation
  run: pipx run assertcheck src/ --check
```

## Why Not Just Use Bandit?

Bandit's `B101` rule flags **all** `assert` usage at low severity. It can't
distinguish test files from production source, so teams routinely disable it.
`assertcheck` makes the distinction explicit, classes by risk, and keeps zero
dependencies so `pipx run` it in any CI environment.

## Why Zero Dependencies?

Every check is pure static analysis on the standard `ast` module — no parser
dependencies, no vendored tooling. Pipe `--json` into `jq`, and keep functions
honest on codebases of any size.

Pair it with `broadcheck` (silent error-swallowing), `mutablecheck` (mutable
defaults), `shadowcheck` (builtin shadowing), `globalcheck` (module-global
mutation), `cccheck` (God functions), `dupcheck` (copy-paste), and `todolint`
(markers) for the complete agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
