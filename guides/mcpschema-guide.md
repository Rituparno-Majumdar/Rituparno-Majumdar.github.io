# mcpschema — Validate MCP Tool Schemas in One Command

MCP (Model Context Protocol) tools are defined using JSON Schema. When a tool definition has a missing `name`, an empty `description`, or an invalid property type, AI agents fail silently — they refuse to call the tool, generate malformed arguments, or waste context tokens on retries. `mcpschema` catches these issues before your agent ever sees them.

## Quick Start

```bash
pip install -e /root/mcpschema

# Check any MCP tool definition file
mcpschema tools.json --verbose
```

## Why This Exists

Before `mcpschema`, validating MCP tool definitions meant:
- Manually inspecting JSON Schema files hoping to spot errors
- Discovering broken tool calls only during agent runtime
- Debugging silent failures with no clear error messages

`mcpschema` does one thing well: it validates MCP tool JSON Schema definitions and reports every issue.

## Real Example

```bash
# Bad tool definition (missing name, bad type)
$ cat bad-tools.json
[
  {
    "name": "",
    "description": "Get weather",
    "inputSchema": {
      "type": "object",
      "properties": {
        "city": {"type": "text", "description": ""}
      },
      "required": ["city", "country"]
    }
  }
]

$ mcpschema bad-tools.json --verbose
──────────────────────────────────────────────────
  mcpschema — MCP Tool Schema Report
──────────────────────────────────────────────────
  Files scanned     : 1
  Files with issues : 1
  Total issues      : 5

  bad-tools.json
    ⚠ Missing or empty 'name' field: bad-tools.json > ?
    ⚠ Description too short (10 chars): bad-tools.json > ?
    ⚠ Property 'city' has unknown type 'text': bad-tools.json > ?
    ⚠ Property 'city' has empty description: bad-tools.json > ?
    ⚠ Required property 'country' not defined in 'properties': bad-tools.json > ?

  ⚠ 5 issue(s) found across 1 file(s).
```

## What mcpschema Checks

1. **Name validity** — non-empty, no special characters
2. **Description quality** — at least 10 characters, not empty
3. **inputSchema structure** — must be a JSON Schema object with `type: object`
4. **Property types** — only valid JSON Schema types allowed (`string`, `number`, `integer`, `boolean`, `array`, `object`, `null`)
5. **Required references** — every item in `required` must exist in `properties`
6. **Property descriptions** — no empty descriptions on properties

---

*Part of the [Rituparno Majumdar](https://github.com/Rituparno-Majumdar) daily project pipeline.*
