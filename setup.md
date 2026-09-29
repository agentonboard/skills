# Machine setup

Reached when the user asks to connect AgentOnboard to this machine, or when `aon` is missing or unconfigured mid-task. The signing flow does not depend on this file.

## Do the steps

Run them in order. Each one's check is the last line, and the checks are what tell you the next step is safe to take — an install that has not been verified can leave a `aon` on PATH that is broken, and every later step would then fail for a reason that looks like a product bug.

### 1. Install the CLI

```bash
curl -fsSL https://agentonboard.xyz/install.sh | sh
```

Run it as the user, not with `sudo`. It writes to `~/.agentonboard/bin` and appends one line to your shell profile.

**Windows (PowerShell):**

```powershell
irm https://agentonboard.xyz/install.ps1 | iex
```

**npm, if the shell installer is unavailable:**

```bash
npm install -g @agentonboard/cli
```

If the install adds `~/.agentonboard/bin` to PATH, a shell started in this session will not see it until that profile is reloaded. Either open a new shell or prepend the directory for the rest of this session — and say which you did, so the next failure is not blamed on a stale shell.

**Check:** `aon --version` prints a version.

### 2. Install this skill

```bash
npx skills add agentonboard/skill
```

Tell the user this one needs a **session restart** to take effect — the agent reads its skills at startup, so the signing flow is not live until they do.

**Check:** the user confirms the skill is listed.

### 3. Save the master key

The key starts with `aon_live_` and comes from the dashboard's **Keys** page.

Ask the user for it, and take whichever route they prefer:

- **They paste it** — run `aon login <key>` yourself. The key goes straight to the CLI and is never echoed into the conversation.
- **They run it themselves** — tell them to run `aon login` in their own terminal and confirm when it is done. This keeps the key out of the conversation entirely, and is the better default when they are hesitant.

The CLI stores it at `~/.agentonboard/config.json`, mode `0600`. It is already saved the moment `aon login` returns; there is no separate save step.

**Check:** `aon login` exits without error.

### 4. Verify

```bash
aon doctor
```

This checks Node version, the config file and its permissions, the token cache, and live connectivity to the identity engine against the saved key.

**Check:** doctor ends with a passing result.

### 5. Report back

Tell the user three things: the CLI version, whether the skill is installed, and doctor's result. Then stop.

## When a check fails

Report the failing check and its output, and do not go past it. Each one blocks the next:

| Symptom | What it means |
|---|---|
| `aon: command not found` | PATH. New shell, or export `~/.agentonboard/bin`. |
| Doctor cannot reach the engine | The CLI is pointed at `http://localhost:3000` instead of production — it defaults to localhost, so a config written by an older build carries the wrong URL. Fix with `export AON_API_URL=https://agentonboard.xyz`, or re-run `aon login` after setting it. |
| `Invalid or nonexistent API key` | The key is wrong, truncated, or revoked. Never guess at a key. Ask the user for a fresh one from the Keys page. |
| `This API key has been revoked` | It was revoked. Revoking is final unless the user clicks Undo within ten minutes on the dashboard. Say so — an agent cannot undo it. |

A key the user has since rotated is the common case here, and it is the one failure the agent cannot resolve on its own.

## Handling the key

The key is a credential. Pass it to `aon login` and let it go — do not echo it back, do not write it to a file, do not put it in a commit, a log, or a test fixture. If the user pastes one, confirm it is saved and move on without restating it.

`aon doctor` prints the first 16 characters of the key and the user's email. That output goes to the user, not to a log or an issue.
