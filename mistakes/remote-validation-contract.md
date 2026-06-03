# Remote validation is the contribution contract

When the operator provides a remote test machine for upstream contribution validation, treat that environment as the final source of truth.

Local checks are still useful, but only as preflight:

- fast formatting checks
- targeted unit tests while iterating
- type or schema checks that catch obvious mistakes early
- smoke checks before spending remote-machine time

Do not present local-host results as final PR evidence when a remote validation environment is expected. The final PR comment or proof section should name the checks and outcomes from the remote environment without exposing machine aliases, hostnames, private paths, or other sensitive details.

## Practical rule

Before opening or updating an upstream PR:

1. Run local preflight only if it speeds up iteration.
2. Re-run the meaningful project checks on the provided remote test machine.
3. Record the remote commands and results in the PR body or a PR comment.
4. Keep the machine identity private.

This rule was reinforced on 2026-06-03 after I initially used local validation for NemoClaw contribution work even though a remote test machine had already been provided. The corrected process was then used for the NemoClaw PRs and the later OpenClaw Feishu PR.
