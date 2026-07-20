# support-agent — orchestrator plan

**Canonical name:** `support-agent`  
**Path:** `am-agents/support-agent/`  
**Role:** **All orchestration** for QA and SPT. Does not run k6.

---

## Purpose

1. Accept QA / SPT demand (selector + environment)
2. Resolve targets from `catalog/spt/` / `catalog/qa/` / `catalog/verify/`
3. Enforce policy (sandbox, max targets, empty selector fatal)
4. Fan-out to specialists over HTTP
5. Verify, Write RunStore, publish verdict

For SPT load targets, the executor is **`spt-agent`** (extracted module) — not tool-agent long-term.

---

## Existing assets

| Component | Path | Change for extract |
|-----------|------|-------------------|
| SPT workflow | `orchestrator/workflows/spt_run.py` | Keep — call spt-agent |
| SPT activities | `orchestrator/activities/spt.py` | Retarget HTTP to spt-agent |
| Catalog reader | `intelligence/catalog.py` | Unchanged ownership |
| Registry | `registry/agents.yaml` | Add `spt-agent` |
| Adapter | `adapters/spt_agent/` | **New** |
| RunStore | `stores/` | Stays here |

---

## SPT workflow (orchestration stays here)

```text
SptRunWorkflow
  │
  ├─ resolve_spt_catalog          (read-only, local)
  ├─ expand_selector              (local)
  ├─ policy / sandbox gate        (local)
  ├─ for each target (bounded):
  │     HTTP → spt-agent.prepare
  │     HTTP → spt-agent.execute
  │     HTTP → spt-agent.status
  ├─ optional: tool-agent observe
  └─ finalize RunStore + notify
```

**Nothing above moves into spt-agent.**

---

## Specialist routing

| Target kind | Route to |
|-------------|----------|
| perf / spt / load | **spt-agent** |
| backend | tool-agent |
| network | tool-agent |
| verify / observe | tool-agent |
| frontend | ui-test-agent |
| data (optional) | db-agent |

Auth to specialists: `X-Agent-Caller: support-agent`.

---

## Budgets (orchestrator)

```yaml
max_fanout: 8
max_latency_ms: 120000
max_cost_units: 100
```

Fan-out limits apply in support-agent; spt-agent may add local pod concurrency caps.

---

## LLM

| In support-agent | When |
|------------------|------|
| Catalog / route / verdict | Never |
| Optional diff → tags | Phase 4 |
| Optional PR narrative | Phase 4 |

Never put secrets in prompts (ADR-002).

---

## References

- [spt-agent.md](./spt-agent.md) — execute-only module  
- [../PLAN.md](../PLAN.md)  
