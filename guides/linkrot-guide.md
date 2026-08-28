# linkrot — Detect Broken Internal Markdown Links in One Command

Internal Markdown links break silently: files get renamed, sections get removed, and nobody notices until a reader clicks a dead link. `linkrot` is a zero-dependency Python CLI that scans your entire Markdown tree and reports every broken reference in seconds.

## Quick Start

```bash
pip install linkrot

# Scan current directory for broken internal links
linkrot

# Scan a specific docs folder with verbose output
linkrot ./docs --verbose

# JSON output — pipe into CI pipelines or jq
linkrot ./docs --json
```

## What linkrot Checks

1. **Missing file targets** — links pointing to `.md` files that don't exist
2. **Missing section anchors** — `[text](file.md#section)` where `#section` doesn't match any heading
3. **Out-of-tree references** — links pointing outside the scanned directory
4. **Cross-file integrity** — validates every internal `[text](path)` across the entire file tree

## Real Example

```bash
$ linkrot ./docs
───────────────────────────────────────────────────────
  linkrot — Broken Internal Link Report
───────────────────────────────────────────────────────

  ❌ 2 broken link(s) found:

    • docs/guide.md:42 → setup.md
      └─ file not found
    • docs/README.md:15 → usage.md#getting-started
      └─ anchor #getting-started not found in usage.md

  Checked 127 link(s), 2 broken.
```

## Exit Codes

- `0` — All internal links are valid (great for CI gating)
- `1` — One or more broken links found

## Why This Exists

Before `linkrot`, checking Markdown link integrity meant:
- Installing heavy Node.js toolchains for a simple file check
- Writing one-off grep scripts that miss edge cases
- Discovering dead links only when users report them

`linkrot` does one thing well: it scans your docs and tells you exactly what's broken.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*