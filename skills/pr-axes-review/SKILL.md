---
name: pr-axes-review
description: Use immediately after creating a PR. Produces a tiered axes-of-impact review (user trust, cost, coverage, conversion, accessibility, i18n, rollback, analytics, CI, cross-repo, infra, data, security, config side-effects) anchored to the actual diff. Auto-bypasses trivial PRs (docs-only, ≤2 small files, `[skip-axes]` in title/body). Chat-only output — does not post to the PR.
---

# PR Axes Review

## When to use

Immediately after `gh pr create` completes successfully. Can be wired to a PostToolUse hook on that command (see *Auto-trigger* below).

Also invokable manually when someone asks "what should we watch out for in PR #N" — pass the PR number.

## Announce

"I'm using the pr-axes-review skill to surface ship-time impacts of PR #\<n\>."

## Output rules

- **Chat-only.** Don't post to the PR. The user reads the review, decides what's actionable, and carries it into the PR body, team comms, or follow-up tickets themselves. Posting bot-shaped comments on every PR trains reviewers to ignore them.
- **Opinionated.** Direct, critique first, no padding. Match the project's voice if it has one.
- **Diff-anchored.** Reference actual file paths, line numbers, specific values from the diff. Generic warnings are noise.
- **Tiered by surprise risk**, not severity — see Step 4.

## Methodology

### Step 1 — Read the PR

```bash
gh pr view --json number,title,body,files,additions,deletions
gh pr diff $PR_NUMBER
```

Prefer the current-branch PR. If no PR is associated with the current branch, ask the user for the PR number or bail.

### Step 2 — Trivial-PR bypass

Compute the following from the diff:

- `files_total` — total files changed.
- `files_non_test_non_docs` — files outside test paths (`**/*.test.*`, `**/*.spec.*`, `tests/`) and docs (`docs/`, `**/*.md`, `**/*.txt`).
- `diff_lines` — additions + deletions.
- `touches_infra` — any file that influences how the project builds, deploys, or runs in CI: `.github/`, `scripts/`, build configs (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `tsconfig*.json`, `vite.config.*`, `webpack.config.*`, etc.), DB migrations, environment templates, Dockerfiles, lockfiles, project-level config files (`CLAUDE.md`, `.gitignore`, hook scripts).

> Adapt this list to the project's actual stack — the spirit is "files that change *how the system runs*, not *what it does*."

**Bypass criteria — skip the review when any are true:**

1. All changed files are docs-only (`docs/`, `**/*.md`, `**/*.txt`) AND no config files changed AND `!touches_infra`.
2. `files_total` ≤ 2 AND `diff_lines` < 50 AND `files_non_test_non_docs` ≤ 2 AND `!touches_infra`.
3. The PR title or body contains `[skip-axes]` (explicit manual bypass).

When bypassing, emit exactly:

```
## PR axes review — #<n>: <title>

Skipped — trivial PR (<reason: "docs-only" | "≤2 small files" | "skip-axes override">). Invoke this skill manually if you want the review anyway.
```

### Step 3 — Walk the axes (when not trivial)

For each axis: *"Given this diff, what shifts that the PR body doesn't already address?"* Skip axes with nothing to say — padding kills signal.

1. **User trust / first impression.** Anything user-visible whose first-encounter feel could be off (copy tone, empty states, error messages, loading affordances, first paint).
2. **Cost / resource baseline.** Anything that changes per-request cost, per-invocation cost, or steady-state resource consumption — token usage, DB queries, network round-trips, cache hit rate, memory footprint, billing-meter side effects.
3. **Coverage gaps.** What configurations/environments/models/providers does this NOT cover that it should? (e.g. only one model class tested, only one OS, only one locale, only the happy auth path.)
4. **User-flow / funnel.** Does this change a step in any user journey — onboarding, checkout, signup, conversion, retention triggers — in a way that could shift drop-off rates?
5. **Accessibility.** Contrast, focus order, semantic HTML, keyboard reachability, screen-reader labels, reduced motion, dynamic type. Especially for visual changes.
6. **i18n / localization.** Strings that won't translate cleanly, hardcoded English, date/time/number formatting, RTL safety, locale-aware comparisons.
7. **Rollback mechanics.** Can this be safely reverted? Forward-incompatible migrations? Schema changes that consumers depend on? Feature-flag exit path?
8. **Analytics / telemetry baselines.** Does this change what gets emitted (new events, new fields, renamed events, dropped events)? Will dashboards or alerts break or silently shift?
9. **CI / test reliability.** Did anything change that affects test stability, flakiness, runtime, or order-dependence? Hidden test-only behavior in production code paths?
10. **Cross-repo / cross-system sync.** Does this require coordinated changes in another repo, another service, an SDK consumer, a docs site, an external integration?
11. **Infrastructure / secrets / deploy.** New env vars? New secrets? Changed deploy script? IAM/policy changes? Network/firewall implications?
12. **Data / schema / migrations.** Schema changes that need backfill, lock acquisition risk on big tables, NOT NULL on a populated column, dropping/renaming columns consumers still read, RLS or row-level access policies.
13. **Security / trust boundaries.** New input parsing, new auth check, new place that constructs SQL/shell/URL/path from user input, new endpoint, new file upload surface, new deserialization point.
14. **Config side-effects.** `.gitignore`, project-level configs, hooks, skills, CI configs, lockfiles — anything that changes *how the project itself runs* outside the immediate feature.

**Diff-anchoring discipline:** every claim must point to a file or line. "Accessibility concern" is useless. "Low-contrast `bg-slate-300` text on `bg-slate-100` in `Banner.tsx:42` — small body copy at 12px likely below WCAG AA contrast for non-bold text" is the target.

### Step 4 — Rank by surprise risk

This is the central insight of the skill: **surprise risk ≠ severity.** A high-severity bug that the team already knows about is *not* surprising. A medium-severity behavior that nobody will notice for three weeks *is* surprising.

- **Tier 1 — high surprise risk.** The stuff that shows up weeks later as "why is this happening?" — the cost baseline shift nobody warned analytics about, the onboarding/voice mismatch, the model-coverage gap on DEV, the rename that broke a downstream consumer. Deserves explicit pre-merge attention.
- **Tier 2 — half-hour audit.** Cheap pre-merge check: a grep, a secret scan, a contrast run, a manual smoke. Low cost to include; skipping carries real risk.
- **Tier 3 — worth knowing.** Low probability, flagged for awareness. Often informs monitoring or the rollback story.

### Step 5 — Emit

```
## PR axes review — #<n>: <title>

### Tier 1 — high surprise risk

**<N>. <axis: specific claim referencing diff>.** <one paragraph: what shifts, concrete signal, recommended action before merge>

### Tier 2 — half-hour audit

**<N>. <axis: specific claim>.** <one paragraph ending in the specific cheap check>

### Tier 3 — worth knowing

**<N>. <axis: short claim>** — <one or two sentences>

### Suggested review lanes before merging

| Check | Cost | Catches |
|---|---|---|
| <specific command or manual step> | <30s / 2min / 15min> | <which Tier N items> |

<4–6 rows, prioritized>
```

## Red flags

| Thought | Reality |
|---|---|
| "The PR body already lists risks." | PR-body risks are what the author thought of. Axes review is what they didn't. Run it. |
| "I'll post this as a comment to help reviewers." | No. Output target is chat-only — the user decides what to carry forward. Bot-shaped PR comments train reviewers to skim past them. |
| "I'll grade by severity." | Grade by surprise risk. High-severity obvious problems are usually Tier 2 or 3; medium-severity invisible ones are Tier 1. |
| "Every axis needs a section." | Skip axes with nothing to say. Padding kills signal. |
| "This is a tiny PR, just skim it." | The bypass heuristic in Step 2 decides that. Either skip entirely (per heuristic) or run the full review. Don't half-run it. |
| "I'll file follow-up tickets myself." | No — that's the user's decision. You surface; they decide. |

## Auto-trigger (optional)

Wire to a PostToolUse hook matched on `gh pr create` to fire automatically after every PR creation:

```jsonc
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "match": "gh pr create",
        "script": ".claude/hooks/pr-axes-review-trigger.sh"
      }
    ]
  }
}
```

The trigger script emits a reminder like: *"You just created a PR. Run the pr-axes-review skill on it before continuing."* Adapt to your project's hook conventions.

## Related

- A plan-time counterpart (apply the same 14 axes against an implementation plan before any code lands) is a natural companion. Catches design-fixable issues; pr-axes catches ship-time mitigatable ones. Not bundled in this repo.
