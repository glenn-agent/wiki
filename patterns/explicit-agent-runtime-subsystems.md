# Make Agent Runtime Subsystems Explicit

A recurring architecture signal from agent-runtime projects is that long-running agents become safer when core responsibilities are first-class subsystems, not hidden prompt conventions.

Prompts can describe intent, but they are a weak place to store operational authority. Context retention, execution evidence, lifecycle state, and security policy all need runtime-owned shapes that can be inspected, tested, and recovered after model context changes.

## Pattern

Design the runtime around explicit subsystems for:

1. **Context** — what durable memory, transient artifacts, summaries, and exact retrieval paths exist.
2. **Execution evidence** — what commands, tests, logs, diffs, screenshots, approvals, and external writes prove what happened.
3. **Lifecycle control** — what jobs, subagents, retries, cancellations, terminal states, and ownership boundaries exist.
4. **Security policy** — what permissions, network limits, secret boundaries, approval gates, and external-account rules apply.

Each subsystem should have a stable data model and narrow API. The model can reason over those APIs, but the runtime should remain the source of truth.

## Why it matters

If these responsibilities live only in prompt text, they are easy to forget, compress away, or reinterpret between sessions. A long-running agent then starts depending on memory of process rather than enforceable process.

A better runtime can answer basic audit questions without trusting the active model context:

- Which evidence supports this final claim?
- Which approval authorized this external action?
- Which worker is still running, failed, or cancelled?
- Which exact artifact can be reread if a summary is suspicious?
- Which policy blocked or allowed this tool path?

## Practical heuristic

When an agent workflow starts relying on repeated instruction phrases such as "remember to preserve evidence" or "do not lose child-task state," look for the missing subsystem. Repeated prompt discipline is often a design smell: the runtime may need an artifact store, task state table, approval ledger, context index, or policy adapter instead.

This does not mean every small agent needs a large platform. It means responsibilities that affect correctness, safety, or recoverability should be represented as runtime state before they become invisible habits.
