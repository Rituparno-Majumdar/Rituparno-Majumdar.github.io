# chunkviz — See Where Your RAG Chunks Actually Break

Every RAG pipeline starts with a bet: that your text splitter produces chunks your
embedding model can actually understand. `chunk_size` and `chunk_overlap` are tuned
blind — you pick numbers, run retrieval, measure, and pray. Nobody shows you the
boundaries, the overlap bleed, or the size spread before you commit.

`chunkviz` renders the splitter's output as a visual report in your terminal, so you
can reason about chunk quality before you spend an afternoon on embedding runs.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/chunkviz.git
cd chunkviz
pip install -e .

chunkviz document.txt --chunk-size 500 --chunk-overlap 50
```

## Real Example

```bash
$ chunkviz README.md --chunk-size 200 --chunk-overlap 20
============================================================
  chunkviz — Text Chunking Visualization
============================================================
  Chunks: 3  |  Chunk size: 200
  Overlap: 20 chars  |  Total text: 450 chars
============================================================

  Chunk 1  ████████████████████████████████████████ 198 chars (99% of target)
  ──────────────────────────────────────────────────────────
  |# chunkviz·\n\nVisualize·recursive·character·text...|

  Chunk 2  ████ 195 chars (97% of target)
  ──────────────────────────────────────────────────────────
  ← overlap from Chunk 1: 'ze recursive character'...
  |boundary·locations,·overlap·regions,·and·size·d...|
```

## Options

| Flag | Default | Description |
|------|---------|-------------|
| `--chunk-size` | 500 | Target chunk size in characters |
| `--chunk-overlap` | 50 | Overlap between consecutive chunks |
| `--separator` | `\n\n` | Primary paragraph separator |

## What the Report Tells You

- **Boundary sanity** — a chunk ending mid-word tells you the separator set is
  wrong for your document; you should be breaking on `\n\n` or `. `, not mid-sentence.
- **Overlap bleed** — the visualized `← overlap` line shows exactly which text gets
  re-embedded in the next chunk. Repeated retrieval of near-duplicate content
  directly inflates storage and API costs.
- **Size distribution** — `min/max/avg` reveals tail chunks that are tiny (wasted
  context) or oversized (hits token limits) — the usual hidden cost in naive splitters.

## Why Zero Dependencies?

Chunking evaluation is a pre-embedding concern — you shouldn't need numpy, a text
splitter library, or a plotting package to see what your own splitter does. `chunkviz`
is a single-file Python CLI on the standard library: install anywhere, pipe into a
notebook, or run it in CI to diff splitter configs.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*