# Origyn

**Thesis-driven company prospecting.** Describe what you're looking for in plain English; Origyn discovers companies that fit, scores them, and writes an evidence-backed memo for each — with every claim traceable to a source.

![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178C6?logo=typescript&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-15-000000?logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)
![Turborepo](https://img.shields.io/badge/Turborepo-monorepo-EF4444?logo=turborepo&logoColor=white)
![Trigger.dev](https://img.shields.io/badge/Trigger.dev-v3-A855F7)
![Supabase](https://img.shields.io/badge/Supabase-Postgres%20%2B%20pgvector-3FCF8E?logo=supabase&logoColor=white)
![OpenRouter](https://img.shields.io/badge/OpenRouter-LLM%20gateway-6E56CF)
![Zod](https://img.shields.io/badge/Zod-structured%20output-3E67B1)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-4-06B6D4?logo=tailwindcss&logoColor=white)

---

## What it does

Sourcing and diligence usually start with a vague mandate — *"early-stage infrastructure companies with a technical founding team and signs of enterprise traction"* — and end in days of manual searching, tab-hoarding, and copy-pasting into a spreadsheet. The hard part isn't finding *names*; it's finding the *right* names and being able to defend why each one made the list.

Origyn turns that mandate into a repeatable pipeline:

1. **Compile the thesis.** A natural-language thesis is parsed into structured, machine-checkable criteria — no invented constraints, empty where unspecified.
2. **Discover candidates.** The thesis drives a planned set of search queries across the open web and regulatory filings, plus "find similar" expansion from strong matches.
3. **Extract grounded facts.** Source documents are chunked and mined for facts, each one bound to the exact evidence span it came from.
4. **Score and rank.** Companies receive a transparent, component-based opportunity score aligned to the thesis.
5. **Write the memo.** A per-company memo is generated, then passed through an **independent reviewer** that flags any claim not backed by the evidence ledger.

The result is a ranked shortlist where every score and sentence can be traced back to a primary source.

## How it works

Origyn is a Turborepo monorepo. A Next.js app owns the analyst-facing UI; durable background workers own the heavy, long-running discovery pipeline; and a set of focused TypeScript packages hold the domain logic so it can be tested in isolation.

```
            ┌──────────────────────────────┐
            │  Next.js app  (analyst UI)    │
            │  theses · shortlist · memos   │
            │  watchlist · usage/cost       │
            └───────────────┬──────────────┘
                            │  trigger run
                            ▼
            ┌──────────────────────────────┐
            │  Trigger.dev workers (durable)│
            │  orchestrate the pipeline     │
            └───────────────┬──────────────┘
                            ▼
   thesis ─▶ query plan ─▶ discover ─▶ extract ─▶ score ─▶ memo ─▶ review
      │           │           │           │         │        │       │
      └───────────┴───────────┴─────┬─────┴─────────┴────────┴───────┘
                                     ▼
        ┌───────────────────────────────────────────────┐
        │  Model gateway  →  OpenRouter (per-stage policy)│
        │  Sources  →  web search · "find similar" · EDGAR│
        │  Supabase Postgres + pgvector (evidence ledger) │
        └───────────────────────────────────────────────┘
```

**Data flow, end to end:** the analyst writes a thesis in the web app and kicks off a run. A durable worker compiles the thesis into criteria, plans queries, and pulls candidate companies from search and regulatory sources. Each candidate's documents are chunked, embedded, and mined for grounded facts; facts feed a component score and a generated memo; an independent review pass checks the memo against the evidence ledger before anything surfaces in the shortlist. Throughout, model usage (tokens, cost, latency) is written to an audit table so spend is visible per workspace.

## Highlights

- **Evidence-first data spine.** The schema enforces the discipline: a source document is not a fact, a fact must link to an evidence span, and an analyst edit *supersedes* a fact rather than overwriting it. Nothing is asserted without provenance.
- **Independent review pass.** Memos are checked by a separate model from the one that authored them, with unsupported claims surfaced rather than silently shipped — an author/reviewer separation borrowed from how diligence actually works.
- **Single source of truth for model selection.** A per-stage model policy maps each pipeline step (triage, extract, score, thesis, memo, review, embeddings) to a model and fallback chain. Upgrading from a cost-conscious lineup to frontier models is a config change, not a code change.
- **Structured outputs, validated.** Every LLM call that produces data returns against a Zod schema, so malformed model output fails fast instead of poisoning the pipeline downstream.
- **Source compliance built in.** A compliance service enforces a host allowlist, polite rate limiting, and a descriptive User-Agent for regulatory data access, with every fetch logged.
- **Quality gates as code.** An evals package defines explicit acceptance thresholds — schema validity, citation correctness, unsupported-claim rate, duplicate rate, and precision@K — and computes them from fixtures so regressions are caught in CI rather than in production.
- **Multi-tenant by construction.** Every table carries a workspace identifier and is row-level-security isolated; privileged writes go through workers using a service role while members read.
- **Cost observability.** Token counts, estimated cost, and latency for every model call are persisted per stage, powering an in-app usage dashboard.
- **Resilient pluggable discovery.** Web-search discovery uses a primary search backbone with automatic fallback, plus "find similar" expansion and an SEC filings connector — search providers are swappable behind a common interface.

## Tech stack

**Monorepo & tooling** — Turborepo, pnpm workspaces, TypeScript, Prettier, Vitest, GitHub Actions CI.

**Web app** — Next.js 15 (App Router) · React 19 · Tailwind CSS 4 · Radix UI / shadcn-style components · TanStack Table · Recharts · server actions and middleware-based auth.

**Background workers** — Trigger.dev v3 for durable, observable, long-running pipeline tasks.

**Domain packages** — a model gateway over OpenRouter (OpenAI-compatible SDK) with per-stage policy, fallbacks, and structured output; a sources package (web search, "find similar", SEC EDGAR, compliance); a pipeline package (thesis compiler, query planner, fact extraction, scoring, memo generation, review); a database package; and an evals package with quality gates.

**Data** — Supabase (Postgres + pgvector) with SQL migrations, row-level security, and an embedding-backed evidence store.

**LLM** — OpenRouter as a single gateway across multiple model providers, selected per pipeline stage; embeddings for semantic retrieval.

## Status

Active personal project, developed as a Turborepo monorepo with unit tests and CI across the domain packages. Architecture and pipeline are in place end to end; the codebase is private. This repository is a public showcase — the README only. Happy to talk through the design in more detail on request.
