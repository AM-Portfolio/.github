# Alignment with `am-agents`

How this QA plan maps to the **existing** AM-Portfolio agent platform.

---

## Repo naming

| You may say | Actual repo / path |
|-------------|-------------------|
| am-agent | **`AM-Portfolio/am-agents`** (monorepo) |
| Support agent | **`am-agents/support-agent/`** |
| Tool agent | **`am-agents/tool-agent/`** |
| Fin agent | **`AM-Portfolio/am-fin-agent`** — finance only, **not QA** |

---

## Agent mapping

| QA plan role | am-agents component | Notes |
|--------------|---------------------|-------|
| Orchestrator | **support-agent** | `QaRunWorkflow`, router, RunStore, A2A |
| Scope / plan | **support-agent** `intelligence/` + `catalog/` | Was mislabeled "Fin Agent" in v1 plan |
| Backend execute | **tool-agent** | plan/execute HTTP, sandbox |
| Frontend execute | **ui-test-agent** | Playwright, baselines |
| Load / performance | **tool-agent** `tools/spt/` + `catalog/qa/perf/` | ADR-004 selectors |
| Health / metrics | **tool-agent** `tools/observe/` + `catalog/verify/` | check_ref templates |
| Data checks | **db-agent** | Optional, incident-driven |
| Final verdict | **support-agent** verify + RunStore | `overall_status`, notify |
| Finance | **fin-agent** | Out of scope for QA |

---

## What already exists (reuse, do not rebuild)

| Capability | Location | Status |
|------------|----------|--------|
| Specialist registry | `support-agent/registry/agents.yaml` | Live |
| QA catalog + schema | `catalog/qa/` | Live (planned) |
| QA workflow scaffold | `support-agent/.../workflows/qa_run.py` | Planned |
| QA activities | `support-agent/.../activities/qa.py` | Planned |
| Tool-agent perf plugin | `tool-agent/tools/spt/` | Live |
| UI E2E agent | `ui-test-agent/` | Live |
| Verify checks catalog | `catalog/verify/checks.yaml` | Live |
| Platform ports (RunStore, SPT DTOs) | `libs/platform-ports/` | Live |
| ADR-004 SPT selectors | `docs/agent-platform/decisions/ADR-004-*` | Accepted |
| Observability | support-agent Prometheus + OTLP | Live |
| CI for support-agent | `.github/workflows/am-support-agent.yml` | Live |

---

## What this plan adds (net new)

| Item | Phase |
|------|-------|
| `catalog/qa/` — backend, frontend, system, network entries | 1 |
| QA demand schema (`QaDemandRequest`) | 1 |
| `qa_run` workflow or `kind: qa` on existing workflow | 2 |
| PR trigger via `am-pipelines` | 2 |
| `/qa` PR commands | 2 |
| ADR-006 QA catalog decision doc | 1 |
| RunStore `kind=qa` analytics | 4 |

---

## v1 plan corrections

The first draft in this folder incorrectly used **"Fin Agent (Find + Finalize)"**. Corrections:

| v1 (wrong) | v2 (this plan) |
|------------|----------------|
| New QA orchestrator package | **support-agent** |
| Fin Agent = scope + verdict | **support-agent** planner + verify |
| fin-agent in QA diagram | **Removed** — finance is separate |
| Plan implies new executors | **tool-agent** + **ui-test-agent** |
| Generic JSON contracts | **am_platform_ports** DTOs |
| Implementation in `.github` | **am-agents** only |

---

## Specialist routing (from registry)

```yaml
# support-agent/registry/agents.yaml (existing)
defaults:
  prefer: tool-agent

agents:
  - agent_id: tool-agent      # backend, network, SPT, observe
  - agent_id: ui-test-agent   # frontend E2E
  - agent_id: db-agent        # optional data
```

QA catalog entries declare which capability to call — router resolves URL from registry.

---

## Performance vs functional QA

Both live under **qa-agent** / `catalog/qa/`. Performance targets use `catalog/qa/perf/` and the existing tool-agent `spt` plugin. Selector and RunStore rules follow ADR-004.

---

## PR Agent vs QA Agent

| | Gemini PR Agent (`.github`) | QA / SPT (am-agents) |
|---|----------------------------|----------------------|
| Repo | `.github` + `am-pipelines` | `am-agents` |
| Action | Code review, describe | Run tests, produce verdict |
| Trigger | PR opened | PR + `/qa` + QA demand |
| Model | Gemini 1.5 Flash | LLM optional in specialists |
| Complementary | Yes — review + execute both |

---

## Review sign-off targets

1. Confirm orchestrator = **support-agent** only  
2. Confirm **fin-agent** excluded from QA  
3. Confirm **catalog/qa/** location  
4. Approve phase order in [phases/PHASES.md](./phases/PHASES.md)  
5. Approve move to `am-agents/docs/agent-platform/` after review  
