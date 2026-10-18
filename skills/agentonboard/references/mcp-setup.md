# Agent MCP Setup

Reached when an agent needs to connect to the AgentOnboard Remote MCP server, or when the `get_token` tool is missing or returns 401.

AgentOnboard uses a Remote MCP server (`https://agentonboard.xyz/api/mcp`) so agents can mint short-lived session tokens without installing any local machine CLI or storing credentials globally.

## Prerequisites

1. An AgentOnboard API key starting with `aon_live_` from the [AgentOnboard Dashboard](https://agentonboard.xyz).
2. The AgentOnboard skill installed in your agent environment:
   ```bash
   npx skills add agentonboard/skills
   ```

---

## Client Setup

Take whichever route the user prefers:
- **Self-setup (Recommended)**: Give the user the exact command or config snippet below with `<YOUR_KEY>` placeholder. They run or edit it themselves, keeping the API key completely out of chat context.
- **Assisted setup**: The user pastes their `aon_live_...` key in chat. Run the command or write the config file directly, and never echo the key back into the conversation or persist it in a repository.

### Claude Code

Run in terminal:
```bash
claude mcp add --transport http --scope user agentonboard https://agentonboard.xyz/api/mcp --header "Authorization: Bearer <YOUR_KEY>"
```
Restart or exit the session (`/exit`) to reload MCP tools.

### Cursor

Add to global `~/.cursor/mcp.json` (or workspace `.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "agentonboard": {
      "type": "http",
      "url": "https://agentonboard.xyz/api/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_KEY>"
      }
    }
  }
}
```
Reload MCP servers or restart Cursor.

### Codex CLI

Run in terminal:
```bash
export AGENTONBOARD_API_KEY="<YOUR_KEY>"
codex mcp add agentonboard --url https://agentonboard.xyz/api/mcp --bearer-token-env-var AGENTONBOARD_API_KEY
```
Reload with `/mcp` or restart session.

### Claude Desktop

Add to `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`, Windows: `%APPDATA%\Claude\claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "agentonboard": {
      "type": "http",
      "url": "https://agentonboard.xyz/api/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_KEY>"
      }
    }
  }
}
```
Restart Claude Desktop.

### OpenCode

Run in terminal:
```bash
opencode mcp add agentonboard --url https://agentonboard.xyz/api/mcp
```
Or add to `~/.config/opencode/opencode.json`:
```json
{
  "mcp": {
    "agentonboard": {
      "type": "remote",
      "url": "https://agentonboard.xyz/api/mcp",
      "headers": {
        "Authorization": "Bearer <YOUR_KEY>"
      }
    }
  }
}
```

### Hermes Agent

Run in terminal:
```bash
hermes mcp add agentonboard --url https://agentonboard.xyz/api/mcp
```
Or add to `~/.hermes/config.yaml`:
```yaml
mcp_servers:
  agentonboard:
    type: http
    url: https://agentonboard.xyz/api/mcp
    headers:
      Authorization: Bearer <YOUR_KEY>
```

### Pi Agent

Run in terminal:
```bash
export AGENTONBOARD_API_KEY="<YOUR_KEY>"
pi mcp add agentonboard --url https://agentonboard.xyz/api/mcp --bearer-token-env-var AGENTONBOARD_API_KEY
```

### OpenClaw

Run in terminal:
```bash
openclaw mcp add agentonboard --type http --url https://agentonboard.xyz/api/mcp
```
Or add to `~/.openclaw/openclaw.json`:
```json
{
  "mcp": {
    "servers": {
      "agentonboard": {
        "type": "http",
        "url": "https://agentonboard.xyz/api/mcp",
        "headers": {
          "Authorization": "Bearer <YOUR_KEY>"
        }
      }
    }
  }
}
```

---

## Verification

Once configured, verify connectivity by calling the `whoami` MCP tool (no parameters).

**Expected output:**
```json
{
  "email": "user@example.com",
  "key_prefix": "aon_live_..."
}
```

If `whoami` returns your email, the agent is connected and ready to mint scoped tokens with `get_token`.

## Diagnostics & Troubleshooting

| Symptom | Root Cause | Fix |
|---|---|---|
| `Tool get_token not found` | Agent session was not reloaded after MCP was added. | Restart agent session or reload MCP client tools. |
| 401 Unauthorized from MCP | Key is invalid, revoked, or formatted incorrectly. | Verify key on AgentOnboard dashboard. Check that header has `Bearer aon_live_...`. |
| Connection refused or timeout | Network issue or firewall blocking `agentonboard.xyz`. | Ensure agent environment can reach `https://agentonboard.xyz/api/mcp`. |
