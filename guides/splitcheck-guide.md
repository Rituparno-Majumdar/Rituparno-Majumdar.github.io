# splitcheck — Detect Data Leakage Across ML Dataset Splits in One Command

Data leakage is the quiet killer of ML evaluation. When a few samples from your training set also appear in your validation or test set, your reported accuracy is optimistic — and it's almost always accidental (an oversized download, a reused CSV, a copy-paste). Tools that validate each file in isolation never catch it, because the problem only exists *between* files.

`splitcheck` does one thing: scan your split files and report **every row that appears in more than one split**, so you fix the leak before you trust the numbers.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/splitcheck.git
cd splitcheck
pip install -e .

# CSV splits
splitcheck train.csv val.csv test.csv

# JSONL splits (or mixed formats)
splitcheck train.jsonl val.jsonl test.jsonl
splitcheck train.csv val.jsonl
```

## Real Example

```bash
$ splitcheck train.csv test.csv
============================================================
  splitcheck — Data Leakage Report
============================================================
  train.csv            1000 rows
  test.csv              200 rows
------------------------------------------------------------
  LEAKAGE FOUND: 12 shared row(s)
    test.csv+train.csv: 12 row(s)
------------------------------------------------------------
```

## Exit Codes (CI-friendly)

- `0` — Splits are disjoint, no leakage
- `1` — Leakage detected (fail your training/CI pipeline)
- `2` — Input error (malformed file, unreadable path)

## Why It Works

| Feature | Detail |
|---------|--------|
| **Zero dependencies** | Pure Python standard library — install anywhere in seconds |
| **CSV + JSONL** | Headers skipped automatically; JSON objects compared by flattened key/value content |
| **Robust fingerprints** | Whitespace-insensitive, stable ordering, SHA-256 hashed row keys |
| **Multi-split** | Works with 2, 3, or N splits; reports shared-row counts per split pair |

Pipe `splitcheck` into your training CI gate — a non-zero exit on shared rows is the cheapest way to guarantee your reported metrics are honest.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
