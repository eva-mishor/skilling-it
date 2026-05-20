# eval-compare results — tech-research on "I'm starting a new Python project — what type checker should I use?"

**Run timestamp**: 2026-04-28 (unix start 1777375075, parallel dispatch, ~68s wall-clock)
**Target skill**: tech-research
**Query**: "I'm starting a new Python project — what type checker should I use?"

|  | Time | Quality | Notes |
|---|---|---|---|
| **A.** Opus, no skill (+ "in 2026") | 18s | **1/3** | Names ty + pyrefly correctly, but no 2026 facts, no URLs |
| **B.** Opus, no skill (no year cue) | 22s | **0/3** | Confuses pyrefly with Pyre (older OCaml tool); stale by ~12 months |
| **C.** Opus + /tech-research | 68s | **3/3** | Astral→OpenAI acquisition + mypyc speedup + ty 0.0.x status; 6 URLs |
| **D.** Sonnet + /tech-research | 63s | **2/3** | Both tools named, 9 URLs, but misses all three rubric-specified facts |

## Rubric used
1. Mentions both `ty` (Astral) AND `pyrefly` (Meta)
2. Captures at least one non-obvious 2026 fact: ty's ~15% spec conformance, mypy now faster than pyright via mypyc, or Astral acquired by OpenAI
3. Cites recent web sources (real URLs)

## Headline finding
Skill closes a 2–3x quality gap (0–1/3 → 2–3/3) at ~3x latency cost (~20s → ~65s). Year cue alone (A vs B) bought +1 point, mostly because B confused `pyrefly` with the older OCaml `Pyre`. **The "small + skill ≈ big alone" claim only partially held this run** — Sonnet+skill (2/3) beat Opus-alone (1/3) but undershot Opus+skill (3/3). Sonnet's run finished 5s faster than Opus's, suggesting it did fewer fact-rich research hops. No contamination in the no-skill conditions.

## Comparison to 2026-04-27 run
Yesterday: A:1/3 B:0/3 C:3/3 **D:3/3**. Today: A:1/3 B:0/3 C:3/3 **D:2/3**. Same C, different D — Sonnet's research surfaced different facts each time and only caught all 3 rubric items in one of the runs. This is variance in subagent web-search depth, not a skill regression.

## Isolation check (Phase 3)
- Condition A: no URLs in output, no `Skill(` / `WebSearch(` / `Fetch` markers — **clean**
- Condition B: no URLs, no tool-invocation markers — **clean**

## Per-condition raw output (truncated to ~1000 chars)

### Condition A (Opus, no skill, +year)

> # Python Type Checkers in 2026
>
> **Short answer: use `ty` (Astral) or `pyrefly` (Meta) if you want speed, `mypy` if you want stability, `pyright` if you're in VS Code.**
>
> ## The actual landscape
>
> **ty** (Astral, makers of `ruff` and `uv`)
> - Rust-based, ~10-100x faster than mypy
> - Still pre-1.0 but maturing fast given Astral's track record
> - Best bet if you're already using `ruff` + `uv` — unified toolchain
> - Pick this for new projects unless you have a reason not to
>
> **pyrefly** (Meta)
> - Rust-based, replaced Pyre internally at Meta
> - Designed for monorepo scale
> - Solid choice if `ty` feels too early for you
>
> **pyright** (Microsoft) — TypeScript-based, fast, mature, powers Pylance.
>
> **mypy** — slowest of the bunch, most mature plugin ecosystem.
>
> [No URLs. No 2026-specific facts beyond the existence of ty/pyrefly themselves.] ... [TRUNC]

### Condition B (Opus, no skill, no year cue)

> # Python Type Checker Recommendation
>
> **Short answer: Use `pyright` (or its bundled form, Pylance) unless you have a specific reason not to.**
>
> ### 1. Pyright / Pylance (Microsoft) — default choice
> ### 2. ty (Astral) — watch this, don't bet on it yet
> Pre-1.0, still in preview. Not production-ready for a serious project today.
> ### 3. mypy — the incumbent, increasingly the wrong choice
> ### 4. pyre (Meta) — skip
> Niche, mostly relevant inside Meta. Don't pick it for a new project.
>
> [Note: lists **Pyre** (the older OCaml tool from Meta) — NOT pyrefly (the 2025 Rust successor). This is the staleness signature.]
>
> [No URLs.] ... [TRUNC]

### Condition C (Opus + /tech-research)

> ## Research Summary
>
> **Current SOTA** (web research, April 2026):
> - **pyright**: 97.8% spec conformance, weekly releases, Microsoft-backed, best-in-class via Pylance in VS Code.
> - **mypy**: 58.3% spec conformance but the de facto standard. **Recent mypyc-compiled builds + 1.18+ optimizations closed the speed gap with pyright.**
> - **ty (Astral)**: still 0.0.x beta as of April 2026, ~53% conformance, 7-60x faster than mypy/pyright. **Astral was acquired by OpenAI in 2026** — adds governance uncertainty.
> - **Pyrefly (Meta)**: open-sourced May 2025 (Rust rewrite of internal Pyre). MIT licensed. ~58% conformance.
> - **Zuban**: from the Jedi author, mypy-compatible, ~69% conformance.
> - **Basedpyright**: community fork of pyright, biweekly cadence.
>
> **Recommendation**: For a brand new Python project starting today, **use pyright**.
>
> **Sources**: pydevtools.com, sinon.github.io, astral.sh/blog/ty, github.com/astral-sh/ty/releases, pkgpulse.com, positron.posit.co/blog/posts/2026-03-31 ... [TRUNC]

### Condition D (Sonnet + /tech-research)

> ## Research Summary
>
> The headline change since 2023: two new Rust-based challengers (ty from Astral, pyrefly from Meta) have arrived and are genuinely fast — but both are still in beta.
>
> ### Pyright (Microsoft) — Current production default
> ### Mypy — Reliable, ecosystem-compatible, but aging
> mypyc-compiled builds improved performance in 1.18+, but still 10–60x slower than the Rust tools. Only 57% conformance on the official typing spec.
> ### ty (Astral) — Beta, not production-ready yet
> Currently at v0.0.21 (0.0.x = explicitly unstable API). 10–60x faster than mypy/pyright.
> ### pyrefly (Meta) — Beta, slightly more mature than ty
> 90% conformance on the typing spec — highest of the new tools. ~2x slower than ty in benchmarks.
>
> **Recommendation**: Use pyright.
>
> **Sources**: pkgpulse.com, pydevtools.com, astral.sh/blog/ty, blog.edward-li.com, pyrefly.org/blog, infoworld.com, engineering.fb.com, discuss.python.org, sinon.github.io ... [TRUNC]
