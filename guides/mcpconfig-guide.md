# mcpconfig — Validate MCP Client Configuration in One Command

MCP servers are wired into agents through config files that are hand-edited JSON: `claude_desktop_config.json` (Claude Desktop), `.mcp.json` (Claude Code / project-scoped), and `opencode.json` `mcp:` blocks. A single mistake — `command` as a number, `type: "telepathy"`, a missing `url` on an SSE server, a duplicate server name — and the server **silently fails to load**. There's no error until the agent tries to call a tool and finds it gone.

`mcpconfig` does one thing: scan your MCP client config files and report every structural problem before your tools disappear.

## Quick Start

```bash
git clone https://github.com/Rituparno-Majumdar/mcpconfig.git
cd mcpconfig
pip install -e .

# Scan the current directory for config files
mcpconfig

# Point at specific files
mcpconfig ~/.config/claude/claude_desktop_config.json .mcp.json opencode.json

# CI-ready JSON output
mcpconfig --json
```

## Real Example

```bash
$ mcpconfig .mcp.json
───────────────────────────────────────────────────
  mcpconfig — MCP Config Report
───────────────────────────────────────────────────

  ❌ 6 issue(s) found:

    • .mcp.json [duplicate] server name already defined
    • .mcp.json [name] empty server name
    • .mcp.json [command] command must be a string or array of strings
    • .mcp.json [env] env value 'HOME' must be a string
    • .mcp.json [type] invalid type 'telepathy', expected one of ...
    • .mcp.json [url] url must be a string

  Checked 1 file(s), 6 issue(s).
```

## Exit Codes (CI-friendly)

- `0` — All configs valid (CI green)
- `1` — One or more issues found (CI red)
- `2` — No config files found / usage error

## What It Checks

| Check | Description |
|-------|-------------|
| **Structure** | Root must be an object with an `mcpServers` or `mcp` key |
| **Names** | Server names non-empty, no duplicates |
| **Type** | `type` must be one of `stdio`, `sse`, `streamable-http`, `local`, `remote` |
| **Command** | Present, a string or array of strings, never empty |
| **Args** | Must be an array of strings |
| **Env** | Must be an object with string values |
| **URL** | `sse`/`remote`/`http` types require a `url` |
| **Enabled** | Boolean only (opencode-style) |
| **JSONC tolerance** | Strips `//` comments and trailing commas for opencode-style files |

## Why Zero Dependencies?

MCP config files live on every developer machine, in every agent setup — including CI runners that shouldn't install Node.js just to lint a JSON file. `mcpconfig` is a single-file Python CLI on the standard library: install anywhere, pipe into `jq`, fail a GitHub Action with a clean exit code.

Pipe `mcpconfig` into your agent onboarding or CI — a non-zero exit on a broken config is the cheapest way to guarantee your tools actually load.

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
