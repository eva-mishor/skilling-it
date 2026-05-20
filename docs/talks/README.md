# Talks

## SKilling it! — Don't build agents. Build skills.

**AI & Vibe Coding Bootcamp for Builders** · FUSION × Google for Startups × Ottomat · Tel Aviv · Apr 29, 2026

Slides: [skilling-it-fusion-2026.pdf](skilling-it-fusion-2026.pdf)

### What the talk covers

- **Mental model.** Models = CPU. Harness (Claude Code / Codex CLI / Gemini CLI / opencode / Copilot) = OS. Skills = the apps. Don't tune the CPU. Don't rewrite the OS. Install the right apps.
- **The empirical case** for skills, from SkillsBench: +16.2pp average pass-rate gain with curated skills across 84 tasks × 11 domains × 7,308 trajectories. Biggest lifts: Healthcare (+51.9), Manufacturing (+41.9), Cybersecurity (+23.2).
- **Five reasons to use skills**: determinism, the right guidelines, centralized learning, team scale, context efficiency (progressive disclosure).
- **Decision filter** for when you actually need a skill vs. when CLAUDE.md, a hook, or an MCP server is the right tool.
- **Three concrete skill types** you can ship this week: business process, runbook, library/API reference.
- **The downside**: how skills can break. Self-generated skills score *−1.3pp* vs no skill; comprehensive AGENTS.md docs beat a generic skill (Vercel/Next.js result: 79% with default skill vs 100% with AGENTS.md). Skills are a tool, not a religion. Measure.
- **Four authoring rules**: build a gotchas section, avoid railroading, one type per skill, evals (treat skills like code — red/green/refactor).
- **Cost play**: smaller cheaper model + skill (Haiku 4.5 + skills, 27.7%) beats bigger costlier model running cold (Opus 4.5 no skills, 22.0%). SkillsBench Finding 7.

### Practice volume behind the talk

| | |
|---|---|
| Tokens consumed via Claude Code | **4.1B** (~40M/day for 100 days) |
| Coding hours | 823 |
| Claude Code sessions | 1,338 |
| Lines of code shipped (across 11 repos) | 429K |
| Skills + custom agents created | 26 + 21 |

Source: usage telemetry on file Jan – Apr 2026 + cumulative code across 11 repos.

### Resources cited in the deck

- Anthropic — How We Use Skills · [Claude Code Documentation — Skills](https://docs.anthropic.com/en/docs/claude-code) · The Complete Guide to Building Skills · Anthropic — Equipping agents for the real world (Oct 2025)
- [SkillsBench](https://skillsbench.ai) — empirical evaluation
- [Vercel — AGENTS.md outperforms Skills in our evals](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals)
- [Agent Skills Open Standard](https://agentskills.io)
- Built into Claude Code: `/skill-creator` · `/superpowers:writing-skills`

### Companion talk

**Vibe Coding from Scratch — Lessons Learnt** — *Intelligence at Work*, Nov 2025. [event](https://luma.com/tsuna6vb)
