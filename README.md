# Cursor Integration for Preflights

This integration enables Cursor IDE to use Preflights for architecture decision automation via MCP (Model Context Protocol).

## What This Integration Does

- **MCP Server**: Cursor connects to `preflights-mcp` to call clarification tools
- **Guardrails**: Rules that prevent Cursor from making code changes without proper task definition
- **Workflow**: Ensures architectural decisions are documented before implementation

## Requirements

1. **Preflights CLI** installed and available on PATH
2. **preflights-mcp** command available (installed with Preflights)
3. **Cursor IDE** with MCP support enabled

### Installation

```bash
# Install Preflights
pip install preflights

# Verify MCP server is available
preflights-mcp --help  # Should show server info (or start waiting for stdio)
```

## Setup

### 1. Configure MCP Server in Cursor

Copy the MCP configuration to your Cursor settings. The exact location depends on your Cursor version:

- **Cursor Settings** > **MCP** > Add new server
- Or edit your MCP configuration file directly

Use the configuration from `cursor-mcp.json`:

```json
{
  "preflights": {
    "command": "preflights-mcp",
    "args": [],
    "env": {}
  }
}
```

**Important**: Cursor should launch the MCP server with `cwd` set to your repository root. If not, you can pass `repo_path` explicitly in tool calls.

### 2. Add Cursor Rules to Your Repository

Copy `cursorrules.template` to `.cursorrules` in your repository root:

```bash
cp integrations/cursor/cursorrules.template /path/to/your/repo/.cursorrules
```

This ensures Cursor follows the Preflights workflow.

### 3. (Optional) Configure System Instructions

If Cursor supports custom system instructions, add the content from `prompts/cursor_system_instructions.md` to reinforce the guardrails.

## Usage

### Standard Workflow

1. **Ask Cursor to implement a feature**
   ```
   "Add OAuth authentication to the user login flow"
   ```

2. **Cursor checks for `docs/CURRENT_TASK.md`**
   - If missing: Cursor calls `require_clarification` MCP tool
   - Preflights asks clarifying questions
   - User answers via Cursor chat
   - Preflights generates `CURRENT_TASK.md`, `AGENT_PROMPT.md`, and optionally ADR

3. **Cursor proceeds with implementation**
   - Reads `docs/AGENT_PROMPT.md` for instructions
   - Implements strictly according to `docs/CURRENT_TASK.md`
   - Respects allowlist/forbidden paths

### MCP Tools Available

| Tool | Purpose |
|------|---------|
| `require_clarification` | Start/continue clarification session |
| `read_architecture` | Read current architecture state |

### Tool Parameters

**require_clarification**:
```json
{
  "user_intention": "string (required)",
  "optional_context": "string",
  "session_id": "string (for multi-turn)",
  "answers": {"question_id": "answer"},
  "repo_path": "string (optional, defaults to cwd)"
}
```

**read_architecture**:
```json
{
  "repo_path": "string (optional, defaults to cwd)"
}
```

## Generated Artifacts

After successful clarification, Preflights creates:

| File | Purpose |
|------|---------|
| `docs/CURRENT_TASK.md` | Task specification with allowlist, constraints, acceptance criteria |
| `docs/AGENT_PROMPT.md` | Ready-to-use prompt for AI coding agents |
| `docs/ARCHITECTURE_STATE.md` | Current architecture snapshot |
| `docs/adr/*.md` | Architecture Decision Records (when decisions are made) |

## Troubleshooting

### "repo targeting" Issues

If Preflights creates files in the wrong location:

1. **Check Cursor's MCP cwd**: Ensure Cursor launches `preflights-mcp` from your repo root
2. **Pass explicit repo_path**: In tool calls, include `"repo_path": "/absolute/path/to/repo"`
3. **Verify .git exists**: Preflights discovers repo root by climbing up to find `.git`

### Cursor Not Calling Preflights

1. **Check MCP is configured**: Verify `preflights` appears in Cursor's MCP servers list
2. **Check .cursorrules**: Ensure the rules file exists and contains the guardrails
3. **Restart Cursor**: MCP servers may need a restart to pick up config changes

### Session Errors

If you get `SESSION_NOT_FOUND`:
- The session expired or server restarted
- Start a new clarification session (omit `session_id`)

## Best Practices

1. **Always let Preflights handle clarification** - Don't manually create `CURRENT_TASK.md`
2. **Review generated ADRs** - They document architectural decisions for the team
3. **Keep .cursorrules in version control** - Team-wide consistency
4. **Use `read_architecture` before major changes** - Understand current state

## Files in This Integration

```
integrations/cursor/
├── README.md                          # This file
├── cursor-mcp.json                    # MCP server configuration
├── cursorrules.template               # Template for .cursorrules
└── prompts/
    ├── cursor_system_instructions.md  # System-level instructions
    └── cursor_user_prompt_template.md # User prompt examples
```
