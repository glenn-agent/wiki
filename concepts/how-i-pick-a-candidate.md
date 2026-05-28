# How I pick a candidate

This is my own playbook for selecting one daily upstream contribution. Written by me, for me. The operator will revise it as we learn what works.

**Scope is locked.** I only look at two projects:

- `openclaw/openclaw` (the runtime I run on — I dogfood it)
- `NVIDIA/NemoClaw` (the secure-sandbox wrapper I am sandboxed in)

I do not scan, browse, or open anything outside these two repos during scheduled jobs.

---

## What counts as a "good first candidate"

In order. Stop at the first failing rule.

### 0. Existing review loop first

Before looking for a new issue, I check my open PRs in the scoped projects. If an open PR has small maintainer or bot review nits that are clear, low-risk, and testable, that follow-up outranks opening a new candidate.

**Why**: finishing an existing review loop is more respectful to maintainers than adding another PR to their queue. Review follow-up also has stronger context, smaller scope, and a clearer definition of done.

This does not mean endless polishing. I only choose the existing PR path when the requested change is narrow enough to complete and verify in the same daily budget.

### 1. Label filter (initial whitelist)

I only consider issues with at least one of these labels:

- `good first issue`
- `documentation` / `docs`
- `typo`
- `help wanted` *(only if the linked issue is clearly small — re-evaluate body before counting)*

**Why**: low cognitive surface area; maintainers signal "I will accept a small PR here"; small means I can ship and verify same day.

If both projects have zero open issues matching today, I record `NO_GOOD_CANDIDATE` in dated memory with the label filter I used. If three consecutive days produce `NO_GOOD_CANDIDATE`, the operator may loosen the filter by one notch (e.g., add `bug` label, but only `easy` / `triaged` subset).

### 2. Size budget

A candidate passes only if I judge:

- **Effort**: ≤ 2 hours including tests + PR write-up
- **Diff**: ≤ 5 files touched, ≤ 100 lines changed
- **Test surface**: 0 new test infrastructure; only adding to existing test files OR documentation-only

Anything larger goes back to the list. I am not optimizing for impressive PRs.

### 3. Risk filter

I drop candidates that touch:

- Core agent loop / runtime hot path
- Security-sensitive code (auth, secrets, sandbox boundaries)
- Public API or CLI contract (anything in `--help` output)
- Schema / migration / config format changes
- Anything labeled `breaking`, `security`, `needs-design`, `discussion`

### 4. Status filter

I skip candidates where:

- Someone is `assigned`
- There is an open draft PR linked
- The issue is older than 90 days with no activity → maintainers likely dropped it; touching it can re-open a dead discussion
- The issue has > 10 comments arguing about approach → still in design phase, not implementation phase

### 5. Maintainer activity filter

I check `git log --since=7.days` in the target repo. If there is no maintainer activity in the last 7 days, the project is paused — even a perfect PR will sit unreviewed. I prefer to wait.

---

## Selection workflow (daily, ~15 min)

```
1. Check my open PRs in openclaw/openclaw and NVIDIA/NemoClaw.
   If there are clear, small review nits, pick one review follow-up and stop.
2. gh search issues --repo openclaw/openclaw --label "good first issue,documentation" \
     --state open --sort updated --limit 30
3. gh search issues --repo NVIDIA/NemoClaw --label "good first issue,documentation" \
     --state open --sort updated --limit 30
4. For each survivor of label filter:
     a. Read full issue body + every comment.
     b. Apply size / risk / status / maintainer filters above.
     c. If still alive: read the actual file(s) the issue mentions.
     d. Estimate effort honestly. If unsure, drop.
5. Pick at most one. Tag with [openclaw] or [nemoclaw] in dated memory.
6. If zero survive: log NO_GOOD_CANDIDATE with which projects were scanned + the filter used.
```

Alternate projects across days when both have candidates. Don't drain one project's small-issue queue.

---

## Pre-PR review gate (operator-approval required for first 3 PRs)

Once I have a candidate and a verified draft, before pushing to origin I:

1. Run the project's actual tests (`pnpm test` / equivalent). If they don't pass on `origin/main` already (flaky base), I record the baseline and continue carefully.
2. Re-verify diff is minimal — no whitespace churn, no unrelated reformatting.
3. Compose PR title + description following the project's CONTRIBUTING.md or recent merged PR style.
4. DM the operator the draft via Slack with:
   - issue link
   - my proposed diff (gist or `gh pr diff`)
   - my proposed PR title + body
   - what I tested
5. **Wait for operator OK before pushing to my fork or opening the PR.**

After 3 merged PRs, this gate is removed and I can open PRs autonomously, still announcing them via Slack.

---

## Post-PR follow-up

| Within | I do |
|---|---|
| 1h of PR open | Watch CI run; if it fails, push a fix or close + reopen |
| 24h | Reply to any review comment, even if just "looking" |
| 72h no review | One polite ping in the PR thread |
| 7 days no review | Drop it from active tracking; leave PR open |
| Merged | Write a `mistakes/` or `patterns/` wiki note with what worked |
| Closed unmerged | Write a `mistakes/` wiki note with what the maintainer said |

I never force-push after a maintainer has reviewed. I only force-push to clean up my own commits *before* review.

---

## Anti-patterns I avoid

- **No feature work.** I do not propose new functionality unprompted.
- **No drive-by refactors.** If the issue is "fix a typo", I do not also reformat the file.
- **No re-architecting documentation.** I fix what is asked; I do not restructure the docs tree.
- **No claiming tests I did not run.** "Tests pass" only after I have seen them pass.
- **No pretending to triage.** If I read 30 issues and pick none, that is a `NO_GOOD_CANDIDATE` day, not a "I worked hard today" day.
- **No noise PRs to boost count.** A merged PR fixing a real typo > 10 closed PRs of style nitpicks.

---

## Why this exists

The first version of this file was written before my first PR — pure heuristics, no data. It will be wrong in some places. As I get PRs merged (or rejected), I revise it. Every revision is a commit; the git log of this file IS my evolving understanding of "what is shippable."

---

*Operator override knobs you might tune:*

- Label whitelist (line 22)
- Size budget thresholds (line 35)
- Operator-review gate count (currently 3 PRs, line 89)
- Maintainer-activity window (currently 7 days, line 55)
