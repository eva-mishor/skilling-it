---
name: tech-research
description: Research current state-of-the-art before implementing technical decisions. Use BEFORE recommending or implementing tooling (linters, formatters, test frameworks), libraries, dependencies, or architectural patterns. Triggers on questions like "what's the best way to...", "which library should I use for...", or any implementation planning in fast-moving domains (Python tooling, JS frameworks, cloud infrastructure, AI/ML). Prevents stale training-data recommendations.
---

# Technical Research Skill

Research current SOTA before implementation to avoid stale recommendations.

## When This Triggers

1. **Tooling decisions**: linters, formatters, build tools, package managers
2. **Library selection**: frameworks, ORMs, testing libraries
3. **Architecture patterns**: that evolve rapidly (serverless, edge, AI/ML)
4. **"Best practice" questions**: where the answer changes yearly

## Research Workflow

### Phase 0: Staleness Check (Self-Referential)

Before using `references/fast-moving-domains.md`, verify it's fresh. The file drives the volatility check — if it's stale, the skill itself becomes stale.

1. Read the `<!-- Last reviewed: YYYY-MM-DD -->` comment at the top of `references/fast-moving-domains.md`
2. Compare to today's date
3. If **more than 1 month old**, flag to the user BEFORE proceeding:

```
⚠️ fast-moving-domains.md was last reviewed {N} months ago ({date}).
In fast-moving domains, that's stale. Want me to:
  (a) Proceed anyway (acknowledge risk of missing new disruptors)
  (b) Refresh the file first (quick audit of any new entrants or status changes)
```

4. If the user picks (b), run a brief audit: for each ⚡ highly volatile category, do one current-year web search to check for new tools or status changes. Update the file and the `Last reviewed` date. Then proceed.

**Why this matters:** The file lists things that change fast. It goes stale the same way training data does. A 1-month threshold keeps it meaningful without becoming a nag.

### Phase 1: Identify Domain Volatility

Before recommending, check if the domain is fast-moving:

```
Is this domain in references/fast-moving-domains.md?
  YES, marked ⚡ (highly volatile) -> Mandatory web research (Phase 2) + community search (Phase 2b)
  YES, fast-moving                 -> Mandatory web research (Phase 2)
  NO  -> Check: Has this domain changed significantly since 2023?
         YES -> Web research recommended
         NO  -> Training data likely sufficient (acknowledge explicitly, see Phase 1.5)
```

### Phase 1.5: Slow-Domain Exit Disclosure (Mandatory)

If Phase 1 routes to "Training data likely sufficient" and you skip web research, you MUST prepend this disclosure block to your response:

> ⓘ **Answering from training data only.** I assessed **{domain}** as slow-moving — not in the volatility list, and I don't see significant change since 2023. **If you suspect a recent advancement (new tool, standard, or library release in the last 12 months), reply with `override and search` and I'll re-run with web research.**

This converts a silent miss into an acknowledged limitation. The skill cannot detect disruption in classically-stable fields (cryptography, embedded DBs, OS primitives, compression, sorting) because Phase 1's "changed since 2023" check is asked of the same training data that's being graded. The disclosure puts the user in the loop. Do NOT skip it — without it, a slow-domain misclassification produces a confident-but-stale answer with no signal to the user.

If the user replies `override and search`, restart at Phase 2 with mandatory web research.

### Phase 2: Web Search (Mandatory for Fast-Moving Domains)

Execute searches with current year:

```
Primary search:   "{topic} best practices {current_year}"
Comparison:       "{tool A} vs {tool B} {current_year}"
Migration check:  "{old tool} replacement {current_year}"
```

**Minimum 2 searches required.** Look for:
- Recent blog posts (< 6 months old)
- Official documentation updates
- GitHub stars/activity trends
- Migration guides (signals tool replacement)

### Phase 2b: Community Recency Search (Mandatory for Highly Volatile Domains)

For domains marked as highly volatile in `references/fast-moving-domains.md` (currently: AI/ML Tooling, JS Build Tools, React Ecosystem), add **community searches**:

```
Reddit:    "{topic} reddit {current_year}"
Reddit:    "{topic} site:reddit.com"
HN:        "{topic} site:news.ycombinator.com"
Sentiment: "{topic} production experience {current_year}"
Migration: "switched from {tool} to reddit {current_year}"
```

**Minimum 2 community searches required.** Extract:
- Practitioner sentiment (happy/frustrated/migrating away)
- Production pain points not covered in docs or blogs
- Emerging alternatives practitioners are actually switching to
- "I switched from X to Y because..." migration stories

**Recency caveat:** Web search tools cannot reliably enforce date-range filters (e.g., "last month"). Instead, manually check the date on each result you use. Discard community results older than 6 months for highly volatile domains. If results skew old, add the current quarter to your query (e.g., `"RAG framework reddit Q2 2026"`).

**Why community sources matter:** Blog posts lag reality by 3-6 months. Reddit/HN surface real production experience, backlash, and emerging shifts before any blog covers them. A tool can look great in docs while practitioners are actively abandoning it.

### Phase 3: Validate Against Multiple Sources

Cross-reference findings:

| Source Type | What to Extract |
|-------------|-----------------|
| Official docs | Current recommended approach |
| GitHub repo | Last commit date, open issues, activity |
| Dev blogs | Real-world adoption, pain points |
| Stack Overflow | Common problems, community sentiment |
| Reddit/HN (volatile domains) | Practitioner sentiment, migration stories, emerging alternatives, production pain points |

**Red flags for outdated tools:**
- "Deprecated" or "maintenance mode" mentions
- Repos with no commits in 6+ months
- Blog posts titled "Migrating from X to Y"
- New tools explicitly positioned as replacements

### Phase 4: Compare Training Data vs Reality

Before recommending, explicitly state:

```markdown
## Research Summary

**Question**: {what the user asked}

**My initial assumption** (training data): {what I would have recommended}

**Current SOTA** (web research): {what the research shows}

**Delta**: {key differences, if any}

**Recommendation**: {final recommendation with reasoning}

**Sources**:
- {URL 1}: {key finding}
- {URL 2}: {key finding}
```

If training data matches current SOTA, note: "Verified current - training data still accurate."

## Quick Reference: Common Stale Patterns

| If suggesting... | Verify it hasn't been replaced by... |
|------------------|--------------------------------------|
| flake8 + isort | ruff |
| pip | uv |
| mypy | ty (Beta 2025), pyright |
| black | ruff format |
| webpack | vite, turbopack |
| prettier + eslint | biome |
| Cypress | Playwright |
| Jest | Vitest |
| poetry | uv |
| pipenv | uv, poetry |
| Create React App | Vite, Next.js |

**Meta-pattern: Astral toolchain** - Astral (astral.sh) is systematically replacing Python tooling with Rust-based alternatives. When researching Python tools, always check if Astral has a replacement:
- flake8/isort/pylint → **ruff** (stable)
- pip/poetry/pipenv → **uv** (stable)
- mypy/pyright → **ty** (beta, watch for stable)

## Resources

- **[references/fast-moving-domains.md](references/fast-moving-domains.md)** — domain volatility tiers, pre-built search queries, key disruptors. Loaded by Phase 0 and Phase 1.
- **[references/gotchas.md](references/gotchas.md)** — known failure modes and edge cases. Load when skill behavior surprises you, when debugging a stale recommendation, or during the monthly audit.
- **[references/example-workflow.md](references/example-workflow.md)** — worked example showing Phases 1–4 end-to-end on a highly-volatile domain query. Load if you need to see the procedure run in full.
