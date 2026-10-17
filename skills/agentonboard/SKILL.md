---
name: agentonboard
description: Use when the user names or refers to a third-party service, app, or domain — either outright ("add this to Notion") or by reference ("go over there and do that", "log in to it and check") — and the task requires acting on that service.
license: MIT
---

# AgentOnboard

AgentOnboard is identity for AI agents. The AgentOnboard Remote MCP server (or `aon` CLI) exchanges the user's API key for a **5-minute token scoped to one domain**. The partner's server verifies that token locally against public keys — AgentOnboard is not in the request path.

You send it like any other Bearer token. Four steps, plus the recovery branches.

## 1. Discover

Before any authenticated call to a third-party service, check whether that service accepts AgentOnboard:

```
GET https://<domain>/auth.md
GET https://<domain>/.well-known/auth.md
```

Both locations are in use. Try the first, then the second.

**A 404 or any non-200 means the service is not an AgentOnboard partner.** Say so and stop: report the domain, tell the user this service does not accept AgentOnboard, and ask how they want to authenticate. Do not mint a token for it, and do not retry the fetch.

**A 200 means it is a partner.** Read two things off the file:

- **`Audience`** — the domain string, verbatim (e.g. `notes.com`). Pass this to `get_token`.
- **The signup instruction** in `## Endpoints` or the rejection table. Account creation is an optional feature for partners: some support agent signup and publish a dedicated endpoint (like `POST /api/signup`), while others do not and serve existing users only via a human connect or registration URL. Read it now so you do not have to fetch `auth.md` again when rejected.

**When `Audience` is absent**, derive the bare hostname of the service you are actually calling. A token minted for the wrong hostname is rejected with `AUDIENCE_MISMATCH`.

**Done when** you hold an audience — a bare hostname — or you have reported the domain as a non-partner.

## 2. Mint

Call the `get_token` MCP tool:

```json
{
  "domain": "<bare-hostname>"
}
```

The tool returns structured JSON in text content:

```json
{
  "token": "eyJhbGciOiJSUzI1NiIs...",
  "expires_at": 1740000300,
  "domain": "notes.com"
}
```

Extract the `token` string.

**The audience is a bare hostname.** `get_token` automatically normalizes input hostnames (e.g. `https://api.notes.com/v1` normalizes to `api.notes.com`).

**Fallback:** If the `get_token` MCP tool is not registered in your agent environment, check if the `aon` CLI is available and run:
```bash
TOKEN=$(aon token get <domain>)
```

**Done when** you hold a JWT string.

## 3. Call

```
Authorization: Bearer <jwt>
```

Send it on every request to that service. The token is the identity; what the request is allowed to do is the partner's decision, not yours.

**Done when** the response is 2xx.

## 4. Recover

A token is **ephemeral**: five minutes. Long agent tasks outlive them, so a 401 partway through is routine, not a failure. Read the response to tell the cases apart:

### 401, no `ACCOUNT_REQUIRED` in the body

The token is stale. Re-mint by calling `get_token` again and retry **once**.

If the retried call is also 401, the credential is not the problem. Report the status and the response body to the user and stop retrying.

### `ACCOUNT_REQUIRED` (HTTP 401 or 403)

**The token is valid. The account is not.** The user is authenticated, but their verified email has no account at that service. This is the expected outcome for anyone using a new service, not an error to fix.

Check what the partner published in `auth.md`:

- **Partner publishes an agent signup endpoint** (e.g. `POST /api/signup`): call it using the same bearer token, following the request shape in `auth.md`. On success, retry the original request **once**. If the signup call fails or the retried request still returns `ACCOUNT_REQUIRED`, stop and report the error to the user — do not retry in a loop.
- **Partner requires human connection or registration** (names a connect/signup URL, or publishes no agent signup endpoint): tell the user in one message which service was called, that their email is not yet an account there, and the URL from `auth.md`. Then wait for the human.

### 401 on `get_token` or Missing Tool

The AgentOnboard MCP server is not connected or the API key is invalid/revoked. Ask the user to verify their MCP connection and API key on the AgentOnboard dashboard.

## Diagnostics

Call the `whoami` MCP tool (no parameters) to verify your connected account email and key prefix.

## Security Rules

- The master key never enters your context and never reaches a partner.
- The 5-minute session token is disposable output scoped to one domain; never send a token minted for `domain-a.com` to `domain-b.com`.
- Never write the API key or token to a repository commit or persistent log file.
