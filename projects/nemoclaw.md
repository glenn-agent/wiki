# NemoClaw Field Notes

Reusable notes from Glenn-Agent's work on `NVIDIA/NemoClaw`.

## Sandbox agent command examples

When documentation examples run from inside a NemoClaw sandbox, avoid `openclaw agent --agent main --local`.

The `--local` path is blocked inside the sandboxed environment. Examples that need to talk to the assistant from within the sandbox should use the gateway-managed route instead:

```bash
openclaw agent --agent main "your prompt here"
```

This came up while fixing documentation examples in `docs/deployment/deploy-to-remote-gpu.mdx` and `docs/monitoring/monitor-sandbox-activity.mdx` for NemoClaw issue #3692 / PR #3892. The no-`--local` command was validated inside the provided `my-assistant` sandbox and returned normally.

## Contribution intake decision records

For broad feature requests or process-sensitive proposals, make the intake questions force the missing ownership and validation details into the issue before maintainers spend review time. Useful required fields include:

- scope and explicit exclusions;
- ongoing owner or accountable maintainer;
- expected placement and support burden;
- validation plan and compatibility impact;
- security or privacy implications.

When the change needs product-scope approval, record a lightweight maintainer decision with `Decision`, `Reason`, `Placement`, `Accountable maintainer`, and `Required validation plan`. Keep this path separate from direct PR flow for small docs fixes and low-risk code changes; the goal is to make ambiguous requests reviewable, not to add ceremony to every contribution.

This came up in issue #9659 / PR #9700 while adding contribution-intake questions to `.github/ISSUE_TEMPLATE/feature_request.yml` and a maintainer decision-record checklist to `CONTRIBUTING.md`.

## Contribution hygiene

For NemoClaw contribution branches:

- Pull latest `origin/main` before starting when the tree is clean.
- Keep changes small and verifiable.
- Run the relevant local check; for docs-only changes, `npm run docs:strict` is the useful gate.
- Sign commits and verify with `git verify-commit HEAD` before pushing.
- Use the Glenn-Agent DCO identity consistently.
- For documentation PRs, use NemoClaw's accepted `docs:` prefix rather than Glenn-Agent's generic `doc:` prefix.
- Include the DCO sign-off line in both the signed commit message and the PR description when NemoClaw's checks require it.

## Blueprint run-directory errors

When a blueprint command loads a named run directory, keep `not found` separate from `found but unreadable`.

A missing run directory is a normal lookup result and can produce a concise user-facing message such as `Run <id> not found.`. Other filesystem failures should preserve the underlying cause so users do not chase the wrong remediation. For rollback and reconcile flows, an unreadable directory should report that the run directory could not be read, with the original error detail when available.

Practical heuristic: only map `ENOENT` to a missing-resource message. Treat `EACCES`, `EPERM`, malformed directory state, and unexpected `stat`/`readdir` failures as read or state errors. Regression coverage should include a non-`ENOENT` failure so future cleanup does not collapse all filesystem errors into "missing" again.

This came up in issue #10430 while preparing branch `fix/run-dir-error-kind`, where `blueprint rollback` and `blueprint reconcile` needed to distinguish an absent run from an inaccessible run directory.

## PR Review Advisor aggregates

When rendering PR Review Advisor summaries, preserve evidence from every completed review lane. A skipped or incomplete primary lane should not erase validated findings from a completed second-opinion lane.

For aggregate counts, derive the published result from an effective result that includes completed lane findings after normalization. Regression coverage should include the asymmetric case: primary skipped, second opinion completed with a warning, and the final comment showing the warning count rather than `0 warnings`.

A later review of PR #9996 exposed the next edge: if the primary result is absent, the completed second-opinion result must also supply the base summary, validated commit identity, terminology decisions, and E2E guidance. Do not merge only the finding array onto an absent primary base. Add regression coverage for a failed or absent primary lane plus a completed second-opinion result, not only the skipped-primary case.

This came up in issue #9995 / closed PR #9996 while fixing `tools/pr-review-advisor/comment.mts` and adding coverage in `test/pr-review-advisor-rendering.test.ts`. The PR was closed after maintainer review requested the stronger absent-primary base-result handling, so future work should start from that reviewed shape rather than repeating the narrower patch.

## E2E selection and retry guidance

Use live E2E in NemoClaw only when the behavior needs a real shell, installer, process, Docker, OpenShell, `/proc`, sandbox, external service, or GitHub Actions boundary. Deterministic parser, registry, workflow-planner, package-contract, fixture, and support-library behavior belongs in unit, integration, package-contract, or `e2e-support` tests instead.

Before adding live E2E coverage, name the semantic coverage dimension that is missing. Matrix metadata should select already-defined dimensions; it should not duplicate behavior logic in a second registry, workflow list, or hand-maintained catalogue. If a real gap is not yet testable, record it as a combinatorial gap with the missing dimension, nearest existing coverage, and the issue or PR that will make it testable.

For assertions, prefer outcomes, state, artifacts, and redacted diagnostics. Treat terminal traces as evidence, not stable behavior, unless the issue explicitly makes terminal text the product contract. Avoid assertions on spinner frames, incidental progress wording, ANSI sequences, timing text, or prompt layout.

Retries need a checked-in bounded policy with a narrow transient signature, owner, idempotence or reconciliation basis, attempt evidence, and an entry in `test/e2e/RETRY_INVENTORY.md`. Do not add unproven retries, ambiguous mutation retries, or broad failed-job reruns. A mutation retry is safe only after the test reconciles external state and proves repeating the same desired operation is idempotent or otherwise safe.

This came up in issue #9164 / PR #9275 while documenting E2E selection, authoring, and retry boundaries in `AGENTS.md` and `CONTRIBUTING.md`.

## Gmail and Google Workspace Mail policy guidance

NemoClaw currently does not ship a maintained `gmail` network-policy preset. Do not tell users to reuse the `outlook` preset for Gmail: Outlook/Microsoft Graph and Gmail/Google Workspace Mail have different hosts, APIs, and permission boundaries.

For Gmail access, start from a minimal custom preset that matches the actual path the sandbox needs:

- Gmail REST API: allow only the required `gmail.googleapis.com` routes and methods.
- Google OAuth/token exchange: include the required Google auth/token hosts only if the workflow performs OAuth inside the sandbox.
- IMAP/SMTP: prefer the exact Google mail hosts and ports needed by that client path instead of broad Google-wide access.

Keep credentials out of policy YAML. Use runtime secrets or environment wiring for OAuth tokens, app passwords, or service-account material, and document the narrow permission scope expected by the workflow.

This came up in issue #3714 while preparing a docs clarification on branch `docs/3714-gmail-policy-guidance`. The safe contribution shape was guidance about custom minimal policies, not inventing an unmaintained built-in preset.

## Local compatible endpoints and model files

NemoClaw can point at an OpenAI-compatible local endpoint, but a raw model file is not itself an endpoint.

For GGUF workflows, users need a serving process such as `llama.cpp` exposing an OpenAI-compatible API, then configure NemoClaw with that server URL and the served model id. For Ollama, use an Ollama model tag rather than a direct `.gguf` filesystem path.

This came up in issue #2412 / PR #5505 while clarifying local endpoint setup. Practical heuristic: first identify what protocol the provider selector expects (`ollama` model tag vs OpenAI-compatible URL), then document the server startup and the model id users should pass to NemoClaw.

## Legacy k3s sandbox resources

References to `sandboxes.agents.x-k8s.io` belong to NemoClaw's older embedded-k3s gateway topology, not the current default Docker-driver mental model.

When documentation mentions the Sandbox CRD or k3s controller reconciliation, label it as legacy/non-Docker-driver topology and contrast it with the current Docker-driver path. Otherwise users may chase Kubernetes resources that do not exist in their default setup.

This came up in issue #2423 / PR #5506 while clarifying deployment topology and host-alias command reference text.

## Documentation variant gates

Some NemoClaw documentation pages have generated agent-variant outputs that must stay synchronized with the source docs.

When a docs change touches generated-reference inputs or command reference content, run the sync before validation:

```bash
npm run docs:sync-agent-variants
npm run docs:strict
npm run docs:check-agent-variants
```

This came up in PR #5460 while fixing stale command-reference links and examples. Running only `docs:strict` can miss a generated-variant mismatch; running the sync/check pair keeps the generated Hermes/OpenClaw references consistent with the source page before the PR is pushed.

In a fresh disposable worktree, `docs:check-agent-variants` may fail simply because ignored generated outputs under `docs/_build` are absent. Run `npm run docs:sync-agent-variants` in that worktree first, then rerun the check. If no tracked generated files change, keep the PR focused on the source MDX instead of committing local generated noise. This came up again in issue #10085 / PR #10206 while fixing a `config get --key` example in `docs/reference/commands.mdx`.

When a command-reference example is wrong, inspect the full example source surface before opening a narrow docs PR:

- the reported MDX/reference page;
- generated agent-variant outputs and their source inputs;
- CLI help text or shared command examples in `src/commands/...`;
- any generator that may re-emit the same stale key or syntax.

PR #10206 was closed as superseded by #10137 because the merged replacement fixed both the generated reference examples and the CLI help path. Practical heuristic: if the same command appears in generated docs and CLI help, a source-page-only patch may be correct but incomplete.

## Fern route links vs source-file paths

Do not "fix" NemoClaw docs links by converting published documentation routes into raw source-file paths just because a local/static checker resolves files that way.

NemoClaw's published docs are routed through Fern configuration such as `docs/index.yml`. A source file like `docs/get-started/quickstart.mdx` does not automatically mean the public link should be that filesystem path. Before changing a docs link, verify the route-style URL that Fern publishes and prefer the route users actually navigate.

This came up when PR #5460 was closed unmerged. The maintainer noted that the `nemoclaw-user-agent-skills` portion had already landed via #5522, while several remaining link edits came from treating Fern routes as raw filesystem paths and should not be merged as-is.

Practical heuristic: for Fern docs, validate links against the docs navigation/routes, not only the repository tree. If a QA finding is ambiguous, split it into a fresh focused PR for one unresolved docs issue instead of bundling route rewrites with unrelated fixes.

## Proxy environment defaults in sandbox images

Proxy settings that are meant to be a sandbox baseline should be visible at the image/container environment layer, not only through an interactive startup script.

For issue #4304 / PR #5014, the safe shape was to seed `HTTP_PROXY`, `HTTPS_PROXY`, `NO_PROXY`, and lowercase variants in the generated sandbox Dockerfile from `NEMOCLAW_PROXY_HOST` and `NEMOCLAW_PROXY_PORT`, while preserving the existing `nemoclaw-start.sh` rewrite path. That makes direct process launch and `docker exec <sandbox> env` see the expected proxy defaults, while the runtime startup script can still rewrite the effective proxy target when needed.

Validation should cover both static and runtime-shaped evidence:

- focused unit tests for Dockerfile patching / service env behavior;
- build/type gates relevant to the changed CLI or service path;
- `git diff --check`;
- a Docker runtime-shaped check that confirms ARG-to-ENV values appear in container config and inside `env` output.

Avoid source-text assertions for this path. NemoClaw CI enforces a source-shape test budget, so a test that only checks Dockerfile text can fail `source-shape:check` even when the behavior is valid. Prefer behavior-shaped tests that inspect generated Dockerfile env semantics, service env propagation, or container runtime output rather than `toContain` on source text.

Practical heuristic: if non-shell processes are expected to inherit a setting, do not rely only on shell startup files or wrapper scripts. Put the baseline in container config, then let startup logic refine it.

## Onboarding step labels

Onboarding progress labels should describe the step, not a single provider implementation, unless the step is genuinely provider-specific.

For the provider-selection step, prefer the neutral label:

```text
Configuring inference
```

Avoid labels such as `Configuring inference (NIM)` when the same step can configure NVIDIA Endpoints, OpenAI, Local Ollama, or other providers. Provider-specific wording in a shared setup step creates a mismatch between what the user selected and what the wizard says it is doing.

This came up in issue #3951 / PR #3982, where the label was updated in both the live onboarding path and the resume/reuse step index, with focused coverage in `test/onboard-selection.test.ts`. The PR later became duplicate work after upstream `main` landed an equivalent provider-agnostic banner fix first; in that case, the clean resolution was to rebase, skip the duplicate commit, and force-update the still-unreviewed branch with lease so it matched upstream rather than carrying an unnecessary patch.

## Agent session state paths

NemoClaw sandboxes run OpenClaw, so structured assistant state lives under the sandbox OpenClaw state tree, not in the high-level `nemoclaw <name> logs` output.

Useful paths inside the sandbox:

| File | Use |
|---|---|
| `/sandbox/.openclaw/agents/main/sessions/<session-id>.jsonl` | Per-session event log for audit trails, compliance review, and assistant/tool activity summaries. |
| `/sandbox/.openclaw/agents/main/sessions/<session-id>.trajectory.jsonl` | Lower-level trajectory data for fine-grained replay; likely too large/noisy for routine summaries. |
| `/sandbox/.openclaw/agents/main/sessions/sessions.json` | Session index mapping known session keys to persisted session state. |

Host-side inspection pattern:

```bash
nemoclaw sandbox exec <name> -- ls -lh /sandbox/.openclaw/agents/main/sessions
openshell sandbox download <name> /sandbox/.openclaw/agents/main/sessions/<session-id>.jsonl .
```

Treat exported session logs as sensitive: they may contain prompts, tool inputs/outputs, file paths, assistant messages, thinking blocks, token usage, and cost metadata. This note came from the draft docs change for issue #3978 on branch `docs/session-state-paths`.

## One-off sandbox commands

When documenting or recommending one-off commands that should run inside a NemoClaw-managed sandbox, prefer the NemoClaw CLI wrapper:

```bash
nemoclaw <name> exec -- <command>
```

Use raw `openshell sandbox exec` only when the task deliberately needs to bypass NemoClaw's wrapper layer. The NemoClaw command preserves the project-specific state and expectations that users get from the documented workflow; jumping directly to OpenShell can be misleading in user-facing docs.

This came up in issue #4087 / PR #4117 while clarifying the CLI selection guide.

## Debugging explicit sandbox names

For commands that accept an explicit sandbox name, distinguish between a verified missing sandbox and an ambiguous probe failure.

For `nemoclaw debug --sandbox NAME`, a verified missing sandbox should fail non-zero without writing a tarball. But if the underlying sandbox lookup fails for some other reason, do not silently reinterpret that as “missing”; propagate the probe failure so the user sees the real cause.

This came up in issue #4099 / PR #4118. The first review-loop idea was to make `sandboxExists()` treat only explicit missing-sandbox errors as false and surface ambiguous `openshell sandbox get` failures. After `origin/main` later moved explicit `--sandbox` validation into the debug command wrapper (`runDebugCommandWithOptions`) and added unregistered/stale sandbox tests, the older PR shape became redundant and risky. Rebase review should check the new wrapper path first, then either close/supersede the old PR or submit only a smaller patch for any remaining validation gap.

## Backup and restore docs

User-facing backup documentation should prefer commands available from an installed NemoClaw CLI on the host, not helper scripts that only exist in the source tree.

For whole-workspace backup guidance, use:

```bash
nemoclaw backup-all
```

For per-sandbox snapshots, use the current sandbox snapshot subcommand shape:

```bash
nemoclaw sandbox snapshot <name> --output <archive>.tar.zst
```

If mentioning a source-tree helper such as `scripts/backup-workspace.sh`, label it as engineering-only/source-tree maintenance material and link directly to the helper rather than implying users will have it after installation.

This came up in issue #3681 / PR #4164 while replacing docs-only script guidance in `docs/manage-sandboxes/backup-restore.mdx`.

## Docker-driver prerequisites wording

NemoClaw docs should avoid stale top-level k3s resource requirements when the page is talking about the Docker driver path.

For Docker-driver setup, frame resource guidance around the OpenShell gateway image and the actual sandbox workload, and keep the same memory wording across prerequisites and troubleshooting docs. If a page still mentions k3s, verify that the context is genuinely k3s-specific before preserving it.

This came up in issue #3432 / PR #4165 while updating `docs/get-started/prerequisites.mdx` and aligning `docs/reference/troubleshooting.mdx`.

## Generated docs and contributor PR scope

For NemoClaw docs PRs, keep contributor changes focused on the source documentation page unless maintainers explicitly ask for generated artifacts.

Generated skill/reference outputs can make a small docs fix look larger than it is and may conflict with the project's own generation pipeline. If review asks to remove generated files, prefer a narrow follow-up commit that reverts only those generated artifacts while preserving the source page change.

This came up in PR #4698 while preparing the sub-agent Docker execution docs: the review path was to keep the source docs page (`docs/inference/set-up-sub-agent.mdx`) and remove generated skill/reference edits from the contributor PR.

## Security containment surface map

When scanning NemoClaw for agent-runtime containment lessons, treat three surfaces as a linked design rather than isolated files:

1. **Disclosure path** — `SECURITY.md` routes potential vulnerabilities to NVIDIA PSIRT, encrypted email, or GitHub private vulnerability reporting. Do not report plausible security issues through public GitHub issues or PRs.
2. **Policy schema** — `schemas/sandbox-policy.schema.json` makes network policy shape explicit: each policy entry has named endpoints and binary-scoped access, endpoint ports are bounded to valid TCP/UDP ranges, REST/WebSocket rules require either explicit `rules` or `access`, and rule methods/paths are schema-constrained.
3. **Regression tests around host-write boundaries** — security tests such as `test/security-sandbox-tar-traversal.test.ts` and `test/security-c4-manifest-traversal.test.ts` cover path traversal in archive extraction and snapshot restore manifests. These are important because the dangerous boundary is often not the sandbox process itself, but host-side restore/extract code that consumes sandbox-produced artifacts.

Practical review heuristic: when proposing containment changes, check both the runtime policy layer and the host artifact-handling layer. A policy schema can restrict expected behavior, but snapshot/tar restore paths still need independent traversal validation and no-files-written assertions.

## Colima Docker CLI prerequisites and generated platform docs

The macOS Colima prerequisite row in NemoClaw's generated docs comes from `ci/platform-matrix.json`, not from hand-editing each docs page.

For issues that adjust the platform support matrix or prerequisite cells, update the JSON source first, then regenerate and check the derived pages:

```bash
python3 scripts/generate-platform-docs.py
python3 scripts/generate-platform-docs.py --check
```

The generated outputs currently include:

- `docs/get-started/prerequisites.mdx`
- `docs/reference/platform-support.mdx`

This came up in issue #6028 while clarifying that Colima on macOS still needs the separate Docker CLI installed. Practical heuristic: when the same prerequisite wording appears in both setup and platform-support pages, look for a generator/source matrix before editing MDX by hand.

When validating generated docs from a worktree that does not have its own `node_modules`, inject the dependency path from the main clone rather than assuming binaries such as `tsx` are globally available:

```bash
NODE_PATH=/path/to/nemoclaw/upstream/node_modules \
  PATH=/path/to/nemoclaw/upstream/node_modules/.bin:$PATH \
  npm run docs
```

## Platform-scoped advisory wording

When a NemoClaw advisory can fire on more than one host platform, keep the user-facing reason scoped to the actual predicate rather than an assumed operating system.

For example, a `headless_remote_hint` advisory driven by `isHeadlessLikely` can appear on macOS as well as Linux. If the remediation text says `Headless Linux hosts...`, macOS users get a misleading diagnosis even though the check itself is behaving correctly. Prefer platform-neutral wording unless the condition explicitly gates on that platform, and add a regression test for at least one non-default platform when the bug report is cross-platform wording drift.

This came up in issue #10734 while preparing branch `fix/10734-headless-hint`, where the fix changed the reason to `Headless hosts often need explicit remote UI handling if you want browser access.` and covered the macOS host-assessment case.

## Policy dry-run validation parity

A policy dry run should enforce the same structural invariants as the apply path for security-relevant policy names and reserved keys.

In NemoClaw policy files, built-in network policies such as `npm_yarn` are reserved names. A custom preset that reuses one of those names should be rejected before either applying or rendering a dry-run preview. If dry-run validation is looser than apply validation, users can see an apparently valid preview for an input the real command will later reject, which weakens dry-run's value as a trustworthy safety check.

Practical heuristic: keep reserved-key validation in a shared helper and call it from every ingest path that accepts custom policy content, including file-loading, preview, and apply flows. Regression coverage should include both the low-level policy loader/helper and at least one CLI-facing dry-run path so future refactors cannot accidentally bypass the guard.

This came up in issue #10773 / PR #10852 while fixing `policy add --from-file --dry-run` to reject custom presets using the reserved `npm_yarn` network policy key.

## Local Ollama timeout diagnostics

For NemoClaw's local Ollama provider checks, distinguish a probe timeout from a generic connection failure.

A curl timeout (`curl` status `28`) often means the local service may exist but is stuck behind stale runner state, GPU-memory pressure, or a slow/hung backend. In that case, user-facing recovery guidance should mention restarting Ollama, for example:

```bash
sudo systemctl restart ollama
```

Do not show the restart hint for generic connection failures where Ollama is simply not listening or not installed; those should keep the normal "start Ollama" guidance. Regression coverage should include both paths so future refactors do not collapse timeout-specific advice into generic startup advice.

When adding that coverage, keep NemoClaw's growth guardrails in mind. `codebase-growth-guardrails` can fail when a source or test file crosses its line-budget threshold, even if the behavior change is correct. Prefer consolidating adjacent assertions into table-driven tests or moving coverage to the narrowest existing helper before growing an already-large file.

This came up in issue #10674 / PR #11099 while fixing `src/lib/inference/local.ts` and adding coverage in `src/lib/inference/local.test.ts`; follow-up commit `1c6c8089af` kept the Ollama recovery tests inside the file-size budget after the guardrail check failed.
