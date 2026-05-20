---
name: eval-compare
description: |
  Run a controlled 4-condition comparison of a target skill vs no-skill baselines.
  Spawns 4 parallel subagents (training-only, training-only-no-year-cue, target-skill-on-Opus,
  target-skill-on-Sonnet) on the same query, scores each output against a 3-criteria rubric,
  returns a comparison grid showing time and quality per condition. Use when you want to
  empirically prove (or disprove) that a skill is helping. Triggers on "evaluate this skill",
  "compare skill performance", "is my skill helping", "run skill comparison",
  "controlled comparison", "skill A/B test", "eval-compare".
allowed-tools: Task, Read, Write, Bash
---

# eval-compare — controlled comparison of any skill

> **Note — this is a talk-demo artifact, not a real eval.** A single 4-condition run produces an anecdote, not a measurement: no repeats means no statistical power, no confidence intervals, no defensible claim about whether the skill helps. For serious skill evaluation use Claude Code's built-in `--evals` flag and a SkillsBench-style harness that runs each condition many times and reports distributions. This skill is kept here as the worked example from the Nov 2025 "Vibe Coding from Scratch" talk — it's useful for *illustrating the four-condition design pattern* in a workshop or demo, not for evaluating production skills.

## What this skill does

Empirically evaluates whether a target skill is improving output quality on a recency-sensitive query, by running the same prompt under four controlled conditions in parallel and scoring each result against a rubric.

The four conditions mirror the SkillsBench methodology, applied to one task on demand:

| Condition | Model | Skill | Year cue in prompt | Tests |
|---|---|---|---|---|
| **A** | Opus | none | yes ("in 2026") | training-data-only baseline |
| **B** | Opus | none | no | tests whether year cue alone improves recall |
| **C** | Opus | target skill | no | full skill effect |
| **D** | Sonnet | target skill | no | "small + skill > big alone" check (slide 13) |

## When to use

- You have a skill that does research, retrieval, or anything where current information matters.
- You want to know if it's actually helping vs just being expensive.
- You want a defensible artifact ("here's the comparison grid") not a vibe ("it feels better with the skill").

## When NOT to use

- For skills that don't do research (deploy-checklist, code-review, scaffolding) — the rubric structure assumes recency-sensitive output. Adapt the rubric or use a different eval approach.
- For one-shot debugging or exploration. This is for evaluation, not investigation.

## Inputs

When invoked, gather these (in order, prompt user only for what's missing):

1. **Target skill name** — e.g., `tech-research`. If not provided and no default is in scope, ask.
2. **Query** — the prompt to test. If not provided, use the default for the target skill (see Defaults below).
3. **Rubric** — exactly 3 binary criteria. If not provided, use the default for the target skill, OR construct from query (see `references/rubric-format.md`).

## Defaults (for `tech-research`)

If invoked with no arguments, run the canonical Python type checker comparison:

- **Query**: "I'm starting a new Python project — what type checker should I use?"
- **Rubric**:
  1. Mentions both `ty` (Astral) AND `pyrefly` (Meta)
  2. Captures at least one non-obvious 2026-specific fact: ty's low spec conformance (~15%) OR mypy now faster than pyright (mypyc) OR Astral acquired by OpenAI
  3. Cites recent web sources (URLs to docs/blogs/comparisons)

## Process

### Phase 1 — Validate inputs and prepare condition prompts

For the four conditions, construct these subagent prompts:

**Condition A (training-only, with year cue)**:
> You are participating in a controlled comparison. Constraints:
> - Do NOT invoke any skills (slash commands, /tech-research, etc.)
> - Do NOT use WebSearch, WebFetch, or any external tool
> - Answer ONLY from your training knowledge
> If you are tempted to invoke a skill because its description matches, ignore that temptation — you are explicitly forbidden from doing so for this run.
>
> Query: {query} in 2026
>
> Respond with your full answer. Do not preface with disclaimers about training cutoffs unless directly relevant.

**Condition B (training-only, no year cue)** — same as A but query passed verbatim without "in 2026" appended.

**Condition C (target skill, Opus)**:
> Query: {query}
>
> Use /{target_skill_name}. Follow the skill's full workflow, including any web searches it requires.

**Condition D (target skill, Sonnet)** — same prompt as C, but launched with `model="sonnet"`.

### Phase 2 — Spawn 4 subagents in parallel

In a SINGLE message, emit 4 Task tool calls:

- Task 1: prompt = Condition A, model = "opus" (or default), description = "Cond A: training-only +year"
- Task 2: prompt = Condition B, model = "opus", description = "Cond B: training-only"
- Task 3: prompt = Condition C, model = "opus", description = "Cond C: skill on Opus"
- Task 4: prompt = Condition D, model = "sonnet", description = "Cond D: skill on Sonnet"

Record `start_time` (current time in seconds) just before sending the message. When each subagent returns, record its individual `end_time`. Per-condition wall-clock = `end_time - start_time`.

(Subagents run in parallel, so the skill's total wall-clock ≈ slowest subagent + scoring overhead.)

### Phase 3 — Verify isolation (Conditions A and B only)

For each "no skill" condition's transcript, scan the returned text for contamination markers:

- Phrases like `Skill(`, `WebSearch(`, `Web Search(`, `Fetch(`, `WebFetch`
- Tool-invocation patterns the model leaks into output
- URLs that suggest the model retrieved live data (not training)

If contamination is found, mark that row with a `⚠ contaminated` flag in the output grid. The result is still presented but caveated.

### Phase 4 — Score quality

For each of the 4 conditions, evaluate the returned answer against the 3-criterion rubric. Each criterion is binary (1 if hit, 0 if not). Total score per condition = 0–3.

Score conservatively: a brief mention without explanation does not count as "captures a fact." A citation needs to be a real URL, not a plausible-sounding domain name.

If you cannot determine a criterion clearly from the output, mark it as 0 and note "ambiguous" in the per-condition speaker notes.

### Phase 5 — Render the grid

Output format (markdown table):

```
# eval-compare results — {target_skill_name} on {query[:60]}

|  | Time | Quality | Notes |
|---|---|---|---|
| **A.** Opus, no skill (with year) | {Ts} | {qa}/3 | {flags or one-line summary} |
| **B.** Opus, no skill (no year) | {Ts} | {qb}/3 | {flags} |
| **C.** Opus + /{skill} | {Ts} | {qc}/3 | {flags} |
| **D.** Sonnet + /{skill} | {Ts} | {qd}/3 | {flags} |

## Rubric used
1. {criterion 1}
2. {criterion 2}
3. {criterion 3}

## Headline finding
{1–2 sentences: did the skill help? does small+skill match big+skill? any contamination?}
```

Render the grid AS markdown so it displays cleanly in the Claude Code terminal.

### Phase 6 — Save the run

Write the full grid + each condition's raw output (truncated to 1000 chars per condition) to `assets/last-run.md` in the skill directory. This creates a paper trail and lets the user reference the comparison later.

Also write a one-line entry to `assets/run-history.md` (append): `{timestamp} | {target_skill} | {query[:60]} | A:{qa}/3 B:{qb}/3 C:{qc}/3 D:{qd}/3`.

## Gotchas

1. **Subagent compliance for "no skill" conditions is prompt-based, not enforced.** A subagent may invoke the target skill anyway because its description matches strongly. Mitigation: Phase 3 isolation check + visible `⚠ contaminated` flag on the grid. On stage, contamination is a feature, not a bug — it shows the audience why measurement matters.

2. **Token counts per condition are not directly available** through the Task tool. v1 reports time and quality only. For precise tokens, use the Bash variant: `claude --print --model opus -p "..."` × 4 with `--output-format json` to capture `usage.input_tokens` and `usage.output_tokens` per call. Future v2.

3. **Quality scoring is LLM-as-judge** — the parent agent reads outputs and scores them. This means the scoring is correlated across conditions (same scorer for all). For research-grade evaluation, score each output blind via separate Claude API calls. v1 accepts the bias.

4. **Rubric must be 3 binary criteria, not 5.** More than 3 criteria slows scoring and dilutes the signal. If you find yourself wanting 5, split into two separate eval-compare runs with different rubrics.

5. **Default works for `tech-research` only.** Other skills need their own query + rubric. See `references/rubric-format.md` for the format.

6. **Sonnet model parameter requires Sonnet to be available** in the current Claude Code installation. If the runtime errors with "model not found," fall back to running Condition D with the default model and labeling it "second run, same model" in the grid.

7. **Don't run more than 4 conditions.** Adding "self-generated skill" or "haiku" or "with-MCP" expands the matrix but kills the signal — the audience can read 4 rows in 5 seconds, 8 rows is a wall.

8. **Don't run this on a skill that itself spawns subagents.** You'd be running subagents-in-subagents, which Claude Code may handle but is hard to reason about. Test the target skill standalone first.

## Output: the grid is the deliverable

The user wants ONE thing: the comparison grid. Don't bury it in commentary. Render the grid first, then add a 2–4 sentence interpretation, then offer to save or re-run.

## Adapting to other skills

For non-`tech-research` skills, the conditions still map cleanly:

- **A/B (no skill)**: prompt-only baselines test what the model knows / does without procedural guidance
- **C (skill on big model)**: full skill effect
- **D (skill on small model)**: cost-play check

But the **rubric** must be redesigned per-skill. A `code-review` skill needs different criteria than a `tech-research` skill. See `references/rubric-format.md`.
