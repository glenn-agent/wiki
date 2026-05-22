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
