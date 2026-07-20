# Target folder structure (am-agents)

Planning layout for **qa-agent extract** inside **`AM-Portfolio/am-agents`**.

Legend: **exists** · **planned**

---

## Repo map

| Path | Role | Status |
|------|------|--------|
| `support-agent/` | **Orchestration only** | **exists** — add QaRun + qa-agent adapter |
| `qa-agent/` | **New specialist** — QA execute only | **planned ★** |
| `ui-test-agent/` | Frontend E2E specialist | **exists** |
| `tool-agent/` | Tools, observe; SPT plugin | **exists** |
| `db-agent/` | Optional data | **exists** |
| `catalog/qa/` | QA targets (data) | **planned** — owned outside qa-agent |
| `catalog/verify/` | Health/metrics templates | **exists** |
| `catalog/spt/` | Perf targets (existing SPT path) | **exists** |
| `libs/platform-ports/` | Shared DTOs | **exists** |

**Lock:** No orchestration package inside `qa-agent/`. No Temporal worker inside `qa-agent/`.

> This `.github/qa-agent/` folder is **plan docs only**. Implementation module is `am-agents/qa-agent/`.

---

## 1. `am-agents` — target

```text
am-agents/
├── catalog/
│   ├── qa/                               # planned — data only
│   │   ├── backend/
│   │   ├── frontend/
│   │   ├── system/
│   │   └── network/
│   ├── verify/                           # exists
│   ├── spt/                              # exists — SPT path unchanged
│   └── prompts/
│
├── support-agent/                        # ORCHESTRATION
│   └── src/am_support_agent/
│       ├── orchestrator/
│       │   ├── workflows/
│       │   │   ├── qa_run.py             # planned
│       │   │   └── spt_run.py            # exists — SPT stays here
│       │   ├── activities/
│       │   │   ├── qa.py                 # planned — HTTP to qa-agent
│       │   │   └── spt.py                # exists
│       │   └── router.py
│       ├── adapters/
│       │   └── qa_agent/                 # planned — HTTP client
│       ├── registry/agents.yaml          # add qa-agent entry
│       └── stores/                       # RunStore stays here
│
├── qa-agent/                             # NEW SPECIALIST ★ — NO ORCHESTRATION
│   ├── README.md
│   ├── pyproject.toml
│   ├── Dockerfile
│   ├── helm/
│   ├── app/                              # or src/am_qa_agent/
│   │   ├── main.py                       # health, ready, execute, status, cancel
│   │   ├── runners/
│   │   │   ├── backend.py
│   │   │   ├── frontend.py
│   │   │   ├── system.py
│   │   │   └── network.py
│   │   ├── safety.py
│   │   └── schemas.py
│   └── tests/
│
├── tool-agent/                           # unchanged for this extract
├── ui-test-agent/                        # unchanged
├── db-agent/
│
└── docs/agent-platform/
    └── decisions/
        └── ADR-006-qa-agent-extract.md   # planned
```

---

## 2. What lives where

| Concern | Module |
|---------|--------|
| PR / demand trigger | am-pipelines → support-agent |
| Selector expand, fan-out, failure_mode | support-agent |
| RunStore parent + summary | support-agent |
| Backend / system / network runners | **qa-agent** |
| Playwright E2E | ui-test-agent and/or qa-agent |
| Observe / SPT load | tool-agent + existing SPT workflow |
| Catalog YAML | `catalog/` (data) |

---

## 3. This `.github` repo (plan only)

```text
.github/
└── qa-agent/                 # keep plan documents (this folder)
    ├── PLAN.md
    ├── agents/qa-agent.md
    └── ...
```

---

## 4. Import / coupling rules

```text
support-agent  →  am_platform_ports
support-agent  →  qa-agent / ui-test-agent / tool-agent  (HTTP only)
qa-agent       →  am_platform_ports (optional) + runners
qa-agent       ✗  support-agent
qa-agent       ✗  Temporal workflows
qa-agent       ✗  catalog writer / selector expand
qa-agent       ✗  fan-out to other specialists
```

---

## 5. What must not appear in `am-agents/qa-agent/`

| Forbidden | Why |
|-----------|-----|
| `orchestrator/`, Temporal worker | Orchestration is support-agent |
| `QaRunWorkflow` | Parent workflow stays in support-agent |
| Catalog resolve / selector expand | ADR-004 — orchestrator responsibility |
| Fan-out to other specialists | Single-target executor |
| PR comment / Cliq notify | support-agent / am-pipelines |
| Import of support-agent packages | Boundary break |

---

## 6. Phase → folders

| Phase | Adds |
|-------|------|
| 0 | Keep plan in `.github/qa-agent/` |
| 1 | `am-agents/qa-agent/` scaffold + Helm + contract tests |
| 2 | support-agent adapter + `QaRunWorkflow`; pilot |
| 3 | PR `/qa`; system/frontend routing |
| 4 | Dashboards / optional LLM in support-agent only |

See [phases/PHASES.md](./phases/PHASES.md).
