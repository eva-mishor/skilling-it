# Fast-Moving Technical Domains

<!-- Last reviewed: 2026-04-12 -->
<!-- This file drives the skill's volatility check. It's a list of things that change fast, so it goes stale fast. Review monthly. -->

Domains where training data becomes stale quickly. Always verify via web search.

## Volatility Tiers

- **Highly volatile** (marked with ⚡): Community recency search mandatory (Phase 2b). These shift month-to-month.
- **Fast-moving**: Year-scoped web search mandatory (Phase 2). These shift yearly.

## Key Disruptors to Watch

**Astral (astral.sh)** - Systematically replacing Python tooling with Rust:
| Old Tool | Astral Replacement | Status |
|----------|-------------------|--------|
| flake8, isort, pylint | ruff | Stable, dominant |
| pip, poetry, pipenv | uv | Stable, rapidly adopted |
| mypy, pyright | ty | Beta (Dec 2025), watch for stable |

Always search "astral {category}" when researching Python tooling.

## Tooling & Build Systems

| Category | Examples | Why It Changes Fast |
|----------|----------|---------------------|
| Python Linting | ruff vs flake8, pylint | ruff replaced flake8+isort in 2023-2024 |
| Python Packaging | uv vs pip, poetry, pdm | uv disrupting in 2024-2025 |
| Python Type Checking | ty vs mypy vs pyright | ty (Astral) entering market 2025 |
| ⚡ JS Build Tools | vite, turbopack, rspack | Constant churn, webpack declining |
| Formatters | biome vs prettier+eslint | Rust-based tools replacing JS |

## Frameworks & Libraries

| Category | Examples | Why It Changes Fast |
|----------|----------|---------------------|
| ⚡ React Ecosystem | Next.js, Remix, server components | Major paradigm shifts yearly |
| Python Web | FastAPI, Litestar, Django | FastAPI patterns evolving |
| ORMs | SQLModel, Prisma, Drizzle | New entrants, API changes |
| Testing | Playwright vs Cypress, vitest vs jest | Market share shifts |

## Infrastructure & Cloud

| Category | Examples | Why It Changes Fast |
|----------|----------|---------------------|
| Containers | Docker alternatives, Podman | Licensing changes, new tools |
| K8s Tooling | Helm, Kustomize, Timoni | Ecosystem fragmentation |
| IaC | Terraform vs Pulumi, OpenTofu | Licensing drama, new tools |
| Serverless | New runtimes, edge functions | Constant platform evolution |

## ⚡ AI/ML Tooling (Highly Volatile)

| Category | Examples | Why It Changes Fast |
|----------|----------|---------------------|
| LLM Frameworks | LangChain, LlamaIndex, DSPy | Extremely rapid evolution |
| Vector DBs | Pinecone, Weaviate, Chroma | New entrants monthly |
| Model Serving | vLLM, TGI, Ollama | Performance improvements |
| Fine-tuning | LoRA variants, PEFT methods | Research moves to production |

## Search Queries by Domain

Use `{current_year}` — never hardcode the year.

### Python Tooling
```
"python linting best practices {current_year}"
"ruff vs flake8 {current_year}"
"python package manager comparison {current_year}"
"uv pip poetry comparison"
"python type checker comparison {current_year}"
"mypy vs pyright vs ty {current_year}"
"astral ty type checker"
```

### JavaScript/TypeScript
```
"react meta framework comparison {current_year}"
"vite vs webpack {current_year}"
"biome vs eslint prettier {current_year}"
```

### Infrastructure
```
"terraform alternatives {current_year}"
"kubernetes deployment tools comparison"
"docker alternatives {current_year}"
```

### Databases
```
"python orm comparison {current_year}"
"vector database comparison {current_year}"
"sqlite alternatives embedded database"
```

### ⚡ Community Recency Queries (for highly volatile domains)

Web search can't enforce date ranges — add year/quarter to queries and manually discard results older than 6 months.

```
"{topic} reddit {current_year}"
"{topic} site:reddit.com"
"{topic} site:news.ycombinator.com"
"{tool} production experience {current_year}"
"{tool A} vs {tool B} reddit {current_year}"
"switched from {tool} to reddit {current_year}"
"migrating from {tool} {current_year}"
"{topic} reddit Q{current_quarter} {current_year}"
```

**Key subreddits by domain:**
- AI/ML: r/LocalLLaMA, r/MachineLearning, r/LangChain
- Python: r/Python, r/learnpython
- JS/React: r/reactjs, r/javascript, r/nextjs
- Infrastructure: r/devops, r/kubernetes
