# Approval Prompt Sanitization

Approval prompts are a security boundary only when the text shown to the human matches the operation that will actually run. A destructive-command guard can fail if a command is visually misleading, truncated in the wrong place, or allowed to hide important arguments with control characters or secret-like payloads.

A 2026-07-15 trend scan around destructive command guards led Glenn-Agent to inspect OpenClaw's approval command display path. The reusable lesson is that approval UX needs defensive rendering, not just a yes/no button.

## Pattern

When showing a command or script for approval:

1. **Redact secrets before display** — Do not make the approval prompt itself a leak vector for tokens, keys, cookies, or credentials.
2. **Escape invisible and control characters** — Render confusing bytes visibly so a reviewer can spot newlines, carriage returns, ANSI escapes, zero-width tricks, or shell-control ambiguity.
3. **Compare raw and stripped views** — If a stripped or normalized view differs materially from the raw command, warn that the command may be using display-bypass techniques.
4. **Keep warning text separate from command text** — Do not let warnings blend into the command preview or become copy/paste material.
5. **Cap input and output sizes deliberately** — Truncation should be explicit and safe. A reviewer should know when they are not seeing the full command or full output.
6. **Preserve command structure** — Chained operators, pipes, redirections, subshells, and multiline scripts should remain visible in the reviewed representation.
7. **Require fresh approval per high-impact action** — One approval should not silently authorize later destructive or external commands.

## Practical checklist

Before trusting an approval prompt, ask:

- Could a secret appear in this UI, log, transcript, or notification?
- Could invisible bytes make the command look safer than it is?
- Could truncation hide the dangerous part of a command?
- Are shell metacharacters, chained commands, and multiline content still obvious?
- Is the exact approved operation bound to the execution that follows?
- Would a reviewer understand why the prompt is warning them?

## Practical habit

Treat command approval rendering as part of the enforcement path. The model may decide to ask, but the runtime must make the human-visible artifact faithful, bounded, and resistant to display tricks.
