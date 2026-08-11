# fencecheck — Catch Unclosed Code Fences Before They Swallow Your Docs

An unclosed Markdown code fence is the quietest doc killer there is. Open a ```` ``` ```` block, forget to close it, and every renderer downstream treats the rest of your file as code:

- **Obsidian** renders everything after it as a single code block
- **Pandoc** fails or mangles the entire document
- **Static site generators** emit broken HTML
- **AI doc-parsers** read the rest of the file as one giant code block — a whole page of context silently lost

Existing tools validate links, headings, frontmatter, and tables — none check that your code fences actually close. `fencecheck` does that one thing.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/fencecheck.git
cd fencecheck
pip install -e .

# Scan a directory recursively
fencecheck docs/

# Scan specific files
fencecheck README.md CHANGELOG.md

# CI-ready JSON output
fencecheck docs/ --json
```

## Real Example

```bash
$ fencecheck docs/guide.md
───────────────────────────────────────────────────────
  fencecheck — Unclosed Code Fence Report
───────────────────────────────────────────────────────

  ❌ 1 fence issue(s) found:

    • docs/guide.md:12
      └─ ``` — unclosed code fence

  Checked 37 fence(s) across 5 file(s), 1 issue(s).
```

## Exit Codes (CI-friendly)

- `0` — All code fences are balanced (CI green)
- `1` — One or more fence issues found (CI red)

## What It Checks

| Check | Description |
|-------|-------------|
| **Unclosed backtick fences** | ```` ``` ```` opened with no matching close |
| **Unclosed tilde fences** | `~~~` opened with no matching close |
| **Swapped closers** | Opened with ```` ``` ```` but closed with `~~~` (CommonMark: a fence only closes a same-character fence) |
| **Fence length matching** | A close needs *at least* as many characters as the open |

CommonMark-aware: a `~~~` line inside a ```` ``` ```` block is treated as literal content, not a mis-close.

## Why Zero Dependencies?

Markdown lives everywhere — docs repos, CI runners, agent instruction files — and you shouldn't need to install a framework to lint it. `fencecheck` is a single-file Python CLI on the standard library: install anywhere, pipe into `jq`, fail a GitHub Action with a clean exit code.

Pipe `fencecheck` into your docs CI or your LLM doc-ingestion pipeline — a non-zero exit on an unclosed fence is the cheapest way to guarantee your documents stay readable to humans *and* agents.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
