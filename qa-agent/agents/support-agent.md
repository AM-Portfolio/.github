# support-agent — QA orchestrator plan

**Canonical name:** `support-agent`  
**Path:** `am-agents/support-agent/` · package `am_support_agent`  
**Role in QA:** Orchestrate scope, routing, verification, and final verdict.

---

## Purpose in QA

support-agent is the **only orchestrator** for the QA agent. It:

1. Accepts QA demand (selector + environment)
2. Resolves targets from `catalog/qa/` and `catalog/verify/`
3. Routes each target to the correct specialist
4. Aggregates results with partial-failure rules (ADR-004)
5. Writes RunStore and publishes verdict

It does **not** run Playwright, k6, or pytest directly.

---

## Existing assets to extend

| Component | Path | QA extension |
|-----------|------|--------------|
| QA workflow | `orchestrator/workflows/qa_run.py` | Planned |
| QA activities | `orchestrator/activities/qa.py` | Planned |
| Legacy perf workflow | `orchestrator/workflows/spt_run.py` | Exists — may merge into qa_run |
| Legacy perf activities | `orchestrator/activities/spt.py` | Exists |
| Catalog reader | `intelligence/catalog.py` | `list_qa()`, resolve QA selectors |
| Router | `orchestrator/router.py` | Map QA target kind → specialist |
| Registry | `registry/agents.yaml` | No change — already lists specialists |
| RunStore | `stores/run_store.py` | `kind=qa` runs |

---

## QA workflow (planned)

```text
QaRunWorkflow
  │
  ├─ activity: resolve_qa_catalog     (read-only)
  ├─ activity: expand_selector        (TargetSet)
  ├─ activity: route_and_fanout       (parallel, bounded)
  │     ├─ HTTP → tool-agent
  │     └─ HTTP → ui-test-agent
  ├─ activity: verify_results         (catalog/verify checks)
  └─ activity: finalize_summary       (RunStore + notify)
```

Gating: `SUPPORT_AGENT_QA_PARITY` until staging proven.

---

## Planner / scope (replaces v1 "Fin Agent")

**Find phase**

- Read PR diff or demand context
- Match changed paths to catalog tags (`backend`, `frontend`, `api`, etc.)
- Build selector: `{ "ids": [...], "tags": [...] }`
- Fail if selector empty (ADR-004)

**Finalize phase**

- Merge `ChildRunResult` from all specialists
- Apply `failure_mode`: `continue` (default) or `fail_fast`
- Set `overall_status`: succeeded | partial | failed
- Emit human-readable PR summary + structured `QaRunSummary`

---

## Specialist calls

| Target kind | Route to | Capability |
|-------------|----------|------------|
| backend | tool-agent | `tools.execute` |
| network | tool-agent | `tools.execute` (spt/observe) |
| spt / load | tool-agent | `tools.execute` (spt plugin) |
| frontend | ui-test-agent | `ui.test.run` |
| verify | tool-agent | `tools.query` + observe |
| data (optional) | db-agent | `db.execute` |

Auth: `X-Agent-Caller: support-agent` (existing pattern).

---

## Budgets (from registry defaults)

```yaml
max_fanout: 8
max_latency_ms: 120000
max_cost_units: 100
```

QA runs must respect the same budgets as other support-agent workflows.

---

## Observability (exists — no new design)

- Spans: `support_agent.task` with `capability`, `domain`, `status`
- Metrics: `support_agent_resolution_outcomes_total`
- No payloads or secrets in traces

---

## Tests to add (Phase 2+)

| Test | Type |
|------|------|
| QA catalog resolve | unit |
| Selector empty → fatal | unit |
| Route backend → tool-agent mock | contract |
| Partial failure summary | unit |
| Full QA demand e2e | integration (staging) |

---

## References

- [support-agent/README.md](https://github.com/AM-Portfolio/am-agents/blob/main/support-agent/README.md)
- [PLAN.md](../PLAN.md)
- [ALIGNMENT.md](../ALIGNMENT.md)
