# Tech-Research Skill — Worked Example

Reference example showing all phases for a highly-volatile domain query. Load when you need to see the procedure end-to-end.

## Example: "Which LLM framework for a RAG pipeline?"

**Phase 1**: LLM frameworks is in `fast-moving-domains.md` under AI/ML Tooling (highly volatile) → mandatory research + community search.

**Phase 2**: Search `"LLM RAG framework best practices 2026"`, `"langchain vs llamaindex 2026"`.

**Phase 2b**: Community search (highly volatile domain):
- `"RAG framework reddit 2026"` → find r/LocalLLaMA, r/LangChain threads
- `"langchain llamaindex site:news.ycombinator.com"`
- `"switched from langchain to reddit 2026"` → migration stories
- Discard any results older than 6 months.

**Phase 3**: Cross-reference official docs + GitHub activity + blogs + Reddit/HN sentiment.

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
