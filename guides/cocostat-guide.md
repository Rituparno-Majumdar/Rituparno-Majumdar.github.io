# cocostat — Quick COCO Dataset Statistics

Inspect COCO annotation files from your terminal — no Jupyter notebook required.

## Quick Start

```bash
pip install cocostat
cocostat annotations.json
```

## Why This Exists

Every time you pick up a new COCO dataset, you need to answer basic questions: How many images? How balanced are the classes? What's the average bbox size? Instead of writing 15 lines of Python each time, `cocostat` does it in one command.

## Real Example

```bash
# Before: Load COCO JSON, count images, build Counter for classes,
#          compute bbox stats, format output — every single time
# After:
cocostat instances_train2017.json
```

```
  ┌─ COCO Dataset Statistics ─────────────────────────────┐
  │ Images:                   118,287                     │
  │ Annotations:              860,001                     │
  │ Categories:               80                          │
  │ Avg annot/image:           7.27                       │
  ├───────────────────────────────────────────────────────┤
  │  Class distribution (top 15)                          │
  │    person           262,348 (30.5%) ████████████████   │
  │    car              138,401 (16.1%) ████████           │
  │    ...                                                │
  └───────────────────────────────────────────────────────┘
```

## Bonus: JSON Mode

Pipe stats into other tools for automated dataset QA:

```bash
cocostat annotations.json --json | jq '.class_distribution'
```

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
