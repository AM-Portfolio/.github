# Phased rollout

**qa-agent extract.** Orchestration remains in support-agent for all phases.

---

## Phase 0 — Review (current)

| Deliverable | Location |
|-------------|----------|
| Keep plan | `.github/qa-agent/` |
| qa-agent extract spec | [agents/qa-agent.md](../agents/qa-agent.md) |

**Exit:** Review checklist in PLAN.md signed off.

---

## Phase 1 — Scaffold `am-agents/qa-agent/`

| Task | Owner |
|------|-------|
| Create module (app, Dockerfile, Helm) | qa-agent |
| HTTP API: execute / status / cancel | qa-agent |
| Runners: backend + network (pilot) | qa-agent |
| Contract tests | qa-agent |
| Draft ADR-006 qa-agent extract | docs |
| Registry entry (not yet live) | support-agent |

**Exit:** qa-agent pod healthy; contract tests green; **no orchestration code in module**.

---

## Phase 2 — Wire support-agent → qa-agent

| Task | Owner |
|------|-------|
| Adapter `adapters/qa_agent/` | support-agent |
| `QaRunWorkflow` + activities | support-agent |
| `catalog/qa/` sample backend/network entries | catalog |
| Staging pilot demand | ops |

**Exit:** Staging QA demand completes via qa-agent for pilot targets.

---

## Phase 3 — PR surface + remaining domains

| Task | Owner |
|------|-------|
| PR / `/qa` via am-pipelines | am-pipelines |
| System runner + catalog entries | qa-agent + catalog |
| Frontend routing decision executed | support-agent router |
| Optional observe/verify via tool-agent | support-agent |

**Exit:** PR-triggered QA comment + labels on pilot repo.

---

## Phase 4 — Intelligence / ops

| Task | Owner |
|------|-------|
| RunStore dashboards | am-obs-platform |
| Optional LLM scope in **support-agent only** | support-agent |
| Flaky / risk-based selection | support-agent |

**Exit:** Metrics live; qa-agent still execute-only.

---

## Dependency graph

```text
Phase 0 review
    │
    ▼
Phase 1 qa-agent scaffold
    │
    ▼
Phase 2 support-agent → qa-agent
    │
    ▼
Phase 3 PR + domains
    │
    ▼
Phase 4 intelligence (orchestrator only)
```

---

## Out of scope forever (for qa-agent module)

- Temporal workflows inside qa-agent  
- Selector expand / fan-out  
- Parent RunStore ownership  
- Calling other specialists as orchestrator  
- Becoming the platform orchestrator  
