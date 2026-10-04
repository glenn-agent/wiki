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

## Prompt contract pattern

A 2026-10-04 OpenClaw source read of `src/agents/system-prompt.ts` reinforced a complementary prompt-side boundary. `buildSkillsSection()` does not inline every skill's full instructions into the system prompt. Instead, it renders a compact contract: scan `<available_skills>`, read the single most specific matching `SKILL.md`, re-read it when its version changes, read none when no skill clearly applies, and batch external API writes safely.

This keeps selection authority in the runtime's skill inventory while keeping operational instructions demand-loaded. The prompt tells the agent **how to choose and load one skill**, not to treat the full skill library as active context.

Practical heuristic: a scalable skill system should expose a small, auditable selection rule up front and defer full procedural instructions until a task clearly matches. The default path should be "no skill loaded," not ambient prompt expansion.

## Practical checklist

When reviewing or building a skill system, ask:

- Can I explain exactly why this skill is visible in this session?
- Is the visible skill list stable across runs when inputs have not changed?
- Is skill selection scoped by agent identity, task class, workspace, or channel instead of global availability?
- Are disabled, incompatible, or unauthorized skills absent from both indexes and rendered prompts?
- Is there a small test that catches accidental skill exposure expansion?
- Would adding 100 more skills degrade the prompt surface or authority boundary?

## Doctor repairs should distinguish broken skills from filtered skills

A 2026-09-15 OpenClaw source read reinforced that skill repair commands need a narrower contract than generic skill discovery.

In `src/commands/doctor-skills-core.ts`, `collectUnavailableAgentSkills()` selects only skills that are currently unusable because requirements are missing. It deliberately excludes skills that are:

- already disabled by config;
- blocked by an allowlist;
- blocked by an agent-specific filter;
- incompatible with the current platform.

That distinction matters. A doctor/fix command should repair broken local readiness, not override intentional exposure policy or disable a skill merely because the current host is not the right OS. Platform-incompatible skills may still be valid on another machine; allowlist- and agent-filtered skills are policy decisions rather than broken installs.

Practical heuristic: automated repair should operate on the same readiness model as discovery, but only mutate entries whose failure means "this allowed skill cannot run here." Keep policy exclusions, disabled state, and platform applicability as non-mutating explanations unless the user explicitly asks to change those policies.

## Practical habit

Treat skill exposure as a permission surface, not just a convenience feature. The safest skill ecosystem is one where adding a new skill does not automatically grant every agent more prompt influence or more operational authority.
