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
