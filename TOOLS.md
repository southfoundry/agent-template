# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff
that's unique to your setup.

## What Goes Here

* Camera names and locations
* SSH hosts and aliases
* Preferred voices for TTS
* Device nicknames
* API endpoints
* Anything environment-specific

Add whatever helps you do your job. This is your cheat sheet.

## 1Password (Shared Vault)

**Vault:** `openclaws`
**Auth:** Service account token in env var `OP_SERVICE_ACCOUNT_TOKEN`

All agents share one vault. Items follow the naming convention:
```
<scope>/<service>[/<qualifier>]
```
Scopes: `shared/`, `weeb/`, `milton/`, `flem/`, `morrow/`, `edwina/`, `navexa/`, `foundry/`

```bash
op item list --vault openclaws          # List all
op read "op://openclaws/shared/..."     # Read a secret
```

Never log secret values. Use `op run` / `op inject` at runtime.

## Threading (Matrix & Slack)

**Thread** = proper threaded discussion (messages in thread panel, not root).
**Reply** = quote-reply in root (clutters the room).

### Matrix
```
# Thread reply (what we want for mailroom):
message send, channel: matrix, target: <room>, threadId: <parent_message_id>, message: "reply"

# Quote-reply in root (NOT a thread — avoid in mailroom):
message send, channel: matrix, target: <room>, replyTo: <message_id>, message: "reply"
```

### Slack
```
# Thread reply:
message send, channel: slack, target: <channel>, threadId: <parent_message_ts>, message: "reply"
```

### Mailroom Protocol
- Acknowledging a root message → `threadId: <that_message_id>`
- Discussion, status updates → `threadId: <original_request_id>`
- New `[Request]`, `[Done]`, `[Blocked]`, `[Skill Update]`, `[Infra Change]` → root (no threadId)
