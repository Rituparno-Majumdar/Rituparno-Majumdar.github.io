# Running Multi-Agent Research with OpenCode

A practical guide to launching parallel research agents inside OpenCode using the [`opencode-research-agents`](https://github.com/Rituparno-Majumdar/opencode-research-agents) toolkit.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/opencode-research-agents.git
cd opencode-research-agents
```

## Launch Parallel Agents

Create a topics file and fire agents in parallel:

```bash
echo "rag-retrieval-augmentation" > topics.txt
echo "prompt-injection-defenses" >> topics.txt
echo "llm-evaluation-methods" >> topics.txt

python3 agent_orchestrator.py --topics topics.txt --output research/
```

## What Happens

1. Each agent loads its research question independently
2. Agents search, extract, and synthesize findings from the web
3. Results are saved as structured Markdown to `research/<topic>/`
4. A cross-cutting summary is generated in `research/synthesis.md`

## Why Parallel?

Sequential research lets each subsequent agent build on prior findings. But **parallel epistemic parallelism** — assigning the same question to 3 agents with different model backends — reveals bias patterns in synthesized outputs. Use it for robust literature reviews.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
