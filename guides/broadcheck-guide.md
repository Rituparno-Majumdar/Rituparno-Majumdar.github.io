# broadcheck — Stop Shipping Handlers That Eat Your Errors

The worst production failure is the one you never see. Python gives you three
ways to build one: a **bare** `except:`, a **broad** `except Exception:`, and
an **empty** handler body that does nothing but `pass` (or a docstring, or
`...`). Compose them — `except Exception: pass` — and every error, plus the
information needed to fix it, is gone.

LLMs generate this smell constantly. Given a defensive instruction, models
wrap code in `try/except Exception` and leave the handler blank "to keep it
safe." It compiles, tests pass, and then somewhere in production a pipeline
silently returns `None` and the root cause is invisible. `broadcheck` makes
the whole class visible again.

`broadcheck` walks the AST and reports handlers that are bare, broad, or
empty — filtering out the legitimate `except ValueError: return fallback()`
cases. Specific exceptions with real handling are never flagged. Zero
dependencies.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/broadcheck.git
cd broadcheck
pip install -e .

# Scan everything under src/
broadcheck src/

# CI gate — fail the build on risky handlers
broadcheck . --check

# Tolerate "except Exception" but keep flagging bare/empty
broadcheck src/ --allow-broad

# Lift the floor: also report broad-only handlers
broadcheck src/ --min-severity medium

# Machine-readable output
broadcheck . --json
```

## Real Example

```python
def load_config(path):
    try:
        with open(path) as f:
            return json.load(f)
    except Exception:          # broad + empty — error disappears
        pass
```

```bash
$ broadcheck config_loader.py
FILE                 LINE  COL  KIND        EXCEPT
config_loader.py     5     5    broad+empty except Exception
```

## Severity Model

| Kind | Example | Severity | Rationale |
|------|---------|----------|-----------|
| `bare` | `except:` | high | Catches `KeyboardInterrupt` / `SystemExit` too |
| `empty` | `except X: pass` | high | Swallows the error with zero handling |
| `broad` | `except Exception:` | medium | Masks genuine errors, hides root cause |

`--allow-broad` skips **pure** broad handlers but keeps `bare` and any
handler paired with an empty body — `except Exception: pass` is the worst of
both, so it stays flagged.

## CI Mode (Exit Codes)

- `0` — no risky handlers (CI green)
- `1` — at least one finding with `--check` (CI red)
- `2` — invalid arguments

```yaml
- name: Fail on silent error-swallowing
  run: pipx run broadcheck src/ --check
```

## Why Zero Dependencies?

Every check is pure static analysis on the standard `ast` module — no parser
dependencies, no vendored tooling. `pipx run` it in CI, pipe `--json` into
`jq`, and keep PRs honest on codebases of any size.

Pair it with `mutablecheck` (aliasing defaults), `shadowcheck` (builtin
shadowing), `cccheck` (God functions), `dupcheck` (copy-paste), and `todolint`
(markers) for the complete agent-era quality net on pure-Python codebases.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*