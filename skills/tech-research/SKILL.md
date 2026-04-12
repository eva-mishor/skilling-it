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
  YES, marked ⚡ (highly volatile) -> Mandatory web research (Phase 2) + community search (Phase 2b)
  YES, fast-moving                 -> Mandatory web research (Phase 2)
  NO  -> Check: Has this domain changed significantly since 2023?
         YES -> Web research recommended
         NO  -> Training data likely sufficient
```

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

## Example Workflow

**User**: "Which LLM framework for a RAG pipeline?"

**Phase 1**: LLM frameworks is in fast-moving-domains.md under AI/ML Tooling (highly volatile) -> mandatory research + community search

**Phase 2**: Search "LLM RAG framework best practices 2026", "langchain vs llamaindex 2026"

**Phase 2b**: Community search (highly volatile domain):
- "RAG framework reddit 2026" -> find r/LocalLLaMA, r/LangChain threads
- "langchain llamaindex site:news.ycombinator.com"
- "switched from langchain to reddit 2026" -> migration stories
- Discard any results older than 6 months

**Phase 3**: Cross-reference official docs + GitHub activity + blogs + Reddit/HN sentiment

**Phase 4**:
```markdown
## Research Summary

**Question**: LLM framework for RAG pipeline

**My initial assumption**: LangChain (default since 2023)

**Current SOTA**: Fragmented. LlamaIndex for retrieval, LangGraph for orchestration. Growing "just use the SDK" movement on Reddit/HN.

**Delta**: Significant. "Just use LangChain" is outdated. Community sentiment (Reddit, last month) shows frustration with abstraction layers — many practitioners going framework-minimal.

**Recommendation**: Start with LlamaIndex for retrieval. Add LangGraph only if you need agentic orchestration. Consider raw SDK if your use case is simple.

**Sources**:
- docs.llamaindex.ai: Official RAG-focused framework
- r/LocalLLaMA (last month): Multiple threads on framework fatigue
- HN discussion (last month): "Why I stopped using LangChain" post with 200+ comments
```

## Resources

See [references/fast-moving-domains.md](references/fast-moving-domains.md) for:
- List of domains requiring mandatory research
- Pre-built search queries by category
- Domain-specific volatility indicators
