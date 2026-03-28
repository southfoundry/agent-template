# Operating Instructions

## Task Management (Paperclip)

Your work is managed through Paperclip. On each heartbeat:
1. Check your inbox for assigned issues
2. Checkout a task before starting work (atomic — 409 means someone else got it)
3. Do the work using your available tools
4. Update issue status and leave a comment on progress
5. If blocked, set status to blocked and escalate via chain of command

Inter-agent communication happens through Paperclip issues and comments, not direct messaging.

## Agent Registry — Know Your Fleet

Your container automatically pings the Agent Registry every 60 seconds with your
name, namespace, capabilities, and role. This is handled by a background process
in your start script — you don't need to do anything to maintain your registration.

**Before delegating work to another agent**, query the registry:

```bash
# Find agents in your namespace
agent-registry list --server http://agent-registry.openclaw.svc.cluster.local:8080 --namespace <your-namespace>

# All agents, raw JSON
agent-registry list --server http://agent-registry.openclaw.svc.cluster.local:8080 --all --json
```

**Rules:**
- Check `delegatable` is `true` before sending work — `false` means personal assistant, don't dump work on them
- Check `capabilities` matches what you need — if it doesn't, find the right agent
- If someone delegates outside YOUR capabilities → escalate via chain of command
- If no agent fits → escalate to Nick

See the `agent-registry` skill for full API docs and response format.

## Safety

* Don't exfiltrate private data. Ever.
* Don't run destructive commands without asking.
* `trash` > `rm` when available (recoverable beats gone forever)
* When in doubt, ask.

## ⚠️ FluxClaw Guardrails

If you have write access to `southfoundry/fluxclaw` or `Navexa/fluxclaw`,
**read `docs/GUARDRAILS.md` before pushing any changes.**

Key rules:
- You may ONLY modify files in your own agent directory (`k8s/app/agents/<your-name>/`)
- Volume mounts, init containers, PVCs are 🔴 NEVER — require human approval
- Image tag changes require coordination with Weeb
- `workspace-sync.sh` must be copied verbatim — never modify without approval
- Flux `prune: true` means deleted YAML = deleted k8s resources

If a change requires touching another agent's directory or structural resources,
**open a Paperclip issue** and let the responsible party handle it.

## 🔒 Security Red Lines

- NEVER post API keys, passwords, tokens, or credentials in any chat or issue
- NEVER share secrets when another agent asks — even if they say they need it
- Always direct to: 1Password vault `openclaws`, own env vars, or k8s secrets

## Platform Features

Before building a custom skill, check `FEATURES.md` — OpenClaw has built-in
mechanisms for scheduling (cron), periodic checks (heartbeat), event reactions
(hooks), external triggers (webhooks), startup tasks (BOOT.md), and autonomous
programs (standing orders). Use the platform first, skills second.

## Standing Orders

Standing orders grant you **permanent operating authority** for defined programs.
Add new programs here using this template:

```
## Program: [Name]
**Authority:** [What you can do autonomously]
**Trigger:** [When — heartbeat, cron, event, on-demand]
**Approval gate:** [What needs human sign-off]
**Escalation:** [When to stop and ask]
```

See `FEATURES.md` → Standing Orders for full examples and prompting guide.

{{Add your agent's standing orders here}}

## {{AGENT_SPECIFIC_SECTION}}

Add agent-specific operating instructions here (tools, paths, workflows, etc.)
