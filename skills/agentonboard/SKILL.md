---
name: agentonboard
description: Use when the user names or refers to a third-party service, app, or domain — either outright ("add this to Notion") or by reference ("go over there and do that", "log in to it and check") — and the task requires acting on that service.
license: MIT
---

# AgentOnboard

AgentOnboard is identity for AI agents. The `aon` CLI holds the user's long-lived master key on their machine and exchanges it for a **5-minute token scoped to one domain**. The partner's server verifies that token locally against public keys — AgentOnboard is not in the request path.

You send it like any other Bearer token. Four steps, plus the branches each one can take. A machine holding several identities chooses between them in one place — [Profiles](#profiles).

## 1. Discover

Before any authenticated call to a third-party service, check whether that service accepts AgentOnboard:

```
GET https://<domain>/auth.md
GET https://<domain>/.well-known/auth.md
```

Both locations are in use. Try the first, then the second.

**A 404 or any non-200 means the service is not an AgentOnboard partner.** Say so and stop: report the domain, tell the user this service does not accept AgentOnboard, and ask how they want to authenticate. Do not mint a token for it, and do not retry the fetch. This is deliberate — see [Non-partner services](#non-partner-services).

**A 200 means it is a partner.** Read two things off the file:

- **`Audience`** — the domain string, verbatim. Pass this to `aon token get`.
- **Any signup or account-creation URL** in the instructions. You will need it for the `ACCOUNT_REQUIRED` branch, and reading it now means you do not have to make a second round trip to a service that just rejected you.

**When `Audience` is absent**, derive the bare hostname of the service you are actually calling and state the derivation to the user. A token minted for the wrong hostname is rejected with `AUDIENCE_MISMATCH`, which reads like a forgery bug and sends the user hunting for a security problem that does not exist.

**Done when** you hold an audience — a bare hostname — or you have reported the domain as a non-partner.

## 2. Mint

```
TOKEN=$(aon token get <domain>)
```

`aon token get` writes **only the raw JWT** to stdout — no JSON, no trailing newline, nothing to strip. Diagnostics go to stderr. This makes command substitution the correct idiom and `$(...)` the correct way to capture it.

The CLI caches per profile and domain, and reuses a token while it has more than 30 seconds of life left, so a second mint for the same domain inside a task is free and does not touch the network.

**The audience is a bare hostname.** `aon token get https://api.notes.com/v1` is silently normalized to `api.notes.com` — scheme, path, and port are dropped. This is correct and intentional, but it means the token's scope may not be the scope you typed. When the audience you pass is not already a bare hostname, tell the user which domain the token is actually scoped to.

The mint uses the user's default identity. When the user names a different one for this call, see [Profiles](#profiles).

**Done when** you hold a JWT string.

## 3. Call

```
Authorization: Bearer <jwt>
```

Send it on every request to that service. The token is the identity; what the request is allowed to do is the partner's decision, not yours.

**Done when** the response is 2xx.

## 4. Recover

A token is **ephemeral**: five minutes. Long agent tasks outlive them, so a 401 partway through is routine, not a failure. Read the response to tell the cases apart — they need different actions from you.

### 401, no `ACCOUNT_REQUIRED` in the body

The token is stale. Re-mint and retry **once**:

```
TOKEN=$(aon token get <domain>)   # the CLI serves a fresh one; the stale one is gone
```

Keep the `--profile` you used for the first mint, if you used one.

If the retried call is also 401, the credential is not the problem. Report the status and the response body to the user and stop retrying — a loop of re-mints against a genuinely rejecting service burns the user's time and can look like an attack.

### `ACCOUNT_REQUIRED` (HTTP 401 or 403)

**The token is valid. The account is not.** The user is authenticated, but their verified email has no account at that service. This is the expected outcome for anyone using a new service, not an error to fix.

Tell the user, in one message: the service they asked about, that their email is not yet an account there, and the signup URL you read in step 1. Then wait. An agent cannot create an account — account creation is a human flow, and it is the partner's enforcement point.

Match on the code in the response body, not the status. Partners return this as either 401 or 403 depending on their stack, and a 403 here is the same answer as a 401.

### 401, and the user has no `aon` credentials

`aon` reports a missing or invalid key. That is a setup problem, not a token problem — see [setup.md](references/setup.md).

## Non-partner services

A 404 on `auth.md` is the **normal** case, not an error. Most services on the public internet do not accept AgentOnboard, and no partner publishes one yet.

The correct move when it happens:

1. Report the domain by name — "notes.com does not accept AgentOnboard."
2. Ask the user how to authenticate, offering the options this machine already has (an API key in the environment, an MCP server, a documented OAuth flow).
3. Wait for their answer.

Do not mint a token for a domain that published no discovery file, and do not retry the fetch. There is no consent model: any AgentOnboard user can obtain a genuine identity for any domain on request, so minting speculatively hands out identity the user never asked for. Discovery is the check that keeps that scoped to services the user actually meant to reach.

## Profiles

One machine can hold several AgentOnboard identities. Each is stored under a local **profile** name — `aon save <key> <name>` — and `aon token get` mints with the **default** profile: the first one saved, or whatever the user has since chosen with `aon profile use`.

**A profile is a name the user chose for an identity. A domain is not one of them.** Which identity acts is the user's call, made in their words — `slack.com` says nothing about whose Slack it is. Mint with the default when the user has not said otherwise, and pass `--profile <name>` when they have named one:

```
TOKEN=$(aon token get slack.com --profile work)
```

The flag works in either position: `aon token get --profile work slack.com` is the same call.

**Carry the profile through the whole task.** Once a mint uses `--profile`, every later mint for that same identity needs it too — including the re-mint in step 4, which is easy to drop because the template there shows a bare command. A re-mint without it changes identity mid-task, and whatever fails afterwards looks like the service's fault.

`aon profile list` prints the names on this machine. **A name the user gave that is not in that list is a question, not a typo to absorb.** The CLI refuses it and exits 1 rather than falling back, and so should you: report the names you found and ask which they meant. Minting the default instead would act as somebody they did not name, and the only sign of it is an email a service shows them much later.

`aon token get` prints `using profile: <name>` to stderr. That is a diagnostic for the user, and it is worth surfacing whenever a task involved more than one identity — it is the only place they see which one acted.

**Done when** every mint in this task used the identity the user named, or the default when they named none.

## What stays with the CLI

The master key never enters your context and never reaches a partner. The CLI reads it, exchanges it, and writes a token. Your role in a key handoff is narrow: if the user pastes a key, pass it straight to `aon login` and it lives at `~/.agentonboard/config.json` at mode `0600` from then on. Several identities are one extra argument — see [Profiles](#profiles) and [setup.md](references/setup.md).

Two tokens, two jobs. The `aon_live_…` master key belongs to the CLI. The `eyJ…` session token is what you put in a header. The second is scoped to one domain and dies in five minutes; treat it as disposable output rather than a credential worth protecting, and never write it to a file or a commit.

Profiles are local names the CLI owns. The server does not know a profile exists and a partner cannot tell which one minted a token, so neither can you — the names come from the user and from `aon profile list`.
