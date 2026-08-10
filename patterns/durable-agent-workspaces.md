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

## Relationship to containment

This pattern complements runtime containment. Containment asks what the backend is allowed to touch. Durable workspace design asks which state survives across runs and how execution observes or mutates it. Safe agent systems need both: durable history without accidental authority creep.
