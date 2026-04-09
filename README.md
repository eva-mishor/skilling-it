# My Claude Skills

Custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), organized by domain.

## Skills

### Research

| Skill | What it does |
|-------|-------------|
| **tech-research** | Research current state-of-the-art before implementing technical decisions. Prevents stale training-data recommendations for tooling, libraries, and architectural patterns. |

### Session & Workflow

| Skill | What it does |
|-------|-------------|
| **mastermind-session** | Interactive roundtable with 5-10 thought leaders to solve business, strategic, or personal challenges. 5-phase structured process. |
| **consolidate** | Post-session learning extraction. Scans conversation for insights and routes them to the correct persistent storage (memory, CLAUDE.md, rules). |
| **wait-what** | Session checkpoint — structured summary of what was done, why, and what's next. Use before committing or when you lose track. |
| **end-of-day** | Comprehensive end-of-day work summary with metrics and reflection. |
| **tech-to-social** | Turn technical work (commits, PRs, bug fixes, architecture decisions) into LinkedIn posts. 2-3 draft variations per request with hook patterns, tone selection, and series planning. |

### Dev Workflow Guards

| Skill | What it does |
|-------|-------------|
| **preclear** | Pre-session-end checklist — ensures documentation, learning capture, and intelligent branch recommendations. |
| **prepush** | Comprehensive validation before pushing to remote — ensures everything is tidy and documented. |
| **repo-tidy-up** | Periodic repo health checklist — clean up branches, dependencies, stale code, memory, and docs. |

### Meta & Maintenance

| Skill | What it does |
|-------|-------------|
| **skill-creator** | Guide for creating new skills. 6-step process with progressive disclosure patterns and Python utilities for init/validate/package. |
| **bouncer** | Security audit for vetting third-party skills, plugins, hooks, and MCP servers before installation. 4-phase process covering 8 threat categories. |
| **spring-clean** | Audit and optimize Claude Code session context budget. Covers MCP servers, plugins, MEMORY.md, CLAUDE.md, custom agents. |

## Install a skill

```bash
# Install any skill directly from this repo
claude install-skill https://github.com/eva-mishor/my-claude-skills/tree/main/skills/<skill-name>
```

### Examples

```bash
claude install-skill https://github.com/eva-mishor/my-claude-skills/tree/main/skills/bouncer
claude install-skill https://github.com/eva-mishor/my-claude-skills/tree/main/skills/consolidate
```

Some skills are also available as standalone gists:
- [bouncer](https://gist.github.com/eva-mishor/839810fe18d1e8e66cf4a9496ea307e8)
- [consolidate](https://gist.github.com/eva-mishor/2ca6bfe8f3a2ee5170e996b292340ce0)

## Creating new skills

Use the `skill-creator` skill, or manually:

```bash
# Initialize a new skill
python skills/skill-creator/scripts/init_skill.py my-new-skill --path skills/

# Validate
python skills/skill-creator/scripts/quick_validate.py skills/my-new-skill
```

## Repo structure

```
skills/
├── bouncer/               # Security audit for skills/plugins
├── consolidate/           # Post-session learning extraction
├── end-of-day/            # End-of-day work summary
├── mastermind-session/    # Expert roundtable sessions
├── preclear/              # Pre-session-end checklist
├── prepush/               # Pre-push validation
├── repo-tidy-up/          # Repo health & cleanup
├── skill-creator/         # Skill creation guide + tools
├── spring-clean/          # Context budget optimizer
├── tech-research/         # State-of-the-art tech research
├── tech-to-social/        # Technical work → LinkedIn posts
└── wait-what/             # Session checkpoint summaries
```

## License

Skills may have individual licenses. See each skill's directory for details.
