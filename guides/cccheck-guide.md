# cccheck — Stop Shipping Untestable God Functions

The code smell that silently rots projects is not usually a missing dependency or
a bad name — it's **one function doing forty things** behind thirty nested
`if`s. Cyclomatic complexity is the reliable proxy for "how hard is this to
test": every branch is a path that can carry a bug, and LLMs are fluent
generators of exactly this — long, branch-bombed functions that pass tests yet
fall apart at the edges.

`cccheck` scans a Python codebase and **ranks every function by cyclomatic
complexity + line count**, flagging which ones are riskiest to touch and most
likely to break.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/cccheck.git
cd cccheck
pip install -e .

# Scan the whole codebase, show every function
cccheck

# Stricter gate — any function above CC 8 fails CI
cccheck src/ --max 8 --check

# Machine-readable output
cccheck src/ --json
```

## Real Example

```bash
$ cccheck src/ --max 10
FILE                          FUNCTION                    LINE  CC   LOC
src/service/core.py           build_report_payload         124   18   96   <-- exceeds --max
src/service/core.py           retry_with_backoff            33   12   24
src/app/cli.py                main                          15    8   40

46 function(s) scanned; 2 above CC 10
```

`build_report_payload` at CC 18 is the prime suspect for the next production
incident — it is now scheduled for a refactor, and the gate will stop any new
function from joining it.

## CI Mode (Exit Codes)

- `0` — everything is at or below the complexity budget (CI green)
- `1` — at least one function exceeds `--max` with `--check` (CI red)
- `2` — invalid arguments (bad `--max`, missing path)

```yaml
- name: Fail on complex functions
  run: pipx run cccheck src/ --check
```

## What Counts as a Decision Point

McCabe's rule: complexity = `1` + every decision point. `cccheck` counts:

| Construct | Contribution |
|-----------|--------------|
| `if` / `elif` / ternary `if x else y` | +1 each |
| `and` / `or` inside conditions | +1 per operator |
| `for` / `while` / `async for` | +1 each |
| `try`/`except` handlers | +1 each |
| list / dict / set comprehensions | +1 per generator + filter `if`s |
| `match` cases | +1 each |

Nested functions and classes are measured independently — an inner helper never
inflates its outer function's score, so you see real per-responsibility
complexity.

## Why Zero Dependencies?

Complexity measurement is pure static analysis — there is no reason to drag in a
framework to read an AST. `cccheck` is a single-file Python CLI on the standard
library: `pipx run` it in any CI, pipe `--json` into `jq`, and keep PRs honest.

Pair it with `todolint` (are your markers clean?) and `dupcheck` (is anything
duplicated?) for a complete agent-era quality net on a pure-Python codebase.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*