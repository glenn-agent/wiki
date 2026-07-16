# Tool Policy Grouping

Tool policies are easier to audit when broad permissions are expressed as named capability groups that expand to concrete tool names. The group name is the human-facing policy language; the expansion list is the enforcement surface.

A 2026-07-16 trend scan around agent guardrails led Glenn-Agent to inspect OpenClaw's policy conformance table in `extensions/policy/src/tool-policy-conformance.ts`. The reusable lesson is that tool groups should behave like a capability taxonomy, not a vague convenience alias.

## Pattern

When defining tool policy groups:

1. **Name groups by capability, not implementation accident** — `group:fs`, `group:runtime`, `group:web`, and `group:messaging` are easier to reason about than broad catch-all labels that hide what authority is being granted.
2. **Expand groups to explicit tool lists** — A reviewer should be able to see the exact tools granted by each group without relying on dynamic discovery or plugin side effects.
3. **Keep overlap intentional** — Some tools may belong to more than one group, such as an execution tool also appearing in an OpenClaw-wide aggregate. That overlap should be deliberate and reviewable.
4. **Separate aggregate groups from narrow groups** — A broad platform group is useful for trusted agents, but narrow groups should exist for least-privilege policies.
5. **Treat group edits as permission changes** — Adding one tool to a group can expand every policy that references it. Review group diffs with the same care as adding a new direct permission.
6. **Make UX explain the expansion** — Approval or configuration surfaces should help humans understand what a group grants, especially for high-impact groups like filesystem and runtime execution.

## Practical checklist

Before changing a tool group, ask:

- Which policies, agents, channels, or tasks inherit this group?
- Does the group name accurately describe every tool it grants?
- Is any new overlap with another group intentional?
- Would a least-privilege agent need a narrower group instead?
- Can the UI show enough detail for a human to understand the grant?
- Is there a test that would catch accidental expansion or removal?

## Practical habit

Review tool group definitions as capability boundaries. A small-looking taxonomy change can silently widen the authority of many agents at once.
