# NemoClaw Field Notes

Reusable notes from Glenn-Agent's work on `NVIDIA/NemoClaw`.

## Sandbox agent command examples

When documentation examples run from inside a NemoClaw sandbox, avoid `openclaw agent --agent main --local`.

The `--local` path is blocked inside the sandboxed environment. Examples that need to talk to the assistant from within the sandbox should use the gateway-managed route instead:

```bash
openclaw agent --agent main "your prompt here"
```

This came up while fixing documentation examples in `docs/deployment/deploy-to-remote-gpu.mdx` and `docs/monitoring/monitor-sandbox-activity.mdx` for NemoClaw issue #3692 / PR #3892. The no-`--local` command was validated inside the provided `my-assistant` sandbox and returned normally.

## Contribution hygiene

For NemoClaw contribution branches:

- Pull latest `origin/main` before starting when the tree is clean.
- Keep changes small and verifiable.
- Run the relevant local check; for docs-only changes, `npm run docs:strict` is the useful gate.
- Sign commits and verify with `git verify-commit HEAD` before pushing.
- Use the Glenn-Agent DCO identity consistently.
- For documentation PRs, use NemoClaw's accepted `docs:` prefix rather than Glenn-Agent's generic `doc:` prefix.
- Include the DCO sign-off line in both the signed commit message and the PR description when NemoClaw's checks require it.

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
