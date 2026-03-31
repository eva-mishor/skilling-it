# Writing Claude Code Skills — 10-Minute Webinar

> **Format:** Presenter-led walkthrough with live demo moments
> **Audience:** Claude Code users who want to go beyond one-off prompts
> **Goal:** By the end, the audience understands what skills are, when they need one, how to write one well, and how to share it

---

## SLIDE 1: The Problem (0:00–1:00)

**Hook:**

"You're using Claude Code every day. You've got a workflow that works — maybe it's how you review PRs, how you debug production issues, how you onboard new hires to your codebase.

And every time you start a new session, you explain it again. From scratch.

CLAUDE.md helps — it gives Claude persistent context about your project. But CLAUDE.md is passive. It's always loaded, it applies to everything, and it doesn't have structure. It can't say: 'When the user asks for X, follow this specific multi-step process, read these reference files at the right moments, produce these deliverables, and enforce these quality gates.'

That's what skills do."

**Key point:** CLAUDE.md = persistent project context (always loaded). Skills = structured processes (loaded on demand, with their own files, tools, and logic).

---

## SLIDE 2: What Is a Skill? (1:00–2:00)

"A skill is a folder. At minimum, it contains a SKILL.md file with YAML frontmatter and markdown instructions. But the power is in the folder — you can include reference docs, templates, scripts, even config files."

Show the anatomy:

```
.claude/skills/my-skill/
├── SKILL.md              # Core instructions (loaded when triggered)
├── references/            # Detailed docs Claude reads on demand
│   └── framework.md
├── assets/                # Templates to copy and fill
│   └── report-template.md
└── scripts/               # Executable tools Claude can run
    └── validate.sh
```

"SKILL.md is the entry point. It tells Claude: here's the process, here's when to read deeper files, here's the quality bar. The references/ folder is where your detailed knowledge lives — Claude reads those files only when it reaches the step that needs them. Assets are deliverable templates. Scripts are tools Claude can run."

**Key point:** A skill is not a big prompt. It's a structured toolkit with progressive disclosure — Claude loads depth on demand, not all at once.

---

## SLIDE 3: When Do You Need a Skill? (2:00–3:30)

"Not everything needs to be a skill. Here's the decision filter:"

**You need a skill when:**
- You repeat the same multi-step process across sessions
- The process has decision points, gates, or conditional logic
- It needs reference material that's too large for CLAUDE.md
- Other people should be able to run the same process
- You want Claude to produce specific deliverables in a specific format

**You don't need a skill when:**
- A CLAUDE.md instruction covers it ("always use conventional commits")
- It's a one-off task
- It's general knowledge Claude already has

"Think of it this way: CLAUDE.md is your team's coding standards. A skill is your team's playbook for a specific operation."

**The six skill types** (show briefly, don't dwell):

| Type | Example |
|------|---------|
| Business Process | Multi-step workflow ("run our release process") |
| Data Fetching & Analysis | Connect to data, produce reports |
| Code Quality & Review | Enforce standards, run structured reviews |
| Library/API Reference | How to use a specific tool correctly |
| Scaffolding & Templates | Generate boilerplate from requirements |
| Runbook | Symptom → investigation → structured report |

"If your skill straddles two types, it's probably confused about what it does. Pick one."

---

## SLIDE 4: The SKILL.md File (3:30–5:00)

"Let's look at what goes in the file."

### Frontmatter

```yaml
---
name: deploy-checklist
description: "Run pre-deploy validation for production releases. Triggers on 'deploy to prod', 'release checklist', 'pre-deploy check'."
allowed-tools: Bash, Read, Grep
---
```

"Three things to get right in frontmatter:"

**1. The `name` field is your slash command.**
Whatever you put here becomes `/deploy-checklist`. Kebab-case, lowercase.

**2. The `description` is for Claude, not for humans.**
Claude scans every skill description at session start to decide which one matches your request. This is a trigger specification, not a summary.

Bad: "A tool for deployments"
Good: "Run pre-deploy validation for production releases. Triggers on 'deploy to prod', 'release checklist', 'pre-deploy check'."

Front-load the use case. Include trigger phrases. Be specific.

**3. `allowed-tools` controls what Claude can do without asking.**
If your skill only needs to read files, restrict it. Don't give write access to a read-only audit skill.

### Other useful frontmatter

```yaml
disable-model-invocation: true    # Only manual /slash-command, no auto-trigger
user-invocable: false             # Only Claude can invoke (helper skill)
context: fork                     # Run in isolated subagent
paths: "src/**/*.ts"              # Only activate for matching files
```

### The body

"The markdown body is your process. But here's the most important rule:"

**Don't state the obvious.** Claude already knows how to code, how to read files, how to run tests. Focus on what pushes Claude out of its default behavior — domain-specific gotchas, your org's conventions, the things it consistently gets wrong.

---

## SLIDE 5: Progressive Disclosure — The Key Pattern (5:00–6:30)

"This is the single most important structural decision in skill design."

"A 2,000-line SKILL.md is not a skill. It's a context bomb. Claude loads the entire thing into its context window when the skill triggers. Every token you put in SKILL.md is a token that's always present, whether it's relevant to the current step or not."

**The pattern:**

SKILL.md stays compact — process flow, decision points, gate criteria, gotchas, and a directory listing of what's available. It says things like:

> "Read `references/scoring-framework.md` for the full threshold table."

> "Use the template in `assets/report-template.md` for the deliverable."

Claude reads those files only when it reaches that step. Phase 1 doesn't burn tokens on Phase 6 content.

**What goes where:**

| Location | Content | When loaded |
|----------|---------|-------------|
| SKILL.md | Process, logic, gates, gotchas | Always (on trigger) |
| references/ | Detailed frameworks, data, rubrics | On demand (at specific steps) |
| assets/ | Deliverable templates | When producing output |
| scripts/ | Executable tools | When Claude needs to run them |

"Think of SKILL.md as the table of contents and decision logic. The chapters live in references/."

---

## SLIDE 6: Writing Rules That Matter (6:30–8:00)

"Five rules. These are the ones that separate skills that work from skills that don't."

### 1. Build a gotchas section

"The highest-signal content in any skill is what goes wrong. Every time Claude makes a mistake while using the skill, add it as a gotcha."

```markdown
## Gotchas

| Mistake | Fix |
|---------|-----|
| Asks about pricing before establishing pain | Always: pain → capabilities → pricing |
| Cites training data as market research | Flag: "This is from training data, not live research" |
| Skips the gate and advances anyway | Re-read gate criteria. If it doesn't pass, say so. |
```

"This section is a living document. It grows from real failures. After 10 sessions, your gotchas section is the most valuable part of the skill."

### 2. Avoid railroading

"Give Claude principles and constraints, not a script to follow verbatim. Rigid step-by-step instructions break the moment the situation doesn't match. And it will never match exactly."

### 3. Do the work first, then red/green/refactor

"Never write a skill abstractly. The creation process is:

1. **Live session** — run the process with a real case, making real decisions
2. **First draft** — extract what worked into SKILL.md
3. **Red** — what's wrong? What gaps does the skill have? (Use `/skill-creator` to scaffold, then test it on a second case and watch it fail)
4. **Green** — fix the gaps. Add the gotchas you just discovered.
5. **Refactor** — clean up, extract references, align with your other skills' structure
6. **Data audit** — verify every factual claim has a source
7. **Ship**

You'll catch structural gaps at step 3 that are invisible from the design phase. One of my skills missed 5 structural issues — no rescue loop, no kill hierarchy, no onboarding flow — that only surfaced when I ran it on a real case."

### 4. Audit every factual claim

"After writing a skill, grep for specific numbers, example classifications, and 'X is Y' assertions. Search for sources. If unsourced, either find data or reframe as methodology. One first draft I wrote had 8 unsourced claims presented as facts. All wrong."

### 5. One type, one skill

"If your skill is trying to be both a code reviewer AND a scaffolding tool, split it. Skills that straddle types are confused about what they do and do neither well."

---

## SLIDE 7: Sharing and Installing (8:00–9:00)

"Skills live in one of three places, and where you put them determines who gets them."

| Scope | Path | Who gets it |
|-------|------|-------------|
| Personal | `~/.claude/skills/<name>/` | Just you, all projects |
| Project | `.claude/skills/<name>/` | Everyone on this repo |
| Enterprise | Managed settings | All org users |

### Sharing via GitHub

"Put your skill in a public repo. Anyone can install it with one command:"

```bash
claude install-skill https://github.com/you/repo/tree/main/skills/my-skill
```

"That's it. It clones into their personal skills directory and it's available in every session."

### Project skills

"Commit `.claude/skills/` to your repo. Every teammate who clones the repo gets the skills automatically. No install step. This is how you standardize processes across a team — your deploy checklist, your PR review process, your incident response runbook."

---

## SLIDE 8: Recap + What to Build First (9:00–10:00)

"Here's the cheat sheet:"

**A skill is a folder**, not a prompt. SKILL.md + references + assets + scripts.

**Progressive disclosure** keeps context lean. SKILL.md is the router; references are the depth.

**The description triggers the skill.** Write it for Claude, not for humans. Include trigger phrases.

**Gotchas are your best content.** Build them from real failures over time.

**Do the work first.** Run the process live, then codify. Never design in the abstract.

**Start with pain.** Your first skill should be the process you're most tired of re-explaining. The one where you think "I wish Claude just knew how we do this." That's your skill.

**You don't have to start from scratch.** Claude Code has a built-in `/skill-creator` command that scaffolds a new skill for you — frontmatter, directory structure, hook registration. Describe what you want the skill to do in plain language and it generates the starting structure. Then refine with the rules from this talk.

"Try it right now: open Claude Code, type `/skill-creator`, describe the process you're most tired of repeating. It'll generate the skeleton. Then do the work live with that skill, fix what breaks, build up the gotchas section. That's the loop."

---

## SPEAKER NOTES

### Demo moments (if doing live)

1. **Slide 2:** Show a real skill folder structure in the terminal (`tree .claude/skills/`)
2. **Slide 4:** Show frontmatter → trigger the skill with a natural language request → show it activating
3. **Slide 5:** Show Claude reading a reference file mid-process (the "Read references/..." moment in a transcript)
4. **Slide 6:** Live `/skill-creator` — describe a process, show the generated scaffold, then show how you'd refine it through the red/green/refactor cycle
5. **Slide 7:** Live `claude install-skill` from a GitHub URL

### Audience Q&A prompts

- "What process do you repeat most often with Claude?" (leads into "that's your first skill")
- "How many of you have a CLAUDE.md over 200 lines?" (leads into "some of that should be skills")

### Common questions to prepare for

**"How is this different from system prompts?"**
System prompts are static text injected at the start. Skills are structured toolkits — they have folders, progressive disclosure, executable scripts, conditional logic, and they trigger contextually instead of loading always.

**"Can skills call other skills?"**
Not directly, but a skill can reference another skill by name ("Use the `validation-talk` skill to generate the questionnaire") and Claude will invoke it.

**"What about context window limits?"**
That's exactly why progressive disclosure matters. A monolithic 5,000-line skill burns context on content that's irrelevant to the current step. A well-structured skill loads 200 lines initially and reads deeper files only when needed.

**"Can I use skills with other AI tools?"**
Skills follow the Agent Skills open standard (agentskills.io). The SKILL.md format works with compatible tools, though frontmatter fields and features vary by platform.

**"How do I know if my skill is working well?"**
Use it. The gotchas section is your feedback loop. If you're adding gotchas every session, the skill is learning. If you stop adding them, it's mature. For high-stakes skills, build an eval suite — but most teams should start with "use it and fix what breaks."
