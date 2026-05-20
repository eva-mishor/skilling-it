# Claude Skills by Eva Mishor

> Neuroscience ↔ Data science ↔ Building with coding agents daily

Production-tested skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — patterns I use daily across consulting, research, and shipping AI products. Open-source so you can install them, fork them, or learn from them.

I'm a senior data scientist and AI engineer (Ph.D. Neuroscience, former Head of AI Research). Most of these skills started as friction points I kept hitting while building agent systems with Claude — they encode the workflow I now run.

## Practice volume

These skills are not theory. They come from real daily use:

| | |
|---|---|
| Tokens consumed via Claude Code | **4.1B** (~40M/day for 100 days) |
| Coding hours | 823 |
| Claude Code sessions | 1,338 |
| Lines of code shipped (across 11 repos) | 429K |
| Skills + custom agents created | 26 + 21 |

*Source: usage telemetry, Jan–Apr 2026 (~3.5 months on file).*

## Talks

→ **Full talks archive: [github.com/eva-mishor/talks](https://github.com/eva-mishor/talks)**

Selected, most relevant to this repo:

- **SKilling it! — Don't build agents. Build skills.** — AI & Vibe Coding Bootcamp for Builders (FUSION × Google for Startups × Ottomat), Tel Aviv, Apr 2026. [→ slides + notes](docs/talks/) · [→ deck PDF](docs/talks/skilling-it-fusion-2026.pdf)
- **Vibe Coding from Scratch — Lessons Learnt** — *Intelligence at Work*, Nov 2025. [event](https://luma.com/tsuna6vb)

---

## Featured skill — `pr-axes-review`

A 14-axis ship-time review that runs after `gh pr create`. The key idea: **rank by surprise risk, not severity.** High-severity obvious bugs are usually Tier 2 or 3 — the team already sees them. The Tier 1 stuff is the cost baseline shift nobody warned analytics about, the rename that broke a downstream consumer, the model-coverage gap on DEV.

Diff-anchored discipline (every claim points to a file:line, generic warnings rejected), trivial-PR bypass heuristic, chat-only output (no bot-shaped comments on PRs). 14 axes covering user trust, cost, coverage, conversion, accessibility, i18n, rollback, analytics, CI, cross-repo sync, infra, data/schema, security, config side-effects.

→ [skills/pr-axes-review](skills/pr-axes-review/)

If you only look at one skill in this repo, look at this one.

---

## All skills

### Research

| Skill | What it does |
|-------|-------------|
| **tech-research** | Research current state-of-the-art before implementing technical decisions. Prevents stale training-data recommendations for tooling, libraries, and architectural patterns. Includes Phase 0 staleness checks and community-recency search for volatile domains. |

### Session & Workflow

| Skill | What it does |
|-------|-------------|
| **consolidate** | Post-session learning extraction. Scans the conversation for insights and routes each to the correct persistent store (memory, CLAUDE.md, rules). |
| **wait-what** | Session checkpoint — structured summary of what was done, why, and what's next. Use before committing or when you've lost track. |
| **end-of-day** | End-of-day work summary with metrics and reflection. |
| **tech-to-social** | Turn technical work (commits, PRs, bug fixes, architecture decisions) into LinkedIn posts. 2-3 draft variations per request with hook patterns, tone selection, and series planning. |

### Dev Workflow Guards

| Skill | What it does |
|-------|-------------|
| **pr-axes-review** ★ | 14-axis ship-time review fired automatically after `gh pr create`. Tier by surprise risk, not severity. Diff-anchored, chat-only. *(Featured above.)* |
| **preclear** | Pre-session-end checklist — documentation, learning capture, and intelligent branch recommendations. |
| **prepush** | Comprehensive validation before pushing to remote. |
| **repo-tidy-up** | Periodic repo health checklist — branches, dependencies, stale code, memory, and docs. |

### Meta & Maintenance

| Skill | What it does |
|-------|-------------|
| **skill-creator** | Guide for creating new skills. 6-step process with progressive disclosure patterns and Python utilities for init/validate/package. |
| **bouncer** | Security audit for vetting third-party skills, plugins, hooks, and MCP servers before installation. 4-phase process covering 8 threat categories with CVE-informed attack patterns. |
| **spring-clean** | Context-budget engineering for long agent sessions. Harness-agnostic (Claude Code · Codex CLI · Gemini CLI · opencode). Measures baseline startup consumption, then audits the four context-eaters — MCP servers, plugins, persistent memory, project instructions, subagents — and applies fixes in order of token impact. Target: <25% of budget consumed at session start. Turns "why is this session sluggish" into a defensible cleanup PR. |
| **eval-compare** | Talk demo: a 4-condition one-shot comparison harness for skills (Opus/Sonnet × skill-on/off). No repeats, so not a real eval — for serious skill evaluation use Claude Code's `--evals` flag and the SkillsBench infrastructure. Kept here as the worked example from my Nov 2025 Vibe Coding talk. |

---

## Install a skill

```bash
# Install any skill directly from this repo
claude install-skill https://github.com/eva-mishor/skilling-it/tree/main/skills/<skill-name>
```

### Examples

```bash
claude install-skill https://github.com/eva-mishor/skilling-it/tree/main/skills/pr-axes-review
claude install-skill https://github.com/eva-mishor/skilling-it/tree/main/skills/bouncer
```

Some skills are also published as standalone gists:

- [bouncer](https://gist.github.com/eva-mishor/839810fe18d1e8e66cf4a9496ea307e8)
- [consolidate](https://gist.github.com/eva-mishor/2ca6bfe8f3a2ee5170e996b292340ce0)

## Creating new skills

Use the `skill-creator` skill, or manually:

```bash
python skills/skill-creator/scripts/init_skill.py my-new-skill --path skills/
python skills/skill-creator/scripts/quick_validate.py skills/my-new-skill
```

## Repo structure

```
skills/
├── bouncer/          # Security audit for skills/plugins
├── consolidate/      # Post-session learning extraction
├── end-of-day/       # End-of-day work summary
├── eval-compare/     # Talk-demo A/B harness (no repeats — not a real eval)
├── preclear/         # Pre-session-end checklist
├── prepush/          # Pre-push validation
├── pr-axes-review/   # 14-axis ship-time PR review (featured)
├── repo-tidy-up/     # Repo health & cleanup
├── skill-creator/    # Skill creation guide + tools
├── spring-clean/     # Context budget optimizer
├── tech-research/    # State-of-the-art tech research
├── tech-to-social/   # Technical work → LinkedIn posts
└── wait-what/        # Session checkpoint summaries
```

## Related work

- [data-agent](https://github.com/eva-mishor/data-agent) — active-learning loop applying the same evaluate-score-iterate methodology to training-data acquisition rather than LLM-tooling evaluation. The methodological family is shared.

## License

MIT — see [LICENSE](LICENSE). Individual skills may carry additional notices; check each skill's directory.

## About

Eva Mishor, Ph.D. — senior data scientist & AI engineer.
[LinkedIn](https://www.linkedin.com/in/eva-mishor) · [GitHub](https://github.com/eva-mishor) · eva.mishor@gmail.com
