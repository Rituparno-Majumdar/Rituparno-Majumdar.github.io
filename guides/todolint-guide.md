# todolint — Catch Unfinished Work Before It Ships

`TODO`, `FIXME`, `HACK`, `BUG` — the four horsemen of unfinished code. They live
in comments, get written by humans under deadline and by AI agents mid-task, and
nobody ever reads them again. The context evaporates, the debt stays.

`todolint` surfaces every marker in your codebase, groups it by file, and gives
you a CI exit code so a `FIXME` can't silently land on `main`.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/todolint.git
cd todolint
pip install -e .

# Scan the whole repo
todolint

# Scan just the source tree, blockers only
todolint src/ --min-severity high

# Fail CI on any FIXME/HACK/BUG
todolint src/ --check
```

## Real Example

```bash
$ todolint src/
============================================================
  todolint — TODO & FIXME Scanner
============================================================

  src/app.py
         1: [FIXME] (BLOCKER) crashes on empty input
         4: [ TODO] (todo) handle edge case

  src/notes.md
         1: [  BUG] (BLOCKER) memory leak

============================================================
  Scanned 1 path(s) — 3 finding(s): 2 blocker(s), 0 warning(s), 1 todo(s), 0 note(s).
```

## CI Mode (Exit Codes)

- `0` — no blockers found (CI green)
- `1` — at least one **blocker** (`FIXME`, `HACK`, `BUG`) found (CI red)

```yaml
- name: Block FIXMEs
  run: pipx run todolint src/ --check
```

## Markers & Severity

| Marker | Severity |
|-------|----------|
| `FIXME`, `BUG`, `HACK` | 🔴 high (fails `--check`) |
| `XXX` | 🟠 medium |
| `TODO` | 🟡 low |
| `NOTE` | 🔵 info |

Detection is case-insensitive, tolerates `FIXME(high):` / `TODO(user):` suffixes,
and skips hidden directories, binary files, and identifiers like `todo_list` by
default (`--hidden` to include dotfiles).

## Why Zero Dependencies?

Marker scanning is a pre-build hygiene check — you shouldn't need a linter
framework or a tree-sitter binding to find a `FIXME`. `todolint` is a single-file
Python CLI on the standard library: `pipx run` it in any CI, pipe `--json` into
`jq`, and wire it into pre-commit in under a minute.

Pipe `todolint` into your pre-commit hook or your agent's pre-push check — a
non-zero exit on a blocker is the cheapest way to keep unfinished work out of your
merge queue.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
