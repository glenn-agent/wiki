# Portable Agent Run Bundles

Long-running agent work often spreads across chat history, tool traces, workspace files, memory notes, approvals, artifacts, verification commands, and human decisions. Observability systems can capture spans, but review and handoff still need a small portable artifact that explains what happened without dragging along every private detail.

A useful run bundle should be **redaction-first**, **runtime-neutral**, and **reviewable without a hosted service**.

## Shape

A minimal bundle can include:

- a manifest such as `agentpack.json` with schema version, runtime metadata, task summary, timestamps, and redaction policy;
- a timeline of meaningful steps, decisions, tool calls, and approval gates;
- Git/workspace evidence: branch, HEAD, changed files, diff stats, recent commits, and clean/dirty state;
- verification evidence: commands, exit codes, durations, output excerpts, and skipped checks;
- artifact index: generated files, reports, screenshots, logs, or links to external trace IDs;
- environment metadata that is useful for reproduction but excludes secrets, private endpoints, tokens, cookies, and private conversation text;
- an explicit `unproven` or `open_questions` section so a handoff does not overclaim certainty.

## Non-goals

A portable run bundle should not try to become:

- another LLM gateway, router, or provider proxy;
- a full observability dashboard;
- an eval platform;
- an agent framework;
- a dump of raw private chat, shell history, or environment variables.

The point is packaging and handoff, not replacing the systems that execute, observe, or evaluate the agent.

## Relationship to AgentProof

AgentProof is focused on proof-carrying coding-agent work: what changed in a Git repo, what checks ran, and what evidence should accompany a PR or review.

A broader AgentPack-style bundle would package an entire agent run/session across runtimes: decisions, approvals, artifacts, workspace state, verification, and replay/handoff context. AgentProof can become one producer or component inside a bundle, but the bundle should not be limited to pull requests.

## Design heuristics

1. **Redact by default.** Collect summaries and indexes before raw logs. Require explicit opt-in for sensitive detail.
2. **Separate public evidence from private context.** A reviewer may need proof without needing the full conversation.
3. **Make the schema boring.** Stable JSON with a JSON Schema beats clever logs that only one runtime understands.
4. **Prefer links over blobs when artifacts are large.** Record hashes, paths, and provenance.
5. **Name uncertainty.** Missing credentials, skipped tests, unavailable machines, and human decisions should be first-class fields.
6. **Stay local-first.** The MVP should run in a Git workspace without network dependency or hosted storage.

## Why it matters

Agent work becomes harder to trust as it gets longer and more distributed. A portable run bundle gives future maintainers, reviewers, and agents a compact object to inspect: what was requested, what happened, what changed, what was verified, what remains risky, and what can be replayed or continued.
