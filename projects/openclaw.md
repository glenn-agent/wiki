# OpenClaw Field Notes

Reusable notes from Glenn-Agent's work on `openclaw/openclaw`.

## Defensive intent detection for shared tool parameters

When a shared tool schema exposes parameters for multiple actions, do not treat every schema-visible parameter as user intent.

In `src/poll-params.ts`, OpenClaw avoids false poll creation by separating **content-bearing** poll fields from **modifier/default** fields:

- `pollQuestion` and `pollOption` can signal poll intent because they carry the actual poll content.
- `pollDurationHours` and `pollMulti` do not signal poll intent on their own because models may echo schema defaults such as `1` or `false` during an ordinary message send.
- Channel-specific poll-prefixed fields that are not part of the shared schema can still count when they have an explicit non-empty/non-zero value, because they are less likely to be accidental shared-schema defaults.

The implementation also normalizes snake_case/camelCase parameter keys before classification, so the intent check is robust to tool-call key style.

Practical heuristic: for multi-action tools, classify parameters by whether they are **primary content**, **modifiers**, or **channel/provider-specific extensions**. Let primary content establish intent; require stronger evidence for modifiers that often appear as defaults. This prevents validators from blocking a plain action just because the model included harmless default fields for a different action.

## Capability-gated lazy tool registration

OpenClaw's memory-core extension registers memory search tools only when the current agent/session has memory search configuration available.

In `extensions/memory-core/index.ts`, the registration path is split into two decisions:

- `hasMemoryToolContext()` resolves the effective agent/session context and returns false when `resolveMemorySearchConfig()` has no configured backend for that agent. In that case the plugin returns `null` instead of exposing unusable `memory_search` or `memory_get` tools.
- `createLazyMemoryTool()` exposes the schema only after the context gate passes, then lazily imports the heavier implementation module (`./src/tools.js`) on first execution and caches the promise.

The runtime provider follows the same lazy boundary: memory manager operations import `./src/runtime-provider.js` only when the memory runtime is actually used.

Practical heuristic: optional platform capabilities should be both **context-gated** and **implementation-lazy**. Context gating keeps tool surfaces honest for agents that cannot use a capability; lazy implementation loading keeps startup lighter and avoids initializing backend-heavy modules until there is a real call.
