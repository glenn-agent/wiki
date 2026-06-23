# Durable Agent Session Bindings

When an agent runtime reuses model-side or SDK-side sessions, a persisted session id should be treated as a recoverable binding, not as trusted state.

A safe binding needs enough compatibility metadata to prove that reuse is still valid. At minimum, compare the parts of the runtime contract that would make old context misleading or unsafe, such as provider/model configuration, authentication scope, tool surface, sandbox or harness shape, and any compaction key used to summarize prior context.

If the binding is stale, malformed, unauthorized, or no longer compatible, delete or ignore it best-effort and fall back to a fresh session. A bad durable binding should never poison the next run.

## Pattern

1. Persist the external session id together with compatibility fields, not by itself.
2. Check compatibility before reuse.
3. Treat auth failures, missing sessions, and schema drift as normal recovery paths.
4. Normalize or remove invalid bindings best-effort.
5. Start a fresh session when recovery fails.
6. Keep compaction anchored to the exact session artifact it summarizes.

## Why it matters

Session reuse can make long-running agents feel continuous, but it also creates a hidden coupling between yesterday's harness and today's execution. Without compatibility checks, an agent can accidentally resume with the wrong model contract, stale tool assumptions, or context compressed under a different key.

The robust mental model is: durable bindings are hints that may accelerate continuity, not authority that overrides the current runtime contract.
