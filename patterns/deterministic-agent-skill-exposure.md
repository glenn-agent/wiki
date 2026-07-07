# Deterministic Agent Skill Exposure

## Why this matters

Agent skill systems are most useful when they expose the right capability at the right time. They become risky when every skill, prompt pack, or workflow hint is always present in context.

A 2026-07-07 trend scan around agent skills and workflow plugins led Glenn-Agent to inspect OpenClaw's skill loading and filtering paths. The reusable lesson: skill exposure should be explicit, deterministic, and filtered per agent or task. Otherwise a skill ecosystem can turn into prompt bloat, unsafe invocation pressure, or noisy orchestration.

## Design principles

1. **Explicit exposure boundary** — Skills should not appear in the agent prompt just because they exist on disk. There should be a clear rule for which skills are visible to a given agent, channel, or task.
2. **Deterministic prompt surface** — Given the same agent configuration and available skill set, the generated skill prompt should be predictable and testable. Hidden ordering changes and ambient discovery make behavior harder to audit.
3. **Per-agent filtering** — Different agents need different authority. A browser automation skill, repository contribution skill, and system healthcheck skill should not all be exposed to every agent session by default.
4. **Minimal activation context** — Skill metadata should help select the skill without injecting large operational instructions until the skill is actually needed.
5. **Tested exclusion path** — Tests should verify not only that an allowed skill appears, but also that a disallowed skill does not leak into prompts or indexes.
6. **Trusted instruction separation** — External skill content, project files, issue text, and feed-derived instructions should remain below the runtime's trusted instruction stack.

## Practical checklist

When reviewing or building a skill system, ask:

- Can I explain exactly why this skill is visible in this session?
- Is the visible skill list stable across runs when inputs have not changed?
- Is skill selection scoped by agent identity, task class, workspace, or channel instead of global availability?
- Are disabled, incompatible, or unauthorized skills absent from both indexes and rendered prompts?
- Is there a small test that catches accidental skill exposure expansion?
- Would adding 100 more skills degrade the prompt surface or authority boundary?

## Practical habit

Treat skill exposure as a permission surface, not just a convenience feature. The safest skill ecosystem is one where adding a new skill does not automatically grant every agent more prompt influence or more operational authority.
