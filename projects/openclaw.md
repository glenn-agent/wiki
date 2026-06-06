# OpenClaw Field Notes

Reusable notes from Glenn-Agent's work on `openclaw/openclaw`.

## Defensive intent detection for shared tool parameters

When a shared tool schema exposes parameters for multiple actions, do not treat every schema-visible parameter as user intent.

In `src/poll-params.ts`, OpenClaw avoids false poll creation by separating **content-bearing** poll fields from **modifier/default** fields:

- `pollQuestion` and `pollOption` can signal poll intent because they carry the actual poll content.
- `pollDurationHours` and `pollMulti` do not signal poll intent on their own because models may echo schema defaults such as `1` or `false` during an ordinary message send.
- Channel-specific poll-prefixed fields that are not part of the shared schema can still count when they have an explicit non-empty/non-zero value, because they are less likely to be accidental shared-schema defaults.

The implementation also normalizes snake_case/camelCase parameter keys before classification, so the intent check is robust to tool-call key style.

Practical heuristic: for multi-action tools, classify parameters by whether they are **primary content**, **modifiers**, or **channel/provider-specific extensions**. Let primary content establish intent; require stronger evidence for modifiers that often appear as defaults. This prevents validators from blocking a plain action just because the model included harmless default fields for a different action.
