# OpenClaw Features Guide

This is a reference for how OpenClaw features work and when to use them.
When adding new capabilities to an agent, check this guide first — the
platform probably has a built-in mechanism that's more reliable than a
custom skill.

**Rule of thumb:** If it's about *when* something runs, *what triggers it*,
or *how the agent behaves consistently* — it's a platform feature, not a skill.
Skills are for *how to interact with external tools and APIs*.

---

## Table of Contents

1. [Workspace Files (Context Injection)](#workspace-files)
2. [Standing Orders (Autonomous Programs)](#standing-orders)
3. [Cron Jobs (Scheduled Tasks)](#cron-jobs)
4. [Heartbeats (Periodic Awareness)](#heartbeats)
5. [Hooks (Event-Driven Automation)](#hooks)
6. [Webhooks (External Triggers)](#webhooks)
7. [BOOT.md (Startup Tasks)](#bootmd)
8. [Context Engine](#context-engine)
9. [Broadcast Groups](#broadcast-groups)
10. [Multi-Agent Routing](#multi-agent-routing)
11. [Decision Matrix: What to Use When](#decision-matrix)

---

## Workspace Files

**What:** Files auto-injected into every session as context.
**When to use:** Persistent identity, behavior rules, memory.

These files are loaded at the start of every session — you don't need
to read them manually, they're in context automatically:

| File | Purpose |
|------|---------|
| `AGENTS.md` | Operating instructions, rules, standing orders |
| `SOUL.md` | Persona, tone, personality |
| `USER.md` | Who the human is, how to address them |
| `IDENTITY.md` | Agent name, emoji, creature type |
| `TOOLS.md` | Local tool notes, environment specifics |
| `HEARTBEAT.md` | Checklist for heartbeat runs |
| `BOOT.md` | Startup tasks on gateway restart |
| `MEMORY.md` | Curated long-term memory (main session only) |
| `memory/YYYY-MM-DD.md` | Daily memory logs |

### When to add workspace files vs other mechanisms

- **Permanent behavior rule** → Put it in `AGENTS.md` (standing order)
- **Periodic check** → Put it in `HEARTBEAT.md`
- **One-time startup task** → Put it in `BOOT.md`
- **Scheduled task at exact time** → Use a cron job
- **React to an event** → Use a hook or webhook

---

## Standing Orders

**What:** Persistent programs that grant the agent autonomous authority.
**When to use:** Any recurring responsibility the agent should own without being prompted.

Standing orders go in `AGENTS.md` (auto-injected every session). They define:
- **Scope** — what the agent is authorized to do
- **Triggers** — when to execute (schedule, event, condition)
- **Approval gates** — what needs human sign-off
- **Escalation rules** — when to stop and ask

### Why standing orders instead of skills

A skill tells the agent *how* to use a tool. A standing order tells it *what to
do and when*, with clear boundaries. Don't build a skill for "check inbox" — write
a standing order that says "you own inbox triage, here's your authority and limits."

### Template

```markdown
## Program: [Name]

**Authority:** [What the agent can do autonomously]
**Trigger:** [When — heartbeat, cron, event, on-demand]
**Approval gate:** [What needs human approval before acting]
**Escalation:** [When to stop and ask for help]

### Execution Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

### What NOT to Do
- [Boundary 1]
- [Boundary 2]
```

### Example: Mailroom Monitoring

```markdown
## Program: Mailroom Monitoring

**Authority:** React to broadcasts, claim applicable tasks, execute within role
**Trigger:** Every inbound mailroom message (passive) + heartbeat (catch-up)
**Approval gate:** None for reactions/claims. Escalate before executing cross-namespace changes.
**Escalation:** If task is outside your capabilities, or affects another agent's resources

### Execution Steps
1. On inbound broadcast → react with emoji to confirm receipt
2. On [Request] mentioning you → claim with 👀, execute, report [Done]
3. On heartbeat → check for unclaimed requests matching your role

### What NOT to Do
- Don't claim tasks outside your capabilities
- Don't execute requests that modify other agents' workspaces
- Don't reply to broadcasts in root — thread only
```

### Example: Post-Mutation Broadcasting

```markdown
## Program: Infrastructure Broadcasting

**Authority:** Send broadcasts to Mailroom after any mutation
**Trigger:** After every git push, config change, skill update, deployment change
**Approval gate:** None — broadcasting is mandatory and automatic
**Escalation:** Never — this is a mechanical side-effect, not a decision

### Execution Steps
1. Complete the mutation (git push, config write, etc.)
2. IMMEDIATELY send broadcast before next tool call
3. Format: `📡 [Infra Change] <what> — <impact>. No reply needed.`
4. One broadcast per logical change
```

### Prompting for standing orders

Tell your agent:
> "Add a standing order for [task]. You should [do X] every [when], with
> approval needed for [high-risk actions]. Escalate if [conditions]. Put it
> in AGENTS.md."

Or more specifically:
> "Create a Program in AGENTS.md for weekly report generation. Run it every
> Friday at 4pm via cron. Authority to pull metrics and generate the report.
> Approval gate on sending to external parties. Escalate if data sources
> are unavailable."

---

## Cron Jobs

**What:** Gateway-managed scheduled tasks with precise timing.
**When to use:** Exact-time schedules, isolated background work, one-shot reminders.

Cron runs *inside the Gateway* (not the model). Jobs persist across restarts.

### Session types

| Type | Config | Use case |
|------|--------|----------|
| **Main** | `sessionTarget: "main"` | Inject event into main session + heartbeat |
| **Isolated** | `sessionTarget: "isolated"` | Standalone agent turn, own session |
| **Current** | `sessionTarget: "current"` | Bind to the session where created |
| **Custom** | `sessionTarget: "session:my-id"` | Persistent named session across runs |

### Schedule types

| Kind | Config | Example |
|------|--------|---------|
| One-shot | `schedule.kind: "at"` | `"at": "2026-04-01T09:00:00+11:00"` |
| Interval | `schedule.kind: "every"` | `"everyMs": 3600000` (1 hour) |
| Cron expression | `schedule.kind: "cron"` | `"expr": "0 9 * * 1-5"` |

### Delivery modes

| Mode | Behavior |
|------|----------|
| `announce` (default for isolated) | Posts summary to chat channel |
| `webhook` | POSTs result to a URL |
| `none` | Silent, internal only |

### Examples

**Daily morning brief (isolated, announces to Matrix):**
```
cron add:
  name: "Morning brief"
  schedule: { kind: "cron", expr: "0 7 * * *", tz: "Australia/Melbourne" }
  sessionTarget: "isolated"
  payload: { kind: "agentTurn", message: "Generate today's briefing." }
  delivery: { mode: "announce", channel: "matrix", to: "<room_id>" }
```

**One-shot reminder in 20 minutes (main session):**
```
cron add:
  name: "Standup reminder"
  schedule: { kind: "at", at: "20m" }
  sessionTarget: "main"
  payload: { kind: "systemEvent", text: "Reminder: standup in 10 minutes" }
```

**Weekly review with persistent context (custom session):**
```
cron add:
  name: "Weekly review"
  schedule: { kind: "cron", expr: "0 9 * * 1", tz: "Australia/Melbourne" }
  sessionTarget: "session:weekly-review"
  payload: { kind: "agentTurn", message: "Continue the weekly review. Compare to last week." }
  delivery: { mode: "announce", channel: "matrix" }
```

### Prompting for cron

> "Set up a cron job to [task] every [schedule]. Run it isolated and announce
> the result to [channel]."

> "Remind me in 30 minutes to [thing]."

> "Create a recurring job every weekday at 9am Melbourne time to check [thing]
> and report back."

---

## Heartbeats

**What:** Periodic main-session check-ins at a regular interval.
**When to use:** Batching multiple routine checks, context-aware monitoring.

Heartbeats run in the **main session** (full conversational context).
Controlled by `HEARTBEAT.md` — if it's empty or comments-only, heartbeat
is a no-op (`HEARTBEAT_OK`).

### When to use heartbeat vs cron

| Need | Use |
|------|-----|
| Batch multiple checks together | Heartbeat |
| Need conversational context | Heartbeat |
| Exact timing matters | Cron |
| Task needs isolation | Cron |
| Different model/thinking level | Cron |
| One-shot reminder | Cron |
| Reduce API calls (combine checks) | Heartbeat |

### Example HEARTBEAT.md

```markdown
# Heartbeat Checklist

- Check mailroom for unclaimed requests matching my role
- If a background cron job posted results, review and follow up
- If idle for 8+ hours, do a brief check-in
```

### Prompting for heartbeat

> "Add [check] to your heartbeat checklist."

> "Remove the weather check from heartbeat — we don't need that."

> "Use heartbeat to batch your inbox + calendar + mailroom checks."

---

## Hooks

**What:** TypeScript scripts that run inside the Gateway on events.
**When to use:** Automated reactions to commands, messages, or lifecycle events.

Hooks fire on events like `/new`, `/reset`, message received/sent, gateway
startup, session compaction. They're code, not prompts — deterministic and fast.

### Available events

| Event | When it fires |
|-------|---------------|
| `command:new` | User issues `/new` |
| `command:reset` | User issues `/reset` |
| `command:stop` | User issues `/stop` |
| `message:received` | Inbound message from any channel |
| `message:sent` | Outbound message sent |
| `message:transcribed` | Audio transcription complete |
| `message:preprocessed` | All media/link processing done |
| `session:compact:before` | Before compaction |
| `session:compact:after` | After compaction |
| `agent:bootstrap` | Before workspace files injected |
| `gateway:startup` | Gateway starts, channels loaded |

### Bundled hooks

| Hook | What it does | Default |
|------|-------------|---------|
| `session-memory` | Saves session context to `memory/` on `/new` or `/reset` | Disabled |
| `bootstrap-extra-files` | Injects extra files during bootstrap | Disabled |
| `command-logger` | Logs commands to `~/.openclaw/logs/commands.log` | Disabled |
| `boot-md` | Runs `BOOT.md` on gateway startup | Disabled |

### Custom workspace hooks

Place in `<workspace>/hooks/<hook-name>/`:
```
hooks/
  my-hook/
    HOOK.md          # Metadata (YAML frontmatter)
    handler.ts       # TypeScript handler
```

Workspace hooks are disabled by default — enable via `openclaw hooks enable`.

### Hook vs skill vs standing order

| Mechanism | When to use |
|-----------|-------------|
| **Hook** | Deterministic, code-level reaction to an event (no LLM needed) |
| **Skill** | Teaching the agent how to use an external tool/API |
| **Standing order** | Granting the agent autonomous authority for a program |

### Example: Audit logging hook

```typescript
// hooks/audit-log/handler.ts
const handler = async (event) => {
  if (event.type === 'message' && event.action === 'sent') {
    const fs = await import('fs/promises');
    const line = `${event.timestamp.toISOString()} | ${event.context.to} | ${event.context.content?.slice(0, 100)}\n`;
    await fs.appendFile('/path/to/audit.log', line);
  }
};
export default handler;
```

### Prompting for hooks

Hooks are code, not agent prompts. To add one:
> "Create a workspace hook that [does X] when [event Y] fires. Put it in
> `hooks/<name>/` with HOOK.md and handler.ts."

To enable bundled hooks:
> "Enable the session-memory hook so context is saved on /new."

---

## Webhooks

**What:** HTTP endpoints that let external systems trigger agent work.
**When to use:** GitHub events, monitoring alerts, email notifications, CI/CD.

Webhooks expose `POST /hooks/wake` (system event) and `POST /hooks/agent`
(isolated agent run) on the Gateway.

### Endpoints

| Endpoint | What it does |
|----------|-------------|
| `POST /hooks/wake` | Enqueue system event in main session |
| `POST /hooks/agent` | Run isolated agent turn |
| `POST /hooks/<name>` | Custom mapped hook |

### When to use webhooks vs polling

| Approach | Use case |
|----------|----------|
| **Webhook** | External system can push events (GitHub, monitoring, email) |
| **Polling (heartbeat/cron)** | No push mechanism available, or need to batch checks |

### Example: GitHub webhook → agent

```bash
curl -X POST http://gateway:18789/hooks/agent \
  -H 'Authorization: Bearer $HOOKS_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "New PR opened: #42 - Fix auth bug. Review it.",
    "name": "GitHub",
    "deliver": true,
    "channel": "matrix",
    "to": "<room_id>"
  }'
```

### Config (openclaw.json)

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
  }
}
```

### Prompting for webhooks

Webhooks are infrastructure config, not agent prompts. Set them up via:
> "Enable webhooks in the gateway config with a token. I want to wire
> GitHub push events to trigger a code review."

---

## BOOT.md

**What:** A startup script that runs once when the gateway restarts.
**When to use:** Post-restart initialization, health checks, announcements.
**Requires:** `boot-md` hook enabled + internal hooks enabled.

### Example BOOT.md

```markdown
# Startup Tasks

- Check that all expected services are reachable
- Read today's memory file for context
- If there are pending mailroom requests for me, claim them
- Post a brief "I'm back online" to the mailroom if downtime was >10 minutes
```

### Prompting for BOOT.md

> "Add a startup check to BOOT.md that verifies [service] is reachable
> and reports to mailroom if it's not."

---

## Context Engine

**What:** Plugin that controls how the model's context window is managed.
**When to use:** Long-running sessions, better memory recall, smarter compaction.

The default `legacy` engine does simple summarization. Plugin engines (like
`lossless-claw`) can implement DAG summaries, vector retrieval, etc.

### Config

```json5
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw"  // or "legacy" (default)
    },
    entries: {
      "lossless-claw": { enabled: true }
    }
  }
}
```

This is infrastructure-level config, not something agents manage themselves.

---

## Broadcast Groups

**What:** Multiple agents process the same message simultaneously.
**When to use:** Specialized agent teams in one channel.
**Status:** Experimental (WhatsApp only currently).

### Config

```json5
{
  broadcast: {
    strategy: "parallel",
    "<peer_id>": ["agent1", "agent2", "agent3"]
  }
}
```

Each agent gets isolated sessions, separate context, separate tools.

---

## Multi-Agent Routing

**What:** Route inbound messages to different agents based on channel, peer, account.
**When to use:** Multiple agents sharing one Gateway (our cluster setup).

Routing is handled by `bindings` in config. Most-specific match wins:
1. Exact peer match
2. Parent peer (thread inheritance)
3. Guild + roles (Discord)
4. Account match
5. Channel-wide fallback
6. Default agent

This is infrastructure config managed in fluxclaw, not something agents
configure themselves.

---

## Decision Matrix: What to Use When

### "I want the agent to do X every day at 9am"
→ **Cron job** (isolated, with announce delivery)

### "I want the agent to check inbox periodically"
→ **Heartbeat** (add to HEARTBEAT.md, batches with other checks)

### "I want the agent to always follow this rule"
→ **Standing order** (add Program to AGENTS.md)

### "I want something to happen when a message arrives"
→ **Hook** (if deterministic/code) or **Standing order** (if needs LLM judgment)

### "I want GitHub/external events to trigger agent work"
→ **Webhook** (POST to /hooks/agent)

### "I want the agent to do setup work after restart"
→ **BOOT.md** (with boot-md hook enabled)

### "I want the agent to remember things across sessions"
→ **Workspace files** (MEMORY.md, memory/, AGENTS.md)

### "I want to teach the agent how to use an API"
→ **Skill** (SKILL.md + reference files)

### "I want the agent to react to /new or /reset"
→ **Hook** (command:new, command:reset events)

### "I want better context management for long sessions"
→ **Context engine plugin** (infrastructure config)

### "I want multiple agents to respond to the same message"
→ **Broadcast group** (infrastructure config)

---

## Quick Reference: How to Prompt Each Feature

| Feature | How to ask for it |
|---------|-------------------|
| Standing order | "Add a program to AGENTS.md for [task] with [scope/triggers/gates]" |
| Cron job | "Set up a cron to [task] every [schedule], announce to [channel]" |
| Reminder | "Remind me in [time] to [thing]" |
| Heartbeat check | "Add [check] to your heartbeat checklist" |
| Hook (bundled) | "Enable the [hook-name] hook" |
| Hook (custom) | "Create a workspace hook for [event] that [does X]" |
| Webhook | "Enable webhooks and wire [external system] to trigger [task]" |
| BOOT.md task | "Add [task] to your startup checklist" |
| Workspace note | "Remember [thing] in your memory/tools/agents file" |
| Skill | "Create a skill for [external API/tool] with [instructions]" |

---

*This file lives in the agent template. All agents should have it.
Update it when OpenClaw adds new features.*
