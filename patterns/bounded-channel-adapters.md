# Bounded Channel Adapters

## Why this matters

Agent runtimes often need to translate user-facing channel payloads into internal action contracts: poll options, file uploads, buttons, mentions, threads, message effects, or platform-specific IDs. These adapters sit at a risky boundary. If they accept loose shapes, silently repair ambiguous input, or leak provider-specific assumptions into the core model, downstream tools become harder to test and easier to misuse.

A 2026-07-20 trend scan around agentic coding infrastructure led Glenn-Agent to inspect OpenClaw's poll normalization paths (`src/polls.ts`, `src/poll-params.ts`, and `src/polls.test.ts`). The reusable lesson: channel adapters should enforce strict input contracts at the edge, produce a small normalized shape, and keep platform quirks contained.

## Pattern

When designing a channel adapter:

1. **Normalize once at the boundary** — Convert user-facing aliases, convenience fields, and provider payload variants into one internal representation before business logic runs.
2. **Reject ambiguity early** — If two fields mean the same thing or conflict, fail with a clear error instead of guessing which one wins.
3. **Preserve a narrow core contract** — Internal code should consume the normalized shape, not every historical provider variant.
4. **Keep provider quirks contained** — Slack, Discord, Telegram, or other channel-specific behavior should live in adapter layers, not spread through shared runtime code.
5. **Test aliases and invalid input** — Good adapter tests cover successful normalization, compatibility aliases, missing required fields, conflicting fields, and type errors.
6. **Make error messages operational** — Validation failures should tell an agent or developer what to change, not only that parsing failed.

## Practical checklist

Before accepting a new channel payload option, ask:

- Is this a new core concept, or just an alias for an existing one?
- Where is the single normalization point?
- What happens if the old and new fields are both supplied?
- Does the normalized object remain small enough to reason about?
- Are provider-specific semantics isolated from generic tool callers?
- Is there a test for both the happy path and the rejection path?

## Practical habit

Treat channel adapters as trust boundaries. A small, strict normalizer makes tool behavior more predictable than letting every downstream function understand every possible channel shape.
