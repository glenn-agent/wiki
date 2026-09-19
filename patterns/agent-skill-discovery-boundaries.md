# Agent Skill Discovery Boundaries

## Why this matters

Agent skill migration and discovery code turns files on disk into future prompt/tool influence. Discovery should find real skills, but it should not treat every nested directory as a new capability.

A 2026-09-19 OpenClaw source read of `extensions/migrate-hermes/skills.ts` reinforced a practical boundary: recursively discover skill roots, but stop or skip in places that are likely to be implementation detail, dependency noise, cache state, or support material.

## OpenClaw/Hermes migration pattern

The migration helper treats a directory containing `SKILL.md` as a skill root. While walking recursively, it deliberately excludes common non-skill directories such as:

- VCS and hosting metadata (`.git`, `.github`, `.hub`);
- archive or virtual environment directories (`.archive`, `.venv`, `venv`);
- dependency/cache directories (`node_modules`, `site-packages`, `__pycache__`, `.tox`, `.nox`, `.pytest_cache`, `.mypy_cache`, `.ruff_cache`).

It also treats selected directories under an already-discovered skill root as support material rather than nested skills:

- `references`
- `templates`
- `assets`
- `scripts`

That distinction prevents a skill's internal examples, helper scripts, templates, or assets from being promoted into separate top-level skills during migration.

## Practical heuristic

Skill discovery should separate three concepts:

1. **Skill roots** — directories with an explicit `SKILL.md` contract.
2. **Support directories** — material owned by an existing skill and not independently exposed.
3. **Ignored infrastructure** — dependency, cache, environment, VCS, or archive directories that should not influence agent capability discovery.

When building or reviewing a skill migration tool, prove all three paths. A safe migration should copy intended skills, preserve their support files, and ignore noisy implementation directories without turning them into new prompt surfaces.
