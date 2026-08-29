# annocheck — Validate CV Datasets in One Command

YOLO annotation corruption is the silent killer of CV training runs. One bad bounding box or a missing image reference can waste hours of training time. `annocheck` catches these issues instantly.

## Quick Start

```bash
pip install -e /root/annocheck

# Check any YOLO-format dataset
annocheck ./coco-subset --verbose
```

## Why This Exists

Before `annocheck`, validating a CV dataset meant writing ad-hoc Python scripts, or worse — noticing the problem only after training crashed 3 hours in. Common issues like out-of-bounds boxes (e.g., `x_center=1.02`), missing image files, or orphaned images with no labels are invisible until they break your pipeline.

`annocheck` runs a comprehensive check in under a second and tells you exactly what's wrong and where.

## Real Example

```bash
# Before: Manually grep through label files hoping to find issues
find labels/ -name "*.txt" -exec cat {} \; | awk '{if($2>1||$3>1) print}'  # misses so much

# After:
annocheck ./dataset --verbose
# ──────────────────────────────────────────────────
#   annocheck — Annotation Sanity Report
# ──────────────────────────────────────────────────
#   Label files      : 500
#   Images           : 510
#   Total annotations: 1,234
#   Avg anns/file    : 2.5
#
#   ❌ ERRORS (1):
#      • No matching image for annotation file: img042.txt (stem=img042)
#
#   ⚠  WARNINGS (3):
#      • img042.txt:1: x_center=1.02 (expected 0–1)
#      • img137.txt:2: width=-0.05 (expected 0–1)
#
#   🗑  Orphaned images (no label file): 10
#     • stray_img_01.jpg
#     • stray_img_02.jpg
#
#   📊 Class Distribution:
#        0:    456 (37.0%) ██████████████████
#        1:    234 (19.0%) █████████
#        2:    544 (44.1%) ██████████████████████
```

The error (`img042.txt`) references an image that doesn't exist — that label file is training dead weight. The orphaned images mean data you labelled is never being used. Fix both, re-run, and train with confidence.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
