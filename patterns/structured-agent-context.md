# Structured Agent Context

Agent runtimes need more than a single free-form prompt when work stretches across days, repositories, and review loops.

A useful context file should preserve both the current state and the rationale behind that state. The pattern is similar to a design token system: a stable name points to a durable decision, and nearby notes explain why that decision exists.

## Use this pattern when

- an agent repeatedly needs the same operating rules or project constraints
- several files define identity, memory, tasks, skills, or permissions
- a long-running project needs continuity after context compression or session restart
- contributors need to understand not only what the agent should do, but why

## Practical shape

Keep separate, purpose-specific files instead of one giant blob:

- identity and public posture
- operator preferences and boundaries
- workspace/repo map
- contribution focus and current branches
- durable memory and dated raw memory
- writeback/publication rules
- reusable skills or checklists

For each important rule, include enough rationale to prevent accidental simplification later. A terse instruction like "sync blueprint after workspace changes" is weaker than an instruction paired with why: the blueprint repo is the public-safe snapshot, so unsynced workspace drift hides how Glenn-Agent actually operates.

## Good habits

- Keep context files public-safe by default.
- Store secrets only in local secret stores, never in context files.
- Prefer small, named sections that can be searched and updated independently.
- Promote repeated lessons from raw daily memory into durable wiki notes.
- Treat context changes as code changes: review diffs, scan for secrets, and commit with a clear message.

## Failure modes

- **Prompt landfill:** one file grows until nobody can tell which rules are still active.
- **Rationale loss:** future edits preserve a rule's wording but remove the reason that made it safe.
- **Private leakage:** operational notes accidentally include tokens, private endpoints, or personal details.
- **Stale maps:** repo paths, branches, or contribution priorities change without updating context.

The target is not more documentation for its own sake. The target is recoverable agency: after interruption, compression, or handoff, the next run can reconstruct both the work and the judgment behind it.
