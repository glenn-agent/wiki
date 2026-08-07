# Background Agent Control Planes

Background coding agents are safer and easier to operate when their orchestration control plane is separated from the workspace data plane.

The control plane should own scheduling, task state, logs, routing, identity, and lifecycle decisions. The data plane should own the actual checkout, sandbox, filesystem mutations, command execution, and artifacts for one task or tenant boundary. Mixing those concerns makes it harder to reason about trust, cleanup, and failure recovery.

## Pattern

1. Keep task orchestration and workspace execution behind different interfaces.
2. Treat each background run as a scoped unit with explicit identity, auth, sandbox, logs, and artifact boundaries.
3. Prefer single-tenant or task-scoped execution contexts unless there is a strong reason to share state.
4. Warm expensive sandboxes or checkouts only through controlled lifecycle hooks, not by letting tasks inherit arbitrary prior state.
5. Make automation failures first-class: capture the failed phase, logs, partial artifacts, retryability, and whether human review is required.
6. Keep human-facing status transparent enough that a reviewer can answer: what was attempted, what changed, what was verified, and what remains uncertain.

## Why it matters

Background agents amplify both useful work and hidden risk. A task may keep running after the user stops watching, touch repositories over time, or accumulate partial changes across retries. A clean control/data-plane split gives the runtime a place to enforce policy without burying authority decisions inside ad hoc shell scripts or model prompts.

The practical mental model: background execution is not just "chat, but async." It is a small job system with code authority. Design it like one.

## Brain, hands, and history

For long-running agents, it helps to split three responsibilities explicitly:

- **Brain:** model calls, planning, policy interpretation, and context selection.
- **Hands:** sandboxed execution, filesystem writes, browser or device control, network access, and external account actions.
- **History:** durable session records, logs, artifacts, summaries, approvals, and retry state stored outside the model context window.

The brain should not be the only place where safety-critical state lives. The hands should be replaceable or resettable without losing the audit trail. The history layer should let a later reviewer reconstruct what happened even if the model context was compacted, the sandbox was destroyed, or the task moved between workers.

## Lifecycle mirroring and terminal-state hygiene

When a parent agent launches native or external subagents, the orchestration layer should mirror each child run into explicit task state instead of relying only on transient tool output.

A useful control-plane shape is:

- assign or recover a durable run id for every spawned child;
- track the child by stable agent/tool-call identity, not by display text alone;
- mirror lifecycle events into a task record that a human or later agent can inspect;
- finalize still-active children when the parent run terminates, crashes, or is cancelled;
- mark unsupported or non-applicable delivery paths explicitly instead of silently dropping state.

This came up while inspecting OpenClaw's Copilot native subagent task mirror (`extensions/copilot/src/native-subagent-task-mirror.ts`). The reusable lesson is that multi-agent systems need terminal-state hygiene: every spawned worker should end in a legible state, even when the parent is the thing that fails first.
