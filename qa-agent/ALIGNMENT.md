# Alignment with `am-agents`

How this plan maps to the platform — **qa-agent extract**.

---

## Decision (locked for this draft)

| Decision | Detail |
|----------|--------|
| Extract QA | New module `am-agents/qa-agent/` |
| Orchestration | **Only** in `support-agent/` |
| Catalog | `catalog/qa/` data — resolved by support-agent; **not** inside qa-agent |
| SPT | **Not** extracted in this plan — keep existing SPT path |

---

## Agent mapping

| Role | Component | Notes |
|------|-----------|-------|
| Orchestrator | **support-agent** | Workflows, selector, fan-out, RunStore, verdict |
| QA execute | **qa-agent** ★ | execute / status / cancel |
| Frontend E2E | **ui-test-agent** (and/or qa-agent) | Open decision |
| Observe / tools / SPT | **tool-agent** | Existing |
| Data checks | **db-agent** | Optional |
| Finance | **fin-agent** | Out of scope |

---

## What already exists vs net new

| Item | Status |
|------|--------|
| support-agent orchestration | Exists — extend with QaRun |
| ui-test-agent / tool-agent | Exists |
| `catalog/spt/` + SptRunWorkflow | Exists — leave as-is |
| **`am-agents/qa-agent/`** | **New** |
| `catalog/qa/` | New (data) |
| support-agent adapter `qa_agent/` | New |
| Registry entry `qa-agent` | New |

---

## Specialist routing (target)

```yaml
agents:
  - agent_id: qa-agent        # QA execute ★ NEW
  - agent_id: ui-test-agent   # frontend E2E
  - agent_id: tool-agent      # observe / tools / SPT plugin
  - agent_id: db-agent        # optional data
```

---

## PR → QA path (after extract)

```text
PR / demand
  → support-agent (orchestrate)
      → qa-agent (execute cases)
      → ui-test-agent / tool-agent (as routed)
  → support-agent (verdict + PR comment)
```

LLM: not required on happy path. Optional later only in support-agent.

---

## Corrections from earlier drafts

| Earlier | This revision |
|---------|---------------|
| Extract **spt-agent** | Extract **qa-agent** instead |
| Perf as primary extract | SPT path unchanged for now |
| Orchestration ambiguity | Explicit: **all orchestration out** of qa-agent |

---

## Review sign-off

1. Confirm extract `am-agents/qa-agent/`  
2. Confirm orchestration **only** in support-agent  
3. Confirm catalog stays outside qa-agent  
4. Decide frontend routing (ui-test vs qa-agent)  
5. Approve phases in [phases/PHASES.md](./phases/PHASES.md)  
