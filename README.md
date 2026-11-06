# Thesis

An adaptive multi-agent investment research platform. Enter a company and an investment profile; a central orchestrator dispatches specialized AI agents that pull real data from financial APIs and SEC filings, coordinates their findings through shared research state, and adapts the research plan as new information emerges — producing an evidence-based investment research report.

> This tool is designed to support investment research and decision-making. It is not financial advice and does not guarantee investment outcomes.

## What it does

A user provides a company or ticker alongside an investment profile (horizon, risk tolerance, goals, existing holdings). The system then:

1. Creates an initial research plan
2. Dispatches specialized agents to gather financial data, company information, news, and competitor analysis
3. Monitors findings for gaps, conflicts, and significant developments
4. Adapts the research plan dynamically based on what agents discover
5. Produces a structured report with sourced evidence

The key distinction from asking a general-purpose AI about a stock: the system performs a structured, evidence-based research process using real data sources — and the orchestrator changes course based on what it finds.

## Architecture

```text
User Request
     ↓
Research Orchestrator
     ↓
┌────────────────────────────────────┐
│  Financial  │  News  │  Company   │
│   Agent     │ Agent  │  Agent     │
│             │        │            │
│  Competitor │  Risk  │  Deep-Dive │
│   Agent     │ Agent  │  Agent     │
└────────────────────────────────────┘
     ↓
Shared Research State
     ↓
Orchestrator Decision
  ↙          ↘
More       Sufficient
Research   Evidence
  ↓             ↓
Dispatch    Final Report
```

The orchestrator does not follow a fixed pipeline. It reads the evolving research state and decides what to investigate next — including when to stop.

## Specialized agents

| Agent | Responsibility |
|---|---|
| Financial Analysis | Revenue, earnings, debt, cash flow, ratios |
| News & Developments | Recent events, announcements, lawsuits, regulatory issues |
| Company Research | Business model, products, competitive advantages |
| Competitor Analysis | Relevant competitors identified per company |
| Risk Analysis | Adversarial — reasons the investment could go wrong |
| Verification / Deep-Dive | Dispatched on-demand to investigate specific findings |

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React + Vite |
| Backend | Python, FastAPI, Pydantic v2 |
| LLM | Anthropic API — direct tool use, no agent frameworks |
| Storage | SQLite via SQLAlchemy |
| Data | SEC EDGAR, financial data API, news API |
| Testing | pytest + custom evaluation harness |

## Evaluation

The project includes an evaluation harness that measures orchestrator decision quality throughout the research process — not just the final report. Test scenarios include detecting customer concentration risk, handling conflicting findings, recognizing when sufficient evidence exists, and adapting to major news events.
