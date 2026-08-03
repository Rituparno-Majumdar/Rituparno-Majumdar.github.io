# wikicheck — Validate Obsidian Wikilinks in One Command

Obsidian wikilinks break silently. You rename a note or restructure a folder — and every `[[old-name]]` reference in your vault quietly points nowhere. `linkrot` checks Markdown links `[text](url)`, but **nobody validates the `[[wikilinks]]` that hold your knowledge graph together** — until the graph view turns into disconnected islands.

`wikicheck` catches broken targets, missing heading anchors, malformed links, and orphan notes in a single zero-dependency command.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/wikicheck.git
cd wikicheck
pip install -e .

# Scan the vault
wikicheck /path/to/vault

# CI-ready JSON output
wikicheck /path/to/vault --json

# Skip orphan reporting
wikicheck /path/to/vault --no-orphans
```

## What It Checks

| Check | Example | Detects |
|-------|---------|---------|
| Broken targets | `[[missing-note]]` | No note in the vault matches |
| Heading anchors | `[[note#Old Section]]` | Heading no longer exists in the target |
| Case-insensitive resolution | `[[MATH NOTES]]` → `math notes.md` | Matches Obsidian's filename resolution |
| Folder paths | `[[sub/Nested Note]]` | Resolution through the folder tree |
| Aliases | `[[note\|Display Alias]]` | Parsed and validated correctly |
| Code-block awareness | Links inside ``` fences | Ignored, not flagged |
| Orphan notes | Notes never linked | `index.md` / `README` / `MOC` / `home` excluded |

## Real Example

```bash
$ wikicheck ~/Notes
───────────────────────────────────────────────────
  wikicheck — Obsidian Wikilink Report
───────────────────────────────────────────────────

  ❌ 2 issue(s) found:

    • index.md:7 [broken] no note matches [[broken-target]]
    • philosophy.md:6 [anchor] heading '#WrongHeading' not found in math notes.md

  🗂  1 orphan note(s) (never linked):

    • orphan.md

  Checked 12 wikilink(s), 2 issue(s), 1 orphan(s).
```

## Exit Codes (CI-friendly)

- `0` — All wikilinks valid, no orphans
- `1` — One or more issues found
- `2` — Path does not exist / usage error

## Why Zero Dependencies?

Existing vault checkers drag in Node.js runtimes or heavy stacks. `wikicheck` is a single-file Python CLI on the standard library — install anywhere in seconds, run in a GitHub Action, pipe into `jq`.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
