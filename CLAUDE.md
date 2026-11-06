# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

An adaptive multi-agent investment research platform. A user enters a company/ticker and an investment profile; a central orchestrator dispatches specialized agents that pull real data via tools, tracks everything in shared research state, adapts the research plan when findings demand it, and produces an evidence-based report. An evaluation harness scores orchestrator decisions, not just the final output.

Portfolio project for new-grad SWE / AI-engineering applications. Every design choice should be explainable in an interview.

## Stack

- **Backend:** Python 3.12, FastAPI, Pydantic v2, httpx (async)
- **LLM:** Anthropic Python SDK — direct Messages API + tool use. No agent frameworks (no LangChain, LangGraph, CrewAI, AutoGen). The agent loop and orchestrator are hand-written.
- **Frontend:** React + Vite (port 5173)
- **Storage:** SQLite via SQLAlchemy; research sessions + state snapshots
- **Testing:** pytest (backend), eval harness in `backend/evals/`
- **Data sources:** SEC EDGAR (filings), one financial-data API, one news API (chosen in Phase 3)

## Repo layout

```text
backend/
  main.py               # thin entry point — imports from app.main
  app/
    main.py             # FastAPI app creation + CORS middleware
    api/                # FastAPI routes (thin; no business logic)
    services/           # research session management
    tools/              # data-source wrappers (EDGAR, financials, news)
    agents/             # specialized agents          <-- MATTHEW WRITES
    orchestrator/       # planning + dispatch loop    <-- MATTHEW WRITES
    state/              # shared research state model <-- MATTHEW WRITES
    llm/                # thin Anthropic client wrapper
    models/             # Pydantic request/response schemas
  evals/                # scenario fixtures + harness <-- MATTHEW WRITES
  tests/
frontend/
  src/
```

## Commands

```bash
# backend (run from backend/)
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload          # API on :8000
pytest                                  # unit tests
python -m evals.run                     # eval harness (Phase 10+)

# frontend (run from frontend/)
npm install
npm run dev                             # Vite on :5173
npm run dev:all                         # frontend + backend together
npm run build
```

Update this section whenever a command is added or changed.

## Ownership rules (IMPORTANT — follow these strictly)

This is a learning project.

**Claude Code may write directly:**

- Frontend components, styling, routing, API client code
- FastAPI route wiring, Pydantic schemas, config, env loading
- `tools/` HTTP wrappers for external APIs (request/response plumbing only)
- DB setup, migrations, boilerplate, Dockerfiles, CI config
- Test scaffolding and fixtures for non-core code

**Matthew writes; Claude Code guides only — `agents/`, `orchestrator/`, `state/`, `evals/`:**

- Do NOT write implementation code in these folders unless Matthew explicitly says "write it" or "give me the full code".
- Instead: break the problem down, name the key concepts, give 1–3 hints and a starting step, then stop.
- When reviewing his code: point out what's wrong and why with a small example; let him fix it.
- Interfaces/type signatures for these folders may be proposed in chat, not written to files.

When debugging anything in these folders: ask what he thinks is wrong and what he's tried before proposing a fix. Suggest prints/breakpoints/tracing steps first.

## Architecture

**Frontend → Backend:** Vite proxies all `/api/*` requests to `http://localhost:8000` in dev. React code calls `/api/...` with no hardcoded URLs.

**Agent system — the core idea:** The orchestrator does not follow a fixed pipeline. It reads shared research state and decides what to research next. The research path changes based on what agents discover.

**Orchestrator loop (simplified):**

```text
Create initial research plan
        ↓
Dispatch agents
        ↓
Collect findings into shared state
        ↓
Evaluate evidence
   ↙              ↘
gaps/conflicts?   sufficient?
   ↓                  ↓
dispatch more     generate report
```

**Key agentic scenarios the orchestrator must handle:**
- High debt detected → dispatch follow-up to investigate why and assess serviceability
- Conflicting findings (strong competitive advantage vs. declining market share) → dispatch verification task
- Major news (lawsuit, regulatory probe) → update research plan, investigate financial impact
- Sufficient consistent evidence → stop early, don't over-research

**Shared research state** (passed between orchestrator and agents):
- Research goal + investment profile
- Current research plan (mutable)
- Completed tasks + agent findings
- Sources (every finding carries `source_url`/`source_id` + retrieval time)
- Conflicting claims, open questions, confidence level

**Specialized agents:**
1. Financial Analysis — revenue, earnings, debt, cash flow, ratios
2. News & Recent Developments — earnings announcements, acquisitions, lawsuits, regulatory issues
3. Company Research — business model, products, competitive advantages, weaknesses
4. Competitor Analysis — relevant competitors identified per company, not a fixed list
5. Risk Analysis — adversarial; looks for reasons the investment could go wrong
6. Verification / Deep-Dive — dispatched on-demand by orchestrator for specific follow-up questions

## Design principles

- **Tools over model memory.** Agents get facts from `tools/`; never claim a number that didn't come from a tool result.
- **Every finding carries a source.** `source_url`/`source_id` + retrieval time on all state findings.
- **The orchestrator decides from state, not a fixed sequence.** The plan is data that can change mid-research.
- **Knowing when to stop is a feature.** "Sufficient evidence → generate report" is a tested path.
- **Separate fact / interpretation / uncertainty** in the final report.
- **No buy/sell recommendations.** Include a not-financial-advice notice.
- **Show progress, not chain-of-thought.** UI shows plan steps and status changes, not raw model reasoning.
- **Working version at every phase.** Never leave main broken; build in layers.

## Conventions

- Type hints everywhere; Pydantic models for anything crossing a boundary (API, state, tool results).
- Async for all I/O (httpx, LLM calls).
- Secrets in `.env` (never committed). `.env.example` lists required keys.
- Log every tool call and orchestrator decision (structured logs) — the eval harness depends on this trace.
- Commit messages: `phaseN: short description`.

## Evaluation harness (Phase 10)

Evaluates orchestrator decisions throughout the process, not just final report quality.

Controlled test scenarios:
- **Customer concentration risk** — data contains evidence of heavy dependence on one customer; orchestrator should detect and investigate
- **Conflicting evidence** — sources disagree; orchestrator should request verification, not accept both claims
- **No additional research needed** — consistent evidence with no red flags; orchestrator should stop efficiently
- **Major recent development** — significant event changes outlook; orchestrator should update plan and investigate impact

Metrics: task selection accuracy, important-finding detection, conflict handling, tool use relevance, efficiency (avoid unnecessary calls), source quality, final report accuracy.

## Python environment

The venv at `backend/.venv` stores absolute paths and breaks if moved. If the interpreter path is wrong, recreate it:

```bash
cd backend
python3 -m venv --clear .venv
.venv/bin/pip install -r requirements.txt
```

## Current phase

<!-- Update this as phases complete -->

- [x] Phase 1 — MVP path: ticker in → real data → simple report
- [x] Phase 2 — Frontend/backend foundation
- [ ] Phase 3 — Real data sources (EDGAR + financials + news)
- [ ] Phase 4 — Single tool-using research agent
- [ ] Phase 5 — Specialized agents
- [ ] Phase 6 — Shared research state
- [ ] Phase 7 — Orchestrator loop
- [ ] Phase 8 — Dynamic research decisions
- [ ] Phase 9 — Research progress UI
- [ ] Phase 10 — Evaluation harness
- [ ] Phase 11 — Reliability, polish, README, demo video

### Active: Phase 3
