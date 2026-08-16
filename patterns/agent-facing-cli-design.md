# Agent-Facing CLI Design

Agent-facing CLIs should optimize for predictable automation before interactive convenience.

A 2026-08-16 trend scan surfaced `cli-for-agent` style guidance from the `cursor/plugins` ecosystem. The reusable lesson for OpenClaw and Glenn-Agent is that a CLI used by agents is part of the tool contract, not just a human terminal interface.

## Pattern

1. **Prefer explicit flags over hidden prompts.** Agents need stable, inspectable commands. Interactive questions should have non-interactive equivalents.
2. **Expose dry-run and diff modes.** Before mutating files, accounts, repos, schedules, or runtime config, provide a way to preview the exact intended change.
3. **Make operations idempotent where possible.** Re-running a safe setup, sync, or check command should converge instead of duplicating resources.
4. **Use structured output for machine paths.** Human text is useful, but agents need JSON or another stable format for IDs, statuses, errors, and retry hints.
5. **Return precise exit codes.** Distinguish success, validation failure, auth failure, missing dependency, partial success, and retryable network failure when practical.
6. **Write errors as repair instructions.** A good agent-facing error says what failed, where, whether state changed, and the smallest next command or decision needed.
7. **Keep examples copy-pasteable.** Examples should include required flags, working-directory assumptions, and safe defaults.
8. **Name external side effects.** Commands that push, publish, notify, delete, spend money, or touch external accounts should say so in help text and support confirmation controls.

## Practical checklist

Before relying on a CLI in an autonomous routine, ask:

```text
Can I run it non-interactively?
Can I preview mutations?
Can I parse the result without scraping prose?
Can I safely retry it?
Does failure tell me whether anything changed?
Does the help text reveal external side effects?
```

If the answer is no, wrap the command with extra guardrails or keep it inside an operator-approved path.

## Why it matters

Long-horizon agents fail less often when their tools behave like protocols. Predictable flags, dry runs, idempotency, structured output, and clear errors turn CLIs from fragile terminal conversations into reviewable automation surfaces.
