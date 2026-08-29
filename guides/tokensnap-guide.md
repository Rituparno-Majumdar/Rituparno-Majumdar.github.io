# TokenSnap — Estimate LLM API Costs from Your Terminal

Stop opening browser tabs to check LLM pricing. TokenSnap reads your prompt from a file or stdin, counts tokens, and estimates costs across 12+ providers — all from your terminal.

## Quick Start

```bash
pip install tokensnap
cat my-prompt.md | tokensnap
```

## Why This Exists

Every LLM API call costs real money, but figuring out *how much* means switching between pricing pages, online tokenizers, and mental math. TokenSnap collapses that into one command — pipe your prompt, get a sorted cost table.

## Real Example

```bash
# Before: Open 3 browser tabs, count tokens, multiply prices, compare manually
# After:
$ cat prompt.md | tokensnap --json | jq '.providers | to_entries | sort_by(.value.total_cost) | .[0:3]'
```

Shows the 3 cheapest providers for your prompt — sorted, priced, done.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
