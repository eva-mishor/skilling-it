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

### Phase 1: Identify Domain Volatility

Before recommending, check if the domain is fast-moving:

```
Is this domain in references/fast-moving-domains.md?
  YES -> Mandatory web research (Phase 2)
  NO  -> Check: Has this domain changed significantly since 2023?
         YES -> Web research recommended
         NO  -> Training data likely sufficient
```

### Phase 2: Web Search (Mandatory for Fast-Moving Domains)

Execute searches with current year:

```
Primary search:   "{topic} best practices 2025"
Comparison:       "{tool A} vs {tool B} 2025"
Migration check:  "{old tool} replacement 2025"
```

**Minimum 2 searches required.** Look for:
- Recent blog posts (< 6 months old)
- Official documentation updates
- GitHub stars/activity trends
- Migration guides (signals tool replacement)

### Phase 3: Validate Against Multiple Sources

Cross-reference findings:

| Source Type | What to Extract |
|-------------|-----------------|
| Official docs | Current recommended approach |
| GitHub repo | Last commit date, open issues, activity |
| Dev blogs | Real-world adoption, pain points |
| Stack Overflow | Common problems, community sentiment |

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

## Example Workflow

**User**: "Set up Python linting for this project"

**Phase 1**: Python linting is in fast-moving-domains.md -> mandatory research

**Phase 2**: Search "python linting best practices 2025", "ruff vs flake8 2025"

**Phase 3**: Find that ruff has largely replaced flake8+isort+black for many projects

**Phase 4**:
```markdown
## Research Summary

**Question**: Python linting setup

**My initial assumption**: flake8 + isort + black (traditional stack)

**Current SOTA**: ruff (Rust-based, replaces flake8+isort, 10-100x faster)

**Delta**: ruff is now the dominant choice for new projects (2023-2024 shift)

**Recommendation**: Use ruff. Single tool, faster, drop-in compatible.

**Sources**:
- https://docs.astral.sh/ruff/: Official docs, actively maintained
- https://github.com/astral-sh/ruff: 35k+ stars, mass adoption
```

## Resources

See [references/fast-moving-domains.md](references/fast-moving-domains.md) for:
- List of domains requiring mandatory research
- Pre-built search queries by category
- Domain-specific volatility indicators
