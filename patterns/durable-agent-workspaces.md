# Durable Agent Workspaces

Agent runtimes are easier to reason about when workspace state and execution backends are separate contracts.

A 2026-08-10 trend scan used Cloudflare Computer as an architecture reference. Its preview design is useful because it names three separable ideas:

- an authoritative workspace filesystem backed by durable state;
- one runtime entry point for executing work against that workspace;
- pluggable backends with different isolation and capability tradeoffs.

For OpenClaw/NemoClaw practice, the reusable lesson is not to copy the implementation. It is to make the runtime boundary legible: what state is durable, which backend is executing, what authority that backend has, and how costs or recursion are controlled.

## Pattern

1. **Make workspace state explicit.** A task should know which files, artifacts, and memory surfaces are authoritative instead of depending on ambient process state.
2. **Keep execution pluggable but named.** Shell, container, browser, JavaScript isolate, Python worker, or remote machine backends should be selected through stable identifiers, not hidden behind one vague `run` path.
3. **Separate filesystem durability from compute authority.** A durable workspace does not automatically imply permission to run arbitrary binaries, use the network, or access credentials.
4. **Connect backends lazily.** Do not initialize heavy or high-authority execution surfaces until a task needs them.
5. **Record backend choice in evidence.** Test logs, PR notes, and task records should say where code ran, especially when local preflight and remote validation have different authority or fidelity.
6. **Add cost and recursion guardrails before expanding autonomy.** Long-running agents need budgets, depth limits, cancellation, and audit trails before they can safely loop over workspace mutations and execution.

## Practical heuristic

When adding a new agent execution capability, describe it as:

```text
workspace: <durable state boundary>
backend: <execution implementation>
authority: <filesystem/network/credential/account access>
evidence: <how results are recorded and replayed>
guardrails: <budget, recursion, timeout, cancellation, approval>
```

If any field is fuzzy, the runtime is not ready for broader autonomy yet.

## Canonical history, projected context

A long-running agent should treat canonical execution history as the source of truth and prompt context as a projection of that history.

Apache Maka's backend architecture note reinforced this distinction during a 2026-08-27 radar review. The reusable lesson is not specific to Maka: if an agent workspace preserves events, files, decisions, approvals, and task state in durable records, then each model turn can receive a selected view of that record without pretending the selected view is the whole system state.

Useful design consequences:

- Store execution events and important state transitions outside the model context window.
- Make summaries and prompts reproducible projections from durable records, not the only place where task truth lives.
- Preserve enough provenance to answer which command, tool call, approval, file diff, or external event produced a state change.
- Allow later compaction or handoff to rebuild context from history rather than trusting memory of a prior chat turn.
- Keep projections scoped: a contribution task, daily radar, writeback review, and PR follow-up may need different context slices over the same underlying history.

Practical heuristic: if losing the current prompt would lose the only evidence for what happened, the workspace has not made history durable enough.

## Relationship to containment

This pattern complements runtime containment. Containment asks what the backend is allowed to touch. Durable workspace design asks which state survives across runs and how execution observes or mutates it. Safe agent systems need both: durable history without accidental authority creep.
