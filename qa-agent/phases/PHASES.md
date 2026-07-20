# Phased rollout

**QA** + **spt-agent extract**. Orchestration remains in support-agent for all phases.

---

## Phase 0 — Review (current)

| Deliverable | Location |
|-------------|----------|
| Keep plan | `.github/qa-agent/` |
| spt-agent extract spec | [agents/spt-agent.md](../agents/spt-agent.md) |

**Exit:** Review checklist in PLAN.md signed off.

---

## Phase 1 — Scaffold `spt-agent/`

| Task | Owner |
|------|-------|
| Create `am-agents/spt-agent/` module (app, Dockerfile, Helm) | spt-agent |
| HTTP API: prepare / execute / status / cancel | spt-agent |
| Engines: memory + k6 stub (sandbox gate) | spt-agent |
| Contract tests | spt-agent |
| Draft ADR-006 extract | docs |
| Registry entry (not yet live traffic) | support-agent |

**Exit:** spt-agent pod healthy; contract tests green; **no orchestration code in module**.

---

## Phase 2 — Wire support-agent → spt-agent

| Task | Owner |
|------|-------|
| Adapter `adapters/spt_agent/` | support-agent |
| Retarget `activities/spt.py` capability calls to spt-agent HTTP | support-agent |
| Dual-run parity vs `tool-agent/tools/spt/` | both |
| Enable `SUPPORT_AGENT_SPT_PARITY` in staging against spt-agent | ops |

**Exit:** Staging SPT demand completes via spt-agent; parity report accepted.

---

## Phase 3 — Cutover + QA surface

| Task | Owner |
|------|-------|
| Deprecate / remove `tool-agent/tools/spt/` | tool-agent |
| PR / `/spt` trigger via am-pipelines | am-pipelines |
| Optional observe/verify via tool-agent | support-agent |
| `catalog/qa/` functional domains (parallel track) | catalog + support-agent |

**Exit:** Production SPT path uses spt-agent only; tool-agent SPT plugin gone.

---

## Phase 4 — Intelligence / ops

| Task | Owner |
|------|-------|
| RunStore dashboards | am-obs-platform |
| Optional LLM scope in **support-agent only** | support-agent |
| Flaky / risk-based selection | support-agent |

**Exit:** Metrics live; spt-agent still execute-only.

---

## Dependency graph

```text
Phase 0 review
    │
    ▼
Phase 1 spt-agent scaffold
    │
    ▼
Phase 2 support-agent → spt-agent
    │
    ▼
Phase 3 cutover + PR
    │
    ▼
Phase 4 intelligence (orchestrator only)
```

---

## Out of scope forever (for spt-agent module)

- Temporal workflows inside spt-agent  
- Selector expand / fan-out  
- Parent RunStore ownership  
- Calling other specialists  
- Becoming QA orchestrator  
