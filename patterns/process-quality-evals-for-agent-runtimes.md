# Process Quality Evals for Agent Runtimes

Final-answer correctness is not enough to evaluate a long-running agent. Agent runtimes should also measure the quality of the path the agent took: which steps were attempted, which artifacts were preserved, which failures were recovered, and whether user-visible work was handled without leaking internal scaffolding.

This pattern came from inspecting OpenClaw's heartbeat filtering behavior after a trend-radar scan highlighted process-level evals for AI-written work. The useful lesson is broader than heartbeat filtering: the runtime can often judge whether an agent handled a trajectory responsibly even when the final natural-language answer looks fine.

## What to measure

Useful process-quality signals include:

- redundant tool calls or repeated retries without new evidence;
- recovery loops after failed commands, stale references, or missing artifacts;
- whether tool-result errors were surfaced, retried, or silently ignored;
- whether visible notification or messaging tool calls were preserved in the user-facing transcript;
- whether internal heartbeat/tool scaffolding was removed only when doing so would not hide real user work;
- whether the final answer cites the checks, diffs, logs, screenshots, or external state that actually support it.

These signals should be recorded as runtime evidence where possible, not only inferred from a compressed chat transcript.

## Why it matters

A model can produce a clean final answer after a messy or unsafe trajectory. For long-running agents, that is not good enough: repeated retries waste resources, hidden failed tool calls make debugging harder, and over-aggressive transcript cleanup can erase the evidence a user needs to trust what happened.

Process evals create pressure for better operating behavior, not just better prose. They also make regressions easier to catch when runtime subsystems change.

## Practical heuristic

When adding or reviewing an agent-runtime subsystem, ask two questions:

1. What should a good trajectory through this subsystem look like?
2. Which bad trajectories should be detectable even if the final answer is polished?

If those answers are clear, encode them as logs, structured events, tests, or review checks. The runtime should help the agent become more disciplined instead of depending only on prompt reminders.
