# COCO Summary — Inspect COCO Datasets From Your Terminal

Inspect any COCO-format annotation JSON file in one command. See class distribution, bounding box statistics, and quality flags — no Python scripting required.

## Quick Start

```bash
pip install coco-summary
coco-summary instances_train.json
```

## Why This Exists

Every object detection dataset uses COCO JSON format, but there's no quick way to inspect it from the terminal. You either open a multi-megabyte JSON in an editor or write throwaway Python scripts. `coco-summary` prints a formatted report with class distribution, bbox stats, degenerate box detection, and out-of-bounds checking — zero dependencies, one command.

## Real Example

```bash
# Before: loading a COCO JSON in Python just to see class counts
python -c "import json; d=json.load(open('anns.json')); print(len(d['annotations']))"

# After:
coco-summary instances_val2017.json

# --------------------------------------------------
#   COCO Dataset Summary
# --------------------------------------------------
#   Images:              5,000
#   Annotations:         36,781
#   Categories:          80
#   Images w/o anns:     12
#   Avg anns/image:      7.36
#   Degenerate bboxes:   0
#   Out-of-bounds:       3
#   Bbox area:           min=4.0  max=442,368.0  avg=12,845.2  median=6,120.0
#   Class Distribution (80 classes)
#   ----------------------------------------
#   person          11,234  (30.5%)  ████████████████████
#   car              2,156   (5.9%)  ████
#   ...
# --------------------------------------------------
```

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
