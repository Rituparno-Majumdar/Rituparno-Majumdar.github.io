# docstruct — Validate Markdown Structure in One Command

Inconsistent heading structure is the silent rot of documentation. A missing H1 or a skipped heading level breaks TOC generation, confuses LLM chunkers for RAG, and erodes reader trust. `docstruct` catches these issues instantly.

## Quick Start

```bash
pip install -e /root/docstruct

# Check any docs directory
docstruct ./docs --recursive --verbose
```

## Why This Exists

Before `docstruct`, validating markdown structure meant:
- Manually scrolling through files hoping to spot issues
- Using heavyweight linters with hundreds of rules you don't need
- Discovering structural problems only after deploying your docs site

`docstruct` does one thing well: it validates heading hierarchy and produces a clear quality score (0–100) for every file.

## Real Example

```bash
# Before: Manual inspection — impractical for 100+ files
grep -rn "^#" docs/ | awk '{print NR, $0}'  # misses hierarchy entirely

# After:
docstruct ./docs --recursive --verbose
# ──────────────────────────────────────────────────
#   docstruct — Markdown Structure Report
# ──────────────────────────────────────────────────
#   Files scanned     : 12
#   Files with issues : 3
#   Total issues      : 4
#   Average quality   : 86.7/100
#
#   advanced.md (75.0/100)
#     ⚠ Skipped heading level: H1 → H3 ('Usage' at line 8)
#
#   quickstart.md (80.0/100)
#     ⚠ First heading is H2, not H1 (line 1)
#
#   empty.md (0.0/100)
#     ⚠ No headings found
#
#   ⚠ 4 issue(s) found across 12 file(s).
```

The most common issues `docstruct` catches:

1. **Skipped levels** — `H1 → H3` without an H2. This breaks your document outline.
2. **Missing H1** — Files starting with `## Section` instead of `# Title`. These pages have no top-level context.
3. **Empty files** — Files with no headings at all. Dead weight in your docs.

Fix these issues, re-run `docstruct`, and deploy with confidence.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
