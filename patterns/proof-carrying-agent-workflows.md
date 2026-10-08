# Proof-Carrying Agent Workflows

Agent work should be reviewable as a bundle of claims and evidence, not accepted as a plain statement that the task is done.

The first AgentProof MVP turns this into a small CLI contract for coding-agent work:

- capture the Git branch, HEAD, upstream, ahead/behind state, and whether the tree is clean;
- record changed files and diff stats so reviewers can see the patch boundary quickly;
- run explicit verification commands and preserve command, exit code, duration, and output excerpts;
- generate a Markdown evidence report that can be pasted into a PR, handoff, or review note;
- keep redaction and non-secret defaults central, because proof artifacts are likely to become public.

The reusable lesson is broader than AgentProof itself: agents should not ask maintainers, users, or future sessions to infer what was verified. The work artifact should name three things separately:

1. **What changed** — files, commits, affected behavior, and intended boundary.
2. **What was proven** — exact commands, results, environment, and any focused runtime checks.
3. **What remains unproven** — skipped tests, missing credentials, environment-only validation, or human review assumptions.

This shape complements observability and eval systems rather than replacing them. Observability can explain runtime behavior; a proof-carrying handoff explains whether a concrete change is ready to review.

Practical heuristic: when an agent prepares code for someone else, treat evidence as part of the deliverable. A useful proof should be small enough to read, exact enough to rerun, and honest enough to distinguish real verification from TODOs or environment blockers.
