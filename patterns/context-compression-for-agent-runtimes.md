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

OpenClaw's MCP channel bridge shows a concrete version of this pattern. In `src/mcp/channel-shared.ts`, raw Gateway session rows are normalized into a smaller `ConversationDescriptor` with only the routing and display fields needed to list, read, or reply through a channel session: `sessionKey`, channel route, recipient, optional account/thread identifiers, labels/titles, preview text, and update time. Message polling uses cursor-addressed `QueueEvent` entries so clients can wait for new events without replaying an entire session transcript.

That shape is useful for long-running agents because it separates **reply-capable handles** from full conversation history. The model can carry or inspect a compact descriptor, then request exact message history or attachments only when needed. This is the same design pressure as context compression: keep stable identifiers and previews in the active context, keep full logs outside it, and preserve a route back to exact evidence.

For Glenn-Agent, the immediate operational habit is simple: when outputs are large, preserve evidence in files or artifacts, read only scoped excerpts, and avoid dumping raw logs into the active conversation unless the exact content is needed.

## Session log as recoverable context object

OpenClaw's manual `/compact` path shows the difference between recoverable context management and irreversible summarization.

In `src/auto-reply/reply/commands-compact.ts`, compaction is not just “rewrite the chat into a shorter prompt.” The command resolves the active session entry, session id, session file path, workspace directory, agent directory, skills snapshot, model/provider, harness id, context-token budget, and optional user instructions before calling `compactEmbeddedAgentSession()`. If compaction succeeds, it records updated token counts and, when relevant, the new session id/file back into session state.

That design treats the session log as a runtime object with identity and metadata, not as disposable text. A compactor may produce a shorter model-facing state, but the runtime still knows which session, file, workspace, channel, owner, and harness the compacted state belongs to.

Practical rule: compacted context should remain **recoverably anchored** to the original session artifacts. The active prompt can be summarized, but exact session evidence should remain addressable through session ids, files, message history, tool artifacts, or other stable handles. Otherwise compaction becomes an irreversible lossy rewrite, which is risky for debugging, contribution evidence, approvals, and post-incident review.

## Rolling transcripts should encode reset boundaries, not delete history

OpenClaw's system-agent transcript store shows a smaller version of the same recoverability rule. In `src/system-agent/transcript-store.ts`, the durable machine-wide transcript is a bounded SQLite-backed audit record store with a fixed scope and max-entry cap. Turns are appended with timestamp-plus-UUID keys, while context resets are recorded as explicit `reset` entries instead of destructive deletes.

Reads then decide which view they need:

- `readTranscriptTail(limit)` returns the newest bounded conversational window;
- `readTranscriptTail(limit, { afterLastReset: true })` starts after the newest reset marker;
- reset rows are filtered from model-facing output, but they remain available as durable boundary markers inside storage.

Practical rule: long-running agents should treat "forget this active context" as a view boundary, not necessarily as storage erasure. Bounded retention, explicit reset markers, and filtered read views let the runtime keep recoverable evidence while preventing stale turns from silently reseeding the current conversation.

## Compaction records are boundary markers, not blanket no-op signals

OpenClaw issue `#120455` exposed a related compaction-planning edge. A session whose latest entry is already a `compaction` record may still retain enough model-facing context to need another pass. Treating that latest record as an unconditional no-op confuses a boundary marker with proof that no compactable material remains.

The focused fix in PR `#120467` keeps true no-op conditions narrow: reset-ending branches and empty histories should not plan compaction, but a latest compaction entry should still allow the planner to inspect retained context and prepare another pass when the retained portion remains over budget.

Practical rule: context compaction should be evidence-driven. Boundary records such as resets and compactions are important inputs, but they are not interchangeable. A reset can define a fresh active view; an empty history has nothing to compact; a previous compaction only says summarization happened before. The planner still needs to evaluate the current retained context before deciding whether another compaction is useful.

## Preserve prior summaries during chained compaction

The follow-up review on OpenClaw PR `#120467` exposed another edge in chained compaction. Allowing a second pass is not enough if the second pass only summarizes turn-prefix messages: the runtime must not replace the existing compacted summary with a placeholder such as `No prior history.` just because the current compaction slice contains no fresh summary-bearing entries.

The safer behavior is to treat the existing compacted summary as the baseline history for prefix-only second passes. If no new historical summary is produced, fall back to the previous summary, then append the newly retained boundary information. Regression coverage should prove both the direct compaction output and the replayed session-context view, because a compaction bug can hide in either the summary construction step or the later context reconstruction step.

Practical rule: chained compaction must preserve semantic continuity. A new compaction pass may reduce fresh turns, but it should never erase the prior compacted history unless the runtime has an explicit reset boundary or another evidence-backed reason to do so.

## Risk

Over-aggressive compression can hide the one line that matters. Any compression layer for agent contribution work must keep a path back to raw evidence.
