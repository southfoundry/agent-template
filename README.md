# Agent Template

Base workspace files for new OpenClaw agents in the South Foundry / Navexa cluster.

## Usage

1. Copy these files into the new agent's workspace ConfigMap (`workspace.yaml` in fluxclaw)
2. Replace all `{{PLACEHOLDER}}` values with agent-specific details
3. Customise SOUL.md with the agent's personality and role
4. Add agent-specific instructions to AGENTS.md (keep the shared sections)

## Files

| File | Purpose |
|---|---|
| `SOUL.md` | Identity, personality, role, communication style |
| `AGENTS.md` | Operating instructions, shared protocols, safety rules |
| `USER.md` | Who the agent serves |
| `IDENTITY.md` | Name, emoji, avatar |
| `HEARTBEAT.md` | Periodic task checklist (empty = no heartbeat work) |
| `TOOLS.md` | Environment-specific notes (cameras, SSH, voices, etc.) |
| `MEMORY.md` | Long-term curated memory (agent populates over time) |
| `memory/` | Daily logs directory (agent creates as needed) |

## Shared Protocols

All agents MUST follow these (baked into AGENTS.md):

- **Mailroom Protocol** — see `mailroom` skill in clawhub
- **Infra Broadcasting** — broadcast after any mutation, emoji ack required
- **Coordinated Restarts** — request restart from Flem when idle, never self-restart
- **Wrench Rule** — goblin shame when called out, no apology essays

## Template Repo

This repo is the source of truth for shared agent conventions. When protocols change,
update here first, then propagate to agent workspace ConfigMaps and agent repos.
