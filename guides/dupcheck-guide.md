# dupcheck — Kill Copy-Paste Before It Kills Your Codebase

Copy-paste is the most reliable bug factory in software: a fix lands in one
copy, the other keeps the bug. And AI code generation amplifies it — models
happily emit near-identical function bodies across files, and no reviewer has
time to diff every file against every other file by hand.

`dupcheck` scans a directory for **identical code blocks** appearing in **two or
more places** — across files or within a single file — and reports every
occurrence location, ready for a refactor.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/dupcheck.git
cd dupcheck
pip install -e .

# Scan the whole repo (blocks of 5+ identical lines)
dupcheck

# Longer blocks only, specific paths
dupcheck src/ tests/ --min-lines 8

# Machine-readable output
dupcheck --json
```

## Real Example

```bash
$ dupcheck src/
============================================================
  dupcheck — Duplicated Code Block Scanner

  Block 1: 6 lines — 2 occurrence(s)
    src/payments.py:12
    src/ledger.py:31
      | def reconcile(items):
      |     total = 0
      |     for item in items:
      |         total += item.value
      |     return total

  Scanned 24 file(s) — 1 duplicate block(s), 6 duplicated line(s).
```

Two modules carried the same `reconcile` body — a refactor that should happen
in one place, not two.

## CI Mode (Exit Codes)

- `0` — no duplicates (CI green)
- `1` — at least one duplicated block found with `--check` (CI red)
- `2` — invalid arguments (e.g. `--min-lines < 2`)

```yaml
- name: Fail on duplicated code
  run: pipx run dupcheck src/ --check
```

## How Detection Works

- **Normalized lines** — trailing whitespace stripped, comment-only lines
  ignored, so a refactored comment doesn't break a true code match.
- **Sub-block suppression** — a 6-line duplicate that lives inside a 10-line
  duplicate is reported once, as the larger block, not twice.
- **Noise dirs skipped** — `__pycache__`, `.git`, `node_modules`, `target`,
  `vendor` are excluded by default (`--hidden` overrides dotfile skipping).
- **Binary files skipped** automatically.

## Why Zero Dependencies?

Duplicate detection is a pre-refactor hygiene check — you shouldn't pull in a
tree-sitter binding or a similarity framework to find copy-paste. `dupcheck` is
a single-file Python CLI on the standard library: `pipx run` it in any CI, pipe
`--json` into `jq`, and keep your codebase honest.

Wire it into pre-commit or your agent's pre-push check — a non-zero exit on a
duplicate is the cheapest refactor prompt there is.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*