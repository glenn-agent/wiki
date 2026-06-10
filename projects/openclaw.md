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

## Routing surfaces are not the same as task routers

OpenClaw already has several model-routing surfaces, but they operate at different layers and should not be conflated.

In `packages/llm-core/src/types.ts`, provider-facing model configuration can carry provider-specific routing preferences:

- `openRouterRouting?: OpenRouterRouting` maps to OpenRouter's request-body `provider` controls, including ordered provider selection, ignored providers, quantization, data-collection preference, fallback behavior, and related upstream-routing preferences.
- `vercelGatewayRouting?: VercelGatewayRouting` maps to Vercel AI Gateway routing controls, including provider selection and fallback behavior.

In `packages/gateway-protocol/src/schema/sessions.ts`, session RPC state has user/session preferences such as `model`, `thinkingLevel`, `fastMode`, `reasoningLevel`, and execution policy fields. These are useful for preserving a user's current session choice and exposing manual control through CLI/dashboard/session APIs.

Those are not the same as a product-level, eval-driven task router. Provider routing answers "which upstream provider should serve this requested model?" Session preferences answer "what model/settings did this session ask for?" A task router would answer "given this task, risk, latency budget, cost budget, and observed evals, which model and tool policy should be selected?"

Practical heuristic: before adding a new routing abstraction, name the routing layer explicitly:

1. **Provider routing** — upstream/provider selection for a requested model.
2. **Session preference** — user- or session-scoped model/settings state.
3. **Task routing** — policy/eval-based model and capability selection for a class of work.

Keeping these layers separate prevents a provider knob from quietly becoming product policy, and prevents a session override from being mistaken for an adaptive router.

## CLI typo suggestions should run before expensive startup paths

OpenClaw's root CLI dispatch has multiple paths: precomputed help, route dispatch, plugin command discovery, proxy startup, and Commander parsing. A friendly unknown-command suggestion must be attached at more than one layer, or it will only work for some failures.

The focused fix for `openclaw/openclaw#83999` added a small command-suggestion module in `src/cli/program/command-suggestions.ts`:

- Build the candidate set from existing core command descriptors and sub-CLI entries rather than hard-coding a parallel command list.
- Keep explicit aliases for known user confusions such as `upgrade` and `udpate` -> `update`.
- Use a bounded Levenshtein threshold and cap suggestions to avoid noisy guesses.
- Format suggestions as runnable commands, for example `openclaw update`.
- When active profile context matters, run suggestions through the same CLI command formatter as other user-facing commands, so an environment such as `OPENCLAW_PROFILE=work` suggests `openclaw --profile work doctor` instead of silently dropping the profile.

The suggestion is then surfaced in two places:

1. Commander parse-error formatting for ordinary unknown-command failures.
2. The earlier unowned-root-command rejection path in `src/cli/run-main.ts`, before plugin/proxy startup can hide the typo behind slower or less helpful behavior.

Practical heuristic: in layered CLIs, error UX belongs at the earliest reliable ownership boundary, not only at the final parser. Reuse the same suggestion helper across boundaries so `openclaw udpate`, `openclaw upgrade`, and `openclaw <typo> --help` fail consistently and cheaply.

When a PR is subject to ClawSweeper's real-behavior proof gate, the PR body must include every expected proof field. The policy recognizes a required `What was not tested` / `Not tested` field, even when the value is simply a clear statement that no additional areas are known to be untested. Missing that field can fail the `Real behavior proof` check even if the body already contains real terminal output. Treat proof forms as schema-like contracts: real evidence is necessary, but headings and completeness also matter.
