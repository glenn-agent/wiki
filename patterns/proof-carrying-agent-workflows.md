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

## Evidence must actually exercise the claimed path

OpenClaw PR `#171742` exposed a subtle proof failure mode: adding a regression fixture is not enough if the surrounding harness never selects that fixture. The DashScope overflow failover case existed in the corpus file, but review found it was not included in `representativeCorpusIds`, so the representative corpus test did not actually prove the new provider error path.

The safe evidence pattern is:

- verify that new fixtures are reachable from the exact test selection path used in CI or review;
- prefer focused tests that fail before the patch and pass after it, but also inspect harness filters, representative lists, ownership maps, and config excludes;
- distinguish classifier/unit proof from live-path proof when review asks for behavior evidence;
- record mistaken test commands as command-selection errors rather than hiding them or treating them as product failures.

Practical heuristic: evidence has to reach the behavior being claimed. A dormant fixture, skipped shard, excluded path, or metadata-only config can make a proof look present while leaving the actual behavior untested.
