# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff
that's unique to your setup.

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

## What Goes Here

* Camera names and locations
* SSH hosts and aliases
* Preferred voices for TTS
* Device nicknames
* API endpoints
* Anything environment-specific

Add whatever helps you do your job. This is your cheat sheet.

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
