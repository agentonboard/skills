# agentonboard/skills

The AgentOnboard agent skill. Teaches a coding agent to authenticate with a service the way an agent should: discover that the service accepts AgentOnboard, mint a short-lived scoped token via the AgentOnboard Remote MCP server, send it as a Bearer token, and recover when it expires.

## Install

```bash
npx skills add agentonboard/skills
```

Then restart your agent session. Skills are read at startup.

The skill uses the AgentOnboard Remote MCP server (`https://agentonboard.xyz/api/mcp`). For client setup instructions across Claude Code, Cursor, Codex, OpenCode, Hermes, Pi, and others, see [the MCP setup guide](skills/agentonboard/references/mcp-setup.md).

## What it does

Four steps: **discover** (`GET https://<domain>/auth.md`), **mint** (`get_token` MCP tool), **call** (`Authorization: Bearer <jwt>`), **recover** (re-mint on expiry, sign up or surface `ACCOUNT_REQUIRED` to the human).

The `auth.md` discovery file is the open convention introduced by WorkOS. A service that does not publish one is not an AgentOnboard partner, and the skill says so rather than guessing.

## Files

| File | Contents |
|---|---|
| [`SKILL.md`](skills/agentonboard/SKILL.md) | The signing flow: discover, mint, call, recover. |
| [`mcp-setup.md`](skills/agentonboard/references/mcp-setup.md) | Setting up the AgentOnboard Remote MCP connection across various agent runners. Reached by pointer, not on load. |

## Why a separate repo

The skill is the agent-facing half of AgentOnboard's contract with its users; `agentonboard/skills` is distributed on its own, to machines and agent environments that are not running the product. The SDK, server, and core packages live in the `agentonboard` monorepo.

**When the token contract or MCP tools change, update this repo in the same commit.** The audience normalization rule, the status codes for `ACCOUNT_REQUIRED`, the `auth.md` locations, and the MCP tool schema are all defined in the monorepo — this repo restates none of them deliberately, so nothing here self-validates. If the two disagree, the skill is wrong.
