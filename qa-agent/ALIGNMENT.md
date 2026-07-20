# Alignment with `am-agents`

How this plan maps to the platform — including **spt-agent extract**.

---

## Decision (locked for this draft)

| Decision | Detail |
|----------|--------|
| Extract SPT | New module `am-agents/spt-agent/` |
| Orchestration | **Only** in `support-agent/` |
| Migrate from | `tool-agent/tools/spt/` → deprecate after parity |
| Catalog | Stays in `catalog/spt/` (data); not inside spt-agent |

---

## Agent mapping

| Role | Component | Notes |
|------|-----------|-------|
| Orchestrator | **support-agent** | Workflows, selector, fan-out, RunStore, verdict |
| Load / perf execute | **spt-agent** ★ | prepare / execute / status / cancel |
| Backend / network / observe | **tool-agent** | No long-term SPT ownership |
| Frontend E2E | **ui-test-agent** | Unchanged |
| Data checks | **db-agent** | Optional |
| Finance | **fin-agent** | Out of scope |

---

## What already exists

| Capability | Location | After extract |
|------------|----------|---------------|
| `SptRunWorkflow` | support-agent | Keep — retarget calls to spt-agent |
| SPT activities | support-agent `activities/spt.py` | Keep — HTTP to spt-agent |
| SPT plugin | tool-agent `tools/spt/` | **Migrate then deprecate** |
| `catalog/spt/` | catalog/ | Unchanged (data) |
| ADR-004 selectors | docs | Unchanged — enforced in support-agent |
| Registry | support-agent `agents.yaml` | **Add spt-agent** |

---

## Net new

| Item | Phase |
|------|-------|
| `spt-agent/` module | 1 |
| support-agent adapter `spt_agent/` | 2 |
| Registry entry + env `SPT_AGENT_BASE_URL` | 1–2 |
| ADR-006 spt-agent extract | 1 |
| Deprecate tool-agent spt plugin | 3 |
| `catalog/qa/` functional matrix | 1–2 (parallel) |

---

## Specialist routing (target registry)

```yaml
agents:
  - agent_id: tool-agent      # backend, network, observe
  - agent_id: ui-test-agent   # frontend E2E
  - agent_id: spt-agent       # load / perf ★ NEW
  - agent_id: db-agent        # optional data
```

---

## PR → SPT path (after extract)

```text
PR / demand
  → support-agent (orchestrate)
      → spt-agent (execute k6)
      → tool-agent (optional observe)
  → support-agent (verdict + PR comment)
```

LLM: not required on happy path. Optional later only in support-agent planner / narrative.

---

## v2 → v3 corrections

| Earlier plan | This revision |
|--------------|---------------|
| SPT via tool-agent `tools/spt/` | Extract **spt-agent** module |
| Perf listed under tool-agent | Perf = **spt-agent** only |
| Orchestration ambiguity | Explicit: **all orchestration out** of spt-agent |

---

## Review sign-off

1. Confirm extract `spt-agent/`  
2. Confirm orchestration **only** in support-agent  
3. Confirm deprecate `tool-agent/tools/spt/` after parity  
4. Confirm catalog stays outside spt-agent  
5. Approve phases in [phases/PHASES.md](./phases/PHASES.md)  
