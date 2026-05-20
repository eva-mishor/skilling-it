# Tech-Research Skill — Gotchas

Edge cases and known failure modes. Consult when skill behavior surprises you, when debugging a stale recommendation, or during the monthly audit of `fast-moving-domains.md`.

1. **The "since 2023" check is self-referential.** Phase 1 asks the model whether a domain has changed — using the same training data that's potentially stale. Phase 1.5 disclosure mitigates the silent-miss case, but the underlying detector is still blind to its own blind spots.

2. **`fast-moving-domains.md` is a positive list.** Domains not on it default to "slow." Newly-disrupted classically-stable fields (post-quantum crypto 2024, sqlite-vec / DuckDB embedded analytics, HTTP/3 default) only get caught when someone updates the file. Watch for repeated Phase 1.5 disclosures on the same domain — that's the signal to promote it.

3. **The NO/YES branch is "recommended," not "mandatory."** A model can route to "recommended" and still skip web search. Phase 1.5 disclosure does NOT fire on this path — it only fires on NO/NO. The escape hatch leaks; if zero searches actually run, treat the answer with the same caution as a slow-domain exit.

4. **The Astral meta-pattern is hardcoded as a 2024-2025 truth.** SKILL.md embeds "Astral is systematically replacing Python tooling" as a stable fact. If Astral pivots, gets acquired, or its roadmap diverges, the meta-pattern itself becomes a stale recommendation — the exact failure mode this skill exists to prevent. Re-validate during the monthly review of `fast-moving-domains.md`.

5. **Phase 0 self-staleness check decays under repeated "proceed anyway".** No automatic enforcement — if the user always picks (a), the file rots silently. Consider escalating to a hard refresh after 3 months.

6. **Web search has no reliable date filter.** The skill notes this for Phase 2b, but it applies to all phases. A 2-year-old blog post can dominate Phase 3 cross-referencing if the model doesn't manually inspect dates.

7. **Multi-domain queries get under-served.** A query like "best Python ORM for serverless edge deployment" spans ORMs (fast-moving) + Serverless (fast-moving) + edge runtimes (volatile). Phase 1 routes by primary domain only. When two or more domains are flagged, treat the query as ⚡ regardless of any single domain's tier.

8. **Phase 4's research summary template encourages confident framing** even when search yielded thin evidence. "Current SOTA: X" reads authoritative whether X came from 10 sources or 1 low-quality blog post. When evidence is thin, say so explicitly in the summary rather than smoothing it over.

9. **The `override and search` trigger is intent, not literal string.** Phase 1.5 instructs the user to reply with `override and search`, but users will say "search anyway," "look it up," "do the research." Treat any request to override as a valid trigger — the literal string is just a hint.

10. **Don't nest tech-research inside another skill that already does retrieval.** You'll double-search, double-cite, and have to merge two research summaries. Run tech-research as the primary research step, not a sub-step.
