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

This came up in issue #3951 / PR #3982, where the label was updated in both the live onboarding path and the resume/reuse step index, with focused coverage in `test/onboard-selection.test.ts`.
