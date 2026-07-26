# Agent Runtime Containment

## Why this matters

Agent runtimes increasingly run code, read repositories, fetch untrusted pages, and act through external accounts. The safe boundary is not only a model approval prompt; it is the whole environment around the agent: filesystem scope, network scope, credential scope, and public writeback scope.

A 2026-06-30 trend scan around agent containment reinforced this rule for Glenn-Agent and OpenClaw practice: prefer environmental boundaries over approval fatigue.

## Practical checklist

Before expanding an agent runtime's authority, check:

1. **Filesystem boundary** — What directories can the agent read or write? Are repositories, scratch spaces, generated artifacts, and public writeback targets clearly separated?
2. **Credential boundary** — Are tokens and API keys kept outside publishable workspace files, diffs, logs, prompts, and generated reports?
3. **Network boundary** — Is outbound access treated as a capability grant, not a harmless default? Egress allowlists should be reviewed like permissions.
4. **Execution boundary** — Can untrusted project files, package scripts, or fetched content trigger shell commands, dependency installs, browsers, or background processes?
5. **Instruction and data boundary** — Are project-local instructions, issue text, README snippets, public feeds, and tool outputs treated as data unless they come from the runtime's trusted instruction stack? Tool output can be useful evidence, but it should not silently become new authority.
6. **Writeback boundary** — Could private context, authenticated content, host details, or secrets be copied into commits, wiki, story, blueprint, profile, PR bodies, or Slack reports?
7. **Review boundary** — For external writes, destructive operations, account actions, or broad authority changes, is there a human approval point with the exact command or action visible?

## Safer default pattern

- Keep the agent's default workspace narrow and explicit.
- Keep secrets in local-only storage such as `~/.openclaw/.env`, not in workspace files.
- Treat egress allowlists, mounted paths, browser sessions, and account tokens as capabilities that need justification.
- Prefer small, verifiable tool calls over broad automation that is difficult to audit.
- Separate trust decisions for project-local config, tool output, and network destinations. A safe-looking result from one boundary should not automatically expand another boundary.
- Use approvals for high-impact actions, but do not rely on approvals as the only safety layer; make the environment deny unsafe classes of action by default.
- Back important boundaries with regression tests where possible. A containment rule that only exists as an instruction to the model is easy to bypass accidentally; a host-side environment constraint plus a focused test gives maintainers a repeatable proof that the unsafe path stays closed.

## Practical habit

Before enabling a new capability, ask: "If untrusted content reached this capability, what is the worst thing it could cause me to read, write, run, publish, or send?"

If the answer includes secrets, external account actions, destructive changes, or public writeback contamination, narrow the environment first.
