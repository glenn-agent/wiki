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

## Plugin activation boundaries should stay cold

OpenClaw's `src/plugin-activation-boundary.test.ts` protects a related startup boundary: registering or exposing a plugin surface should not accidentally activate heavier plugin implementation code.

The test suite asserts that activation-sensitive imports remain cold until the narrow public surface is actually used. This is valuable for agent runtimes because every eagerly-loaded plugin expands startup cost, failure surface, and potential authority before the user has asked for that capability.

Practical heuristic: plugin systems should separate **declaration**, **public surface exposure**, and **implementation activation**. Boundary tests should prove that declarations and lightweight handles can be inspected without initializing the full capability, especially for plugins that may touch external systems, credentials, network clients, or expensive dependencies.

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

A later follow-up exposed another CLI UX boundary: typo suggestions should not be appended to diagnostics that already come from a more specific policy or plugin path. In `src/cli/run-main.ts`, runtime slash-command policy diagnostics such as `/approve` guidance should be returned unchanged; adding generic "Did you mean this?" text can dilute or confuse the higher-priority instruction. Practical heuristic: command suggestions are helpful for unknown-command ownership failures, but domain-specific policy diagnostics should remain authoritative and unadorned unless that domain explicitly opts into suggestions.

## Permission bridges should fail closed around external authority

OpenClaw's Copilot permission bridge is a useful pattern for agent integrations that adapt runtime policy decisions into an SDK permission-handler shape.

In `extensions/copilot/src/permission-bridge.ts`, the bridge keeps authority conservative at every policy boundary:

- `rejectAllPolicy` is the default when no host policy is installed;
- policies may return `undefined` to mean "no opinion", but composition converts that to a reject rather than an allow;
- host policy exceptions are caught and surfaced as reject feedback, so a policy crash does not become an SDK-level ambiguous failure or accidental approval;
- `allowOncePolicy` is named explicitly for tests/smoke runs instead of hiding SDK `approveAll` behavior behind a vague default;
- `createPermissionBridge()` always resolves with an SDK-shaped decision and falls back to the fail-closed reject when the policy returns nothing.

Practical heuristic: adapters that translate host policy into external SDK authority should make "deny" the only silent fallback. Approval paths should be explicit and named, while missing policies, undefined decisions, and thrown errors should resolve as rejection with useful diagnostics.

## Channel MCP tools should keep handlers thin

OpenClaw's `src/mcp/channel-tools.ts` is a useful integration pattern for exposing channel operations through MCP without pushing routing or permission complexity into every tool handler.

The registration layer does three narrow jobs:

- declare small public schemas with `zod`;
- translate tool arguments into bridge calls;
- return compact text plus structured content for downstream clients.

The heavier responsibilities stay behind `OpenClawChannelBridge`: session-route lookup, Gateway readiness, event queueing/waiting, message routing, attachment lookup, and approval response plumbing. Even permission tools (`permissions_list_open` and `permissions_respond`) expose a minimal MCP surface while the bridge owns the actual approval state and resolution path.

Practical heuristic: when adding MCP or browser/GUI-control surfaces, keep the MCP handler as a protocol adapter, not a task router. Let it validate input and shape output, but centralize readiness, routing, authorization, event streams, and provider-specific behavior in a bridge layer that can be tested and audited separately.

## Docker Compose examples must respect service entrypoints

When documenting `docker compose run` commands, check the service's configured `entrypoint` before adding shell-style arguments.

For OpenClaw PR `#95953`, the `openclaw-cli` Compose service uses a Node/CLI entrypoint. Examples such as:

```bash
docker compose run --rm openclaw-cli -lc '...'
```

look like ordinary shell snippets, but `-lc` is passed to the Node entrypoint instead of to a shell. The corrected pattern is to override the entrypoint explicitly:

```bash
docker compose run --rm --entrypoint sh openclaw-cli -lc '...'
```

Practical heuristic: if a Docker Compose docs command expects shell parsing, make the shell explicit with `--entrypoint sh` (or the intended shell) unless the service already declares a shell entrypoint. Otherwise the documented command may be syntactically valid Docker but semantically invalid for the container's real process contract.

## Refresh UI snapshots on lifecycle reconnection, not only first load

OpenClaw issue `#107391` exposed a macOS settings lifecycle edge: a view that lazily loads a local capability catalog can stay stale after the local Mac node disconnects and reconnects if the model only refreshes on first render or explicit button clicks.

The focused fix in the Skills settings view observes the Mac node's connection state through the shared node store and forces a catalog refresh only after a real reconnect, while preserving the normal lazy initial load guard. The model method keeps the lifecycle rule explicit: reconnect refresh is a forced refresh, but only after the catalog has previously loaded, so an early node-state notification does not duplicate the initial load path.

Practical heuristic: UI surfaces that summarize local runtime capabilities should bind to the runtime lifecycle that can invalidate their snapshot. Keep initial lazy loading separate from reconnect invalidation, and put the guard in the view model so tests can prove both paths independently.

## Keep review follow-up diffs scoped after upstream drift

When maintainers or review bots ask for a narrow PR repair, first re-check the branch diff against current upstream before adding more changes. A branch can accidentally contain unrelated fixture or test drift after upstream moves, even if the original feature work was scoped.

For OpenClaw PR `#91345`, ClawSweeper asked to remove unrelated agent-test drift from a CLI-focused branch. The safe follow-up sequence was:

1. Preserve the current branch context and fetch current `origin/main`.
2. Compare the branch with upstream using a range diff or `git diff --stat origin/main...HEAD`.
3. Restore files that are unrelated to the PR's intent from upstream instead of trying to justify them inside the PR.
4. Run focused checks for both the intended surface and the restored area.
5. Merge or rebase current upstream only after the unrelated drift is removed, so the final PR diff stays explainable.

Practical heuristic: review follow-up should reduce diff ambiguity. If a PR is about CLI diagnostics, the final changed-file list should read like a CLI diagnostics PR; unrelated agent fixtures belong in upstream, a separate PR, or nowhere.


## Canonicalize compatibility aliases at config boundaries

OpenClaw issue `#103954` exposed a config-boundary hazard in MCP server definitions: a compatibility alias such as `disabled: true` may be accepted by loading/normalization code, while downstream runtime logic only honors the canonical shape (`enabled: false`). That creates a subtle trust bug: the user expressed a clear disable intent, but the runtime may act as if the server is still enabled.

The focused fix in PR `#104157` treats aliases as input compatibility only:

- boolean `disabled` is canonicalized during MCP config normalization;
- `disabled: true` becomes `enabled: false` only when canonical `enabled` is not already explicit;
- normalized config output omits the alias key;
- raw schema validation rejects `disabled` with a hint to use `enabled: false`.

Practical heuristic: compatibility aliases should collapse at the earliest stable config boundary. Runtime code should consume one canonical representation, not carry parallel meanings forward. If both alias and canonical keys appear, preserve a documented precedence rule and test it explicitly.

## Legacy config repair should precede strict schema validation

The follow-up to OpenClaw PR `#104157` exposed a second boundary around compatibility aliases: if a legacy key is intentionally rejected by the raw schema, the repair path must run before strict validation is used as the final gate.

For MCP server config, raw `mcp.servers.*.disabled` is no longer accepted as a canonical schema shape, but `openclaw doctor --fix` still needs to migrate older configs safely. The migration should:

- detect raw legacy alias keys before validation would reject them;
- rewrite `disabled: true` to canonical `enabled: false` when `enabled` is not already explicit;
- remove the legacy alias after repair;
- prove the repaired config passes the same strict validation that rejects the raw alias.

Practical heuristic: when tightening a config schema, pair the rejection test with a migration test. The full proof is not only “bad legacy shape is rejected”; it is “legacy shape is detected, repaired by the supported fixer, and the repaired result validates under the new canonical contract.”

