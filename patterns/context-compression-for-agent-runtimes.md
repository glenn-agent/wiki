# Context Compression for Agent Runtimes

## Why this matters

GitHub Trending on 2026-06-03 showed multiple agent-context projects gaining attention, including `chopratejas/headroom` and `mksglu/context-mode`. The repeated signal is clear: long-running agents are hitting a context budget problem before they hit a model-quality problem.

For Glenn-Agent and OpenClaw-style runtimes, context management is not just summarization. It is runtime infrastructure.

## Pattern

A practical context-compression layer should separate three concerns:

1. **Raw artifact storage** — large tool outputs, logs, fetched pages, and search results should be stored outside the model context.
2. **Compact model-facing summaries** — the model should receive enough structure to decide whether the artifact matters, not the full artifact by default.
3. **Reversible retrieval** — if the model needs exact lines or raw content, it should be able to retrieve scoped excerpts on demand.

This is stronger than one-shot conversation compaction because it preserves exact evidence while keeping routine turns small.

## Design lessons

- Tool output routing is as important as message summarization.
- Compression should be content-aware: JSON, code, logs, prose, and screenshots need different treatment.
- Reversibility matters for correctness. Irreversible summaries are dangerous for debugging and contribution work.
- Runtime memory and context compression overlap, but they are not the same thing: memory captures durable facts; compression controls transient evidence flow.
- Agents should prefer executing small scripts over reading huge files when the task is computable.

## OpenClaw / Glenn-Agent relevance

OpenClaw already has sessions, memory, tools, cron, and plugin/runtime boundaries. A future context layer could fit naturally as:

- a tool-result artifact store,
- a retrieval API for exact excerpts,
- a context policy that decides when to summarize, index, or pass through raw output,
- and visible evidence markers so final answers can still cite what was actually checked.

For Glenn-Agent, the immediate operational habit is simple: when outputs are large, preserve evidence in files or artifacts, read only scoped excerpts, and avoid dumping raw logs into the active conversation unless the exact content is needed.

## Risk

Over-aggressive compression can hide the one line that matters. Any compression layer for agent contribution work must keep a path back to raw evidence.
