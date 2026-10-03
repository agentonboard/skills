# agentonboard/skills

The AgentOnboard agent skill. Teaches a coding agent to authenticate with a service the way an agent should: discover that the service accepts AgentOnboard, mint a short-lived scoped token with the `aon` CLI, send it as a Bearer token, and recover when it expires.

## Install

```bash
npx skills add agentonboard/skills
```

Then restart your agent session. Skills are read at startup.

The skill assumes the [`aon` CLI](https://agentonboard.xyz) is installed. For the full one-time setup — CLI, key, verification — see [the quickstart](https://agentonboard.xyz/docs/connect).

## What it does

Four steps: **discover** (`GET https://<domain>/auth.md`), **mint** (`aon token get <domain>`), **call** (`Authorization: Bearer <jwt>`), **recover** (re-mint on expiry, surface `ACCOUNT_REQUIRED` to the human).

A machine can hold several AgentOnboard identities, each stored under a local profile name. The skill passes `--profile` when the user names one, and mints with the default when they do not — which identity acts follows the user's words, not the domain being called.

The `auth.md` discovery file is the open convention introduced by WorkOS. A service that does not publish one is not an AgentOnboard partner, and the skill says so rather than guessing.

## Files

| File | Contents |
|---|---|
| [`SKILL.md`](skills/agentonboard/SKILL.md) | The signing flow: discover, mint, call, recover, plus profiles. |
| [`setup.md`](skills/agentonboard/references/setup.md) | One-time machine setup, including storing several keys. Reached by pointer, not on load. |

## Why a separate repo

The skill is the agent-facing half of AgentOnboard's contract with its users; `agentonboard/skills` is distributed on its own, to machines that are not running the product. The CLI and SDK live in the `agentonboard` monorepo.

**When the CLI or token contract changes, update this repo in the same commit.** `aon`'s flags and subcommands, the audience normalization rule, the status codes for `ACCOUNT_REQUIRED`, and the `auth.md` locations are all defined in the monorepo — this repo restates none of them deliberately, so nothing here self-validates. If the two disagree, the skill is wrong.
# agentonboard/skills

The AgentOnboard agent skill. Teaches a coding agent to authenticate with a service the way an agent should: discover that the service accepts AgentOnboard, mint a short-lived scoped token with the `aon` CLI, send it as a Bearer token, and recover when it expires.

## Install

```bash
npx skills add agentonboard/skills
```

Then restart your agent session. Skills are read at startup.

The skill assumes the [`aon` CLI](https://agentonboard.xyz) is installed. For the full one-time setup — CLI, key, verification — see [the quickstart](https://agentonboard.xyz/docs/connect).

## What it does

Four steps: **discover** (`GET https://<domain>/auth.md`), **mint** (`aon token get <domain>`), **call** (`Authorization: Bearer <jwt>`), **recover** (re-mint on expiry, surface `ACCOUNT_REQUIRED` to the human).

A machine can hold several AgentOnboard identities, each stored under a local profile name. The skill passes `--profile` when the user names one, and mints with the default when they do not — which identity acts follows the user's words, not the domain being called.

The `auth.md` discovery file is the open convention introduced by WorkOS. A service that does not publish one is not an AgentOnboard partner, and the skill says so rather than guessing.

## Files

| File | Contents |
|---|---|
| [`SKILL.md`](skills/agentonboard/SKILL.md) | The signing flow: discover, mint, call, recover, plus profiles. |
| [`setup.md`](skills/agentonboard/references/setup.md) | One-time machine setup, including storing several keys. Reached by pointer, not on load. |

## Why a separate repo

The skill is the agent-facing half of AgentOnboard's contract with its users; `agentonboard/skills` is distributed on its own, to machines that are not running the product. The CLI and SDK live in the `agentonboard` monorepo.

**When the CLI or token contract changes, update this repo in the same commit.** `aon`'s flags and subcommands, the audience normalization rule, the status codes for `ACCOUNT_REQUIRED`, and the `auth.md` locations are all defined in the monorepo — this repo restates none of them deliberately, so nothing here self-validates. If the two disagree, the skill is wrong.
