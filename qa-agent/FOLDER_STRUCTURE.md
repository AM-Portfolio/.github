# Target folder structure (am-agents)

Planning layout for **QA** + **spt-agent extract** inside **`AM-Portfolio/am-agents`**.

Legend: **exists** · **planned** · **migrate**

---

## Repo map

| Path | Role | Status |
|------|------|--------|
| `support-agent/` | **Orchestration only** — workflows, catalog, RunStore, route | **exists** — wire to spt-agent |
| `spt-agent/` | **New specialist** — load/perf execute only | **planned ★** |
| `tool-agent/` | Backend, network, observe | **exists** — drop spt plugin after cutover |
| `tool-agent/tools/spt/` | Legacy SPT plugin | **migrate → spt-agent** then deprecate |
| `ui-test-agent/` | Frontend E2E | **exists** |
| `db-agent/` | Optional data | **exists** |
| `catalog/spt/` | Perf targets (data) | **exists** — owned by platform, not spt-agent |
| `catalog/qa/` | Functional QA matrix | **planned** |
| `catalog/verify/` | Health/metrics templates | **exists** |
| `libs/platform-ports/` | Shared DTOs | **exists** |

**Lock:** No orchestration package inside `spt-agent/`. No Temporal worker inside `spt-agent/`.

---

## 1. `am-agents` — target

```text
am-agents/
├── catalog/
│   ├── spt/                              # exists — data only
│   ├── verify/                           # exists
│   ├── qa/                               # planned
│   └── prompts/
│
├── support-agent/                        # ORCHESTRATION
│   └── src/am_support_agent/
│       ├── orchestrator/
│       │   ├── workflows/
│       │   │   ├── spt_run.py            # exists — call spt-agent not tool-agent
│       │   │   └── qa_run.py             # planned
│       │   ├── activities/
│       │   │   ├── spt.py                # exists — retarget HTTP to spt-agent
│       │   │   └── qa.py                 # planned
│       │   └── router.py
│       ├── adapters/
│       │   └── spt_agent/                # planned — HTTP client
│       ├── registry/agents.yaml          # add spt-agent entry
│       └── stores/                       # RunStore stays here
│
├── spt-agent/                            # NEW SPECIALIST ★ — NO ORCHESTRATION
│   ├── README.md
│   ├── pyproject.toml
│   ├── Dockerfile
│   ├── helm/
│   ├── app/                              # or src/am_spt_agent/
│   │   ├── main.py                       # health, ready, prepare, execute, status, cancel
│   │   ├── engines/
│   │   │   ├── memory.py
│   │   │   └── k6.py
│   │   ├── safety.py
│   │   └── schemas.py
│   └── tests/
│
├── tool-agent/
│   └── tools/
│       ├── spt/                          # deprecate after cutover
│       ├── observe/                      # stays
│       └── ...
│
├── ui-test-agent/                        # unchanged
├── db-agent/                             # unchanged
│
└── docs/agent-platform/
    └── decisions/
        └── ADR-006-spt-agent-extract.md  # planned
```

---

## 2. What lives where

| Concern | Module |
|---------|--------|
| PR / demand trigger | am-pipelines → support-agent |
| Selector expand, fan-out, failure_mode | support-agent |
| RunStore parent + summary | support-agent |
| k6 / prep / status / cancel | **spt-agent** |
| API smoke, DNS, TLS | tool-agent |
| Playwright E2E | ui-test-agent |
| Catalog YAML files | `catalog/` (data) |

---

## 3. This `.github` repo (plan only)

```text
.github/
└── qa-agent/                 # this keep plan
    ├── PLAN.md
    ├── FOLDER_STRUCTURE.md
    ├── agents/spt-agent.md   # extract spec
    └── ...
```

---

## 4. Import / coupling rules

```text
support-agent  →  am_platform_ports
support-agent  →  spt-agent / tool-agent / ui-test-agent  (HTTP only)
spt-agent      →  am_platform_ports (optional DTOs) + engines
spt-agent      ✗  support-agent
spt-agent      ✗  Temporal workflows
spt-agent      ✗  catalog writer / selector expand
tool-agent     ✗  spt-agent (after cutover; no cross-call required)
```

---

## 5. What must not appear in `spt-agent/`

| Forbidden | Why |
|-----------|-----|
| `orchestrator/`, Temporal worker | Orchestration is support-agent |
| `SptRunWorkflow` | Parent workflow stays in support-agent |
| Catalog resolve / selector expand | ADR-004 — orchestrator responsibility |
| Fan-out to other specialists | Single-target executor |
| PR comment / Cliq notify | support-agent / am-pipelines |
| Import of support-agent packages | Boundary break |

---

## 6. Phase → folders

| Phase | Adds |
|-------|------|
| 0 | Keep plan (this review) |
| 1 | `spt-agent/` scaffold + Helm + contract tests |
| 2 | support-agent adapter + registry; dual-run parity |
| 3 | Deprecate `tool-agent/tools/spt/`; PR wiring |
| 4 | Dashboards / optional LLM scope in support-agent only |

See [phases/PHASES.md](./phases/PHASES.md).
