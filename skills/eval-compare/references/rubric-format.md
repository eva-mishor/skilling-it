# Rubric format for eval-compare

## The rule

A rubric is **exactly 3 binary criteria**. Not 2. Not 5. Three.

Each criterion is a yes/no question about the output. The total score per condition is 0–3, and the gap between conditions is what reveals whether the skill is helping.

## Why 3

- 1–2 criteria: too crude. A condition can hit 2/2 by coincidence. The signal is noisy.
- 4+ criteria: too slow to score live. The audience loses the thread. Diluted signal.
- 3 hits the sweet spot: enough to differentiate, few enough to score in 30 seconds, fits a single line per cell on a grid.

## What makes a good criterion

A good criterion is:

1. **Binary** — clearly yes or no, no partial credit. "Mentions ty" is good. "Discusses Astral well" is bad.
2. **Visible in the output** — you can spot it by reading, not by interpreting. "Cites a real URL" is good. "Demonstrates deep understanding" is bad.
3. **Recency-discriminating** — at least one criterion should be hard to satisfy from training alone. This is what makes the skill earn its place.
4. **Independent** — criteria shouldn't strongly correlate. "Mentions ty" and "mentions ty's status" are too correlated.

## Recipe

Use this structure for any recency-sensitive query:

1. **Coverage criterion** — does the output mention the key entities a current expert would mention? (e.g., for tooling questions: all relevant modern tools)
2. **Insight criterion** — does the output capture at least one non-obvious or counterintuitive fact that's only true post-cutoff? (this is the criterion the skill earns its keep on)
3. **Source criterion** — does the output cite recent verifiable sources (URLs, blog posts, docs)?

## Example: tech-research / Python type checkers

Query: "I'm starting a new Python project — what type checker should I use?"

Rubric:
1. **Coverage**: mentions both `ty` (Astral) AND `pyrefly` (Meta) — the two recent Rust-based checkers that complete the modern landscape.
2. **Insight**: captures at least one non-obvious 2026 fact: ty's low spec conformance (~15%) OR mypy now faster than pyright (mypyc-compiled) OR Astral acquired by OpenAI.
3. **Sources**: cites recent web sources (URLs to docs, blogs, comparisons).

## Example: tech-research / React state management

Query: "Best React state library for a new project?"

Rubric:
1. **Coverage**: mentions Zustand AND Jotai (both rose post-2023, in addition to Redux Toolkit).
2. **Insight**: captures at least one current tradeoff or shift — e.g., RSC implications, signals adoption status, why Redux Toolkit is no longer the default for new code.
3. **Sources**: cites at least one recent benchmark or production-experience post.

## Example: tech-research / database choice for a new SaaS

Query: "What database should I pick for a new B2B SaaS in 2026?"

Rubric:
1. **Coverage**: mentions Postgres AND at least one modern serverless option (Neon, Supabase, PlanetScale, Turso, etc.).
2. **Insight**: captures one operational reality (e.g., Vercel acquired Turso, PlanetScale dropped free tier, Neon's branching behavior, etc.).
3. **Sources**: cites at least one recent production post-mortem, pricing change announcement, or comparison.

## Adapting for non-research skills

If the target skill is not research-flavored (e.g., a runbook, code-review, scaffolding skill), the recipe changes. Examples:

### code-review skill

1. **Coverage**: identifies all 3 planted issues (you'd seed the test code with known issues).
2. **Insight**: catches at least one issue beyond the planted ones.
3. **No false positives**: doesn't flag non-issues as issues.

### deploy-checklist skill

1. **Coverage**: addresses all required pre-deploy steps for the project type.
2. **Order correctness**: steps are in valid dependency order (e.g., tests before push).
3. **Specificity**: instructions are project-specific, not generic boilerplate.

For non-research skills, **do not use the default conditions A/B/C/D from eval-compare directly** — the no-skill conditions don't make sense for skills that produce structured operational output. Either:

- Use only conditions C and D (skill on big vs small model)
- Or replace conditions A/B with different skill variants (e.g., "skill v1 vs skill v2")

## What NOT to do

- **Don't use vague criteria**: "good answer," "well structured," "comprehensive."
- **Don't use criteria the model can game**: "mentions sources" without checking they're real URLs.
- **Don't pile on criteria when you can't decide**: the answer to "should this be a criterion?" is usually no.
- **Don't write criteria after seeing the outputs.** That's fitting the rubric to the data — anti-rigor. Write the rubric BEFORE running.
