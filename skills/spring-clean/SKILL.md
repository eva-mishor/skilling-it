---
name: spring-clean
description: Audit and optimize the context budget of an agent session. Use when sessions feel sluggish, context compacts too early, or after installing new plugins/MCP servers/subagents. Harness-agnostic — works for Claude Code, Codex CLI, Gemini CLI, opencode, and other agentic coding harnesses with progressive disclosure. Triggers on "audit context", "optimize context", "context budget", "session overhead".
---

# Spring Clean — Session Budget Optimizer

Measure, identify, and reduce startup context overhead so more of your context budget goes to actual work, not to scaffolding.

**Target**: <25% of the model's context window consumed at session start.

## Harness mapping

Spring-clean is a methodology, not a tool — the steps are the same across harnesses, only the paths and inspection commands differ. Replace the placeholder names below with the conventions for your harness:

| Concept | Claude Code | Codex CLI / opencode | Gemini CLI |
|---|---|---|---|
| Context inspector | `/context` | inspect bundle / debug | inspect bundle |
| Project instructions (always-loaded) | `CLAUDE.md` | `AGENTS.md` | `GEMINI.md` |
| Persistent memory file | `MEMORY.md` | (often `AGENTS.md` doubles) | (varies) |
| Project settings | `.claude/settings.json` | `.codex/`, `.opencode/`, etc. | `.gemini/` |
| Global settings | `~/.claude/settings.json` | per-harness home dir | per-harness home dir |
| Skills/plugins dir | `.claude/skills/`, `.claude/plugins/` | varies | varies |
| Subagents | `.claude/agents/` | varies | varies |

When this skill says "your project-instructions file," substitute whichever file your harness loads at startup. The four context-eaters — instructions, memory, plugins/MCP, subagents — exist in every modern coding agent.

## Phase 1 — Measure baseline

1. Open a fresh session in your harness and run its context-inspection command (see table above). Record token counts by category.
2. Note total context budget and the percentage consumed at session start.
3. Record counts for: system prompt, MCP/tool definitions, custom agents, persistent memory files, installed skills/plugins, project-instructions file(s).
4. Save these numbers — this is your baseline for comparison in Phase 4.

If your harness does not expose a `/context`-style inspector, estimate by summing the token counts of all files the harness loads at startup. A rough character-count → token estimate (`chars / 4`) is close enough for prioritization.

## Phase 2 — Identify waste

Audit each category for unnecessary overhead:

### MCP servers / tool integrations
- List all MCP servers (or equivalent tool bridges) in both project and global settings.
- Flag servers with 10+ tools that aren't used in this project. (Common offender: Linear/Jira/Slack tools on a project that doesn't touch them.)
- Flag servers enabled globally that only apply to specific projects — move them to project scope, or disable.

### Plugins / skill collections
- List enabled plugins (global + project).
- Flag plugins loading 5+ skills/tools you don't actively invoke in this project.
- Check for skill overlap: do plugin skills duplicate project-local skills? If so, pick one and disable the other.

### Persistent memory file (`MEMORY.md` / harness equivalent)
- Stale entries: old branch status, merged PRs, resolved issues, last quarter's context.
- Duplication with the project-instructions file (rules stated in both places).
- Narrative where a one-liner suffices.
- Most harnesses truncate memory files past some line count — prioritize what's above the fold.

### Project-instructions file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`)
- Rules the agent already follows by default — don't restate behaviors the model has out-of-the-box.
- Verbose formatting where semicolons or pipe-delimiters would work.
- Content that belongs in on-demand files (referenced lazily, e.g. via `@file` imports in Claude Code) rather than always-loaded.
- Multiple always-load project docs (`CLAUDE.md`, `TESTING.md`, `README.md`, `STYLEGUIDE.md`) — do all need to be always-loaded? Often one of them can become an on-demand reference.

### Custom subagents
- Check if any always-loaded subagents could be on-demand skills instead.
- Flag agents with overlapping responsibilities — consolidate or split cleanly.
- Long agent prompts add to startup; trim ruthlessly.

## Phase 3 — Optimize

Apply fixes in order of token impact (biggest wins first):

1. **Disable unused MCP servers / tool integrations.** In your harness's settings file, set `"disabled": true` per server. This is usually the single biggest win — MCP tool catalogs are token-heavy.
2. **Disable unused plugins / skill collections.** Edit project or global settings.
3. **Compress the project-instructions file.** Semicolons within rules; line breaks between rules; lazy imports for reference material; remove restatements of default-behavior. The goal: every line earns its place.
4. **Prune the persistent memory file.** Remove stale entries, deduplicate with the project-instructions file, compress narratives to one-liners.
5. **Move verbose docs to on-demand.** Large instruction files (`TESTING.md`, `USAGE_GUIDE.md`, `DEPLOY.md`) can be lazy-loaded via your harness's reference mechanism rather than always-loaded.

**Note**: compression only helps always-loaded files. Skip for on-demand files like `todo.md` or reference docs that are pulled in lazily — those don't cost startup tokens.

## Phase 4 — Verify

1. Run the context inspector again in a fresh session.
2. Compare token counts against your Phase 1 baseline.
3. Calculate reduction percentage.
4. If still >25% startup overhead, revisit Phase 2 for deeper cuts.
5. Report summary: before/after tokens, percentage saved, list of changes made. (If you're running this as a PR, this summary belongs in the PR body.)

## Quick reference: common wins

| Source | Typical savings | Effort |
|---|---|---|
| Disable an unused MCP server | 2–8K tokens | 30 sec |
| Disable an unused plugin / skill bundle | 1–5K tokens | 30 sec |
| Compress project-instructions formatting | 2–6K tokens | 10 min |
| Prune stale memory file | 0.5–2K tokens | 5 min |
| Move large docs to on-demand | 3–10K tokens | 5 min |

## Red flags

| Thought | Reality |
|---|---|
| "My harness handles this automatically." | None of them do. Startup context is a developer-managed budget; no harness auto-prunes your MEMORY.md or disables your unused MCP servers. |
| "I'll just compress everything." | Compression only helps always-loaded files. On-demand files don't cost startup tokens; compressing them is wasted effort. |
| "25% is too aggressive a target." | The target is a forcing function. If you can't get under 25%, the next 5% is where the most painful cuts are — and usually the most valuable. |
| "I'll wait until it actually breaks." | By the time it "actually breaks" (compaction mid-task, lost context, dropped instructions), you've already lost a session of work. Run this proactively after every new plugin/MCP install. |
