# labcheck — Validate Text Annotation Datasets in One Command

Text annotation datasets often contain silent quality issues — duplicate texts with conflicting labels, empty fields, missing columns, or severe class imbalance. These bugs surface hours into training as unexplained model failures. `labcheck` catches them before you train.

## Quick Start

```bash
pip install -e /root/labcheck

# Check a CSV annotation file
labcheck annotations.csv

# Check a JSONL file with custom column names
labcheck data.jsonl --text-col sentence --label-col sentiment

# Set a stricter class imbalance threshold
labcheck train.csv --imbalance-threshold 10
```

## Why This Exists

Before `labcheck`, validating text annotation datasets meant:
- Manually scrolling through 10K+ rows in a spreadsheet
- Writing one-off Python scripts to find duplicates
- Discovering label conflicts only after training produced garbage
- Missing empty fields until they crashed your pipeline

`labcheck` does one thing well: it scans your dataset and reports every quality issue.

## Real Example

```bash
$ labcheck annotations.csv
──────────────────────────────────────────────────
  labcheck — Annotation Quality Report
──────────────────────────────────────────────────
  File: annotations.csv

  ❌ Issues Found:

    • Row 3: empty or missing 'text'
    • Row 7: label conflict — 'customer service was great...' has labels ['positive'] and ['negative']

  📊 Statistics:

    Label distribution: {'positive': 892, 'negative': 98, 'neutral': 10}
      ⚠ Class 'neutral' has only 1.0% (10/1000) — below 5% threshold

  2 issue(s) found.
```

## What labcheck Checks

1. **Empty text fields** — rows with missing or blank text
2. **Missing labels** — rows without annotation labels
3. **Label conflicts** — duplicate texts with different labels
4. **Column existence** — schema mismatches when column names differ
5. **Label distribution** — counts for every class
6. **Class imbalance** — warnings for classes below configurable threshold

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
