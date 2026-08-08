# skillcheck — Validate Agent Skills (SKILL.md) in One Command

Agent Skills are the fastest-growing way to give AI agents durable capability — but a skill is just a directory with a hand-written `SKILL.md`. A typo in the YAML frontmatter or a missing `scripts/run.sh` **silently breaks the skill**: the agent never loads it, and you only find out in production when the capability is needed. There's no runtime error.

`skillcheck` is the zero-dependency lint pass for Agent Skill directories — catch the failures before your agent silently loses a skill.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/skillcheck.git
cd skillcheck
pip install -e .

# Scan the current directory (recursively)
skillcheck

# Scan specific skill directories or search roots
skillcheck ~/.claude/skills ./my-skills

# Machine-readable output for CI
skillcheck --json
```

## What It Checks

| Check | Example | Detects |
|-------|---------|---------|
| `SKILL.md` exists | Empty skill folder | A skill directory without it is invisible to the agent |
| `name` present & valid slug | `name: My_Skill` | Loaders reject non-slug names; must match the directory name |
| `description` present | Missing or `<20` chars | Empty descriptions make the skill unfindable during tool-selection |
| `allowed-tools` is a string list | `allowed-tools: bash` | Wrong type silently drops tool permissions |
| `license` is a string | `license: 5` | Malformed license blocks ingestion |
| Referenced files exist | `scripts/run.sh` missing | The skill runs but fails at the worst moment |
| Frontmatter parses | Broken `---` block | A broken block makes the whole file unreadable |

## Real Example

```bash
$ skillcheck ~/.claude/skills
invalid-skill [invalid-name] name 'Invalid_Name' is not a valid slug (lowercase, hyphens)
invalid-skill [weak-description] description is empty or too short (<20 chars)
invalid-skill [invalid-field] allowed-tools must be a list of strings
broken-ref-skill [broken-reference] referenced file missing: scripts/missing.sh
```

## Exit Codes (CI-friendly)

- `0` — All skills valid
- `1` — One or more issues found

## Why Zero Dependencies?

The 2026 agentic era runs on skills — Claude Code, opencode, and every major agent framework load them from directories. `skillcheck` is a single-file Python CLI on the standard library: install anywhere in seconds, run in CI before shipping a skill pack, or ad-hoc whenever you hand-edit a `SKILL.md`.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
