# support-agent — orchestrator plan

**Canonical name:** `support-agent`  
**Path:** `am-agents/support-agent/`  
**Role:** **All orchestration** for QA. Does not run QA runners.

---

## Purpose

1. Accept QA demand (selector + environment)
2. Resolve targets from `catalog/qa/` (+ verify)
3. Enforce policy (sandbox, max targets, empty selector fatal)
4. Fan-out to specialists over HTTP — primarily **qa-agent**
5. Verify, write RunStore, publish verdict

---

## Assets to add / extend

| Component | Change |
|-----------|--------|
| `workflows/qa_run.py` | New — orchestrates QA demand |
| `activities/qa.py` | New — HTTP to qa-agent |
| `adapters/qa_agent/` | New — client |
| `registry/agents.yaml` | Add `qa-agent` |
| RunStore | `kind=qa` stays here |

SPT (`spt_run.py`) remains as today — not part of qa-agent extract.

---

## QA workflow (orchestration stays here)

```text
QaRunWorkflow
  │
  ├─ resolve_qa_catalog           (local)
  ├─ expand_selector              (local)
  ├─ policy / sandbox gate        (local)
  ├─ for each target (bounded):
  │     HTTP → qa-agent.execute / status
  │     (optional) ui-test-agent / tool-agent
  ├─ optional: tool-agent observe
  └─ finalize RunStore + notify
```

**Nothing above moves into qa-agent.**

---

## Specialist routing

| Target kind | Route to |
|-------------|----------|
| backend / system / network | **qa-agent** |
| frontend | ui-test-agent and/or qa-agent (open) |
| verify / observe | tool-agent |
| perf / spt | existing SPT path (tool-agent spt plugin) |
| data | db-agent |

Auth: `X-Agent-Caller: support-agent`.

---

## LLM

| In support-agent | When |
|------------------|------|
| Catalog / route / verdict | Never |
| Optional diff → tags | Phase 4 |
| Optional PR narrative | Phase 4 |

---

## References

- [qa-agent.md](./qa-agent.md) — execute-only module  
- [../PLAN.md](../PLAN.md)  
