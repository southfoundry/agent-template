# Operating Instructions

## Mailroom Protocol

The Mailroom is the agent coordination channel. Follow these rules strictly.

**Root channel = notifications only.** Everything else goes in a thread.

Root-level messages:
* 📋 `[Request] @agent — do the thing`
* ✅ `[Done] @requester — thing is done`
* 🚨 `[Blocked] @agent — can't do X, need Y`
* 📡 `[Infra Change] what changed — impact. No reply needed.`
* 📢 `[Skill Update] skill — what changed`

**If your name is not in the message, you DO NOT text-reply.** You may react with emoji.

**Claim protocol:** First agent to react with 👀 claims the task. If you see 👀, back off.

See the `mailroom` skill for full protocol.

## Mailroom Broadcasting (MANDATORY)

After ANY mutation you make (git push, config change, skill update, deployment change,
etc.), broadcast to the Mailroom:

```
📡 [Infra Change] <what changed> — <impact>. No reply needed.
```
or for skills:
```
📢 [Skill Update] <skill> — <what changed>
```

This is NOT optional. It is a side effect of completing work, not a separate task.
Other agents MUST react with an emoji (🔧👺🧌🐸👀⚙️) to confirm pickup.

When you SEE an `[Infra Change]` or `[Skill Update]` broadcast from another agent,
react with an emoji to confirm you received it.

## Coordinated Restarts

Agents do NOT have auto-restart (no reloader). Restarts are coordinated through Flem.

If a broadcast affects YOUR config (workspace, skills, agent config) and you want
to pick up the changes, wait until you're idle then post to the Mailroom:
```
[Request] @flem — ready for restart, config updated
```
Flem will restart your pod and confirm. Do NOT restart yourself.

## Safety

* Don't exfiltrate private data. Ever.
* Don't run destructive commands without asking.
* `trash` > `rm` when available (recoverable beats gone forever)
* When in doubt, ask.

## {{AGENT_SPECIFIC_SECTION}}

Add agent-specific operating instructions here (tools, paths, workflows, etc.)
