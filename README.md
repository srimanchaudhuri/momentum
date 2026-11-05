<div align="center">

# Momentum

### *Pick a job. Crack it.*

**The backend for Momentum: it turns a real job posting into a day-by-day plan to be ready to apply, with free resources and an AI coach that can act on the plan.**

</div>

---

## What is Momentum?

Momentum turns a job posting into a plan you can follow. Pick a real listing, or add one you found elsewhere, tap **Crack It**, and you get:

- The job's **requirements**, compared skill by skill with **your background**
- Whether you can be ready **before applications close**, and what to do if you can't
- A **week-by-week roadmap** to the deadline, with each week planned **day by day** as it arrives
- **Free resources** for every day's task, pointing to the exact part that teaches it: *"watch 12:40–21:10"* or *"read the section on window functions"*
- **Assignments**, set and graded by AI, that feed a per-skill **job readiness** score
- A **coach** that answers questions about your plan, and can reschedule or replan it once you approve

This repository is a ground-up rebuild of the Momentum backend. It keeps the existing API contract, so the Momentum mobile app works against it unchanged.

---

## What's new in this version

### Resource engine
Resources are picked by what they actually teach, not by title and view count.

- **Indexes the content itself:** YouTube transcripts as timestamped chunks and web pages as heading-based chunks, shared across all users.
- **Searches from the task, not the skill:** writes several queries for each day's task (multi-query and HyDE).
- **Retrieves in two stages:** BM25 and vector search fused with reciprocal rank fusion, then a cross-encoder reranks the results against the exact task.
- **Picks segments, not whole videos:** each pick links to a timestamp or section and gives a reason grounded in that segment.
- **Respects level and order:** matches difficulty to the user, orders picks with a skill prerequisite graph, and removes near-duplicates.
- **Learns from users:** a resource the user replaces or removes becomes a relevance label for evaluation and tuning.

### Agentic coach and MCP server
- **Momentum as an MCP server:** tools such as `get_today`, `get_progress`, `get_week`, `search_resources`, `explain_from_resource`, `reschedule_task`, `replan_week` and `add_note`. Any MCP client can use them, including Claude Desktop.
- **A coach that plans and checks itself:** a router sends quick questions straight to an answer and bigger ones to a planner, a tool executor and a reflection step that checks every claim against plan data.
- **Long-term memory:** the coach remembers learning style, weak skills from assignments and past struggles.
- **Asks before acting:** every plan change is shown as a diff the user approves.
- **Reaches out first:** a scheduled check-in starts when the user falls behind.

### Built to scale
- Ingestion and plan generation run on queue-based workers, never inside an HTTP request. Jobs use idempotency keys, retries and a dead-letter queue.
- Redis token-bucket rate limiting with per-tier quotas on LLM-heavy endpoints.
- A semantic cache for repeated coach questions and resource searches.
- A transactional outbox for notifications, and a scheduler that is safe to run on several servers.
- Tracing, metrics and dashboards with OpenTelemetry and Langfuse.

---

## Architecture

```
  Mobile app / MCP clients (Claude Desktop, ...)
                    │
┌───────────────────▼─────────────────────────────┐
│  API gateway: auth, rate limits, streaming      │
├─────────────────────────────────────────────────┤
│  Coach agent: router, planner, tool executor,   │
│  reflection, long-term memory                   │
├─────────────────────────────────────────────────┤
│  MCP server: plan, progress and resource tools, │
│  per-tool permission scopes                     │
├─────────────────────────────────────────────────┤
│  Resource engine: query generation, BM25 +      │
│  vector retrieval, RRF, cross-encoder rerank    │
├─────────────────────────────────────────────────┤
│  Workers: ingestion, chunking, plan generation  │
│  (Redis queue)                                  │
├─────────────────────────────────────────────────┤
│  MongoDB: users, goals, plans, progress events, │
│  shared resource corpus · Redis                 │
└─────────────────────────────────────────────────┘
        Evals and observability across all layers
```

One coach message travels top-down: the gateway passes it to the coach agent, the agent calls tools on the MCP server, and `search_resources` runs the resource engine over the indexed chunks. Workers fill the store from below.

---

## Tech stack

| Layer | Technology |
|---|---|
| API | Python, FastAPI |
| Database | MongoDB |
| Queue, cache, rate limits | Redis |
| Retrieval | BM25 inverted index, vector search, cross-encoder reranking |
| Agent | LangGraph, Model Context Protocol (MCP) |
| Observability | OpenTelemetry, Grafana, Langfuse |
| Local development | Docker Compose |
| CI | GitHub Actions |

---

## API

The API contract is kept as an OpenAPI spec in [`docs/`](docs/). It covers auth (email codes and OAuth), background and job preferences, the jobs feed, Crack It, goals and plans, resources, assignments, readiness, coach chat (Server-Sent Events), notes, objectives, stats, notifications and the dashboard.

---

## Getting started

> Setup instructions will be added once the local stack (Docker Compose with MongoDB and Redis) is in place.

---

## Status

Under active development.

- [ ] API contract, design doc and SLOs
- [ ] Local stack and CI
- [ ] Schema and shared resource corpus
- [ ] Ingestion pipeline and workers
- [ ] Resource engine with evaluations
- [ ] Coach agent and MCP server
- [ ] Multi-instance scaling and observability
- [ ] Production deployment and migration of the mobile app

---

<div align="center">

**Built by [Sriman Chaudhuri](https://github.com/srimanchaudhuri)**

</div>
