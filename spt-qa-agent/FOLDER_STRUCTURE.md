# Target folder structure (am-agents)

Planning layout for SPT / QA work inside **`AM-Portfolio/am-agents`**.  
Follows the existing SoT: `docs/agent-platform/FOLDER_STRUCTURE.md`.

Legend: **exists** = already in repo · **planned** = this QA plan adds.

---

## Repo map

| Path | Role | Status |
|------|------|--------|
| `support-agent/` | QA orchestrator — scope, route, verify, report | **exists** — extend |
| `tool-agent/` | Backend, network, SPT, observe execution | **exists** |
| `ui-test-agent/` | Frontend Playwright E2E | **exists** |
| `db-agent/` | Optional data checks | **exists** |
| `catalog/spt/` | Perf/load targets | **exists** |
| `catalog/verify/` | Health/metrics checks | **exists** |
| `catalog/qa/` | QA test matrix by domain | **planned** |
| `catalog/prompts/` | Prompt data | **exists** |
| `libs/platform-ports/` | Shared DTOs (`SptDemandRequest`, RunStore) | **exists** |
| `libs/platform-adapters/` | Grafana, OpenProject, vault, etc. | **exists** |
| `am-pipelines` | PR trigger reusable workflow | **external** |

**Not in scope:** new top-level `qa-agent/` package — QA extends support-agent + catalog.

---

## 1. `am-agents` — QA additions (planned)

```text
am-agents/
├── catalog/
│   ├── spt/                              # exists — perf/load
│   ├── verify/                           # exists — check_ref templates
│   ├── qa/                               # planned ★
│   │   ├── README.md
│   │   ├── selectors.schema.json
│   │   ├── backend/
│   │   │   └── *.yaml                    # API smoke, pytest refs
│   │   ├── frontend/
│   │   │   └── *.yaml                    # ui-test-agent scenario refs
│   │   ├── system/
│   │   │   └── *.yaml                    # multi-service journeys
│   │   └── network/
│   │       └── *.yaml                    # DNS/TLS/latency probes
│   └── prompts/                          # exists
│
├── support-agent/                        # exists — extend
│   ├── src/am_support_agent/
│   │   ├── orchestrator/
│   │   │   ├── workflows/
│   │   │   │   ├── spt_run.py            # exists
│   │   │   │   └── qa_run.py             # planned (or extend spt_run)
│   │   │   ├── activities/
│   │   │   │   ├── spt.py                # exists
│   │   │   │   └── qa.py                 # planned
│   │   │   ├── router.py                 # exists — add qa routing
│   │   │   └── runner.py                 # exists
│   │   ├── intelligence/
│   │   │   └── catalog.py                # exists — add list_qa()
│   │   ├── registry/agents.yaml          # exists
│   │   └── stores/                       # exists — RunStore
│   └── docs/
│       └── qa-orchestration.md           # planned
│
├── tool-agent/                           # exists — no structural change
│   └── tools/
│       ├── spt/                          # exists
│       ├── observe/                      # exists
│       └── ...                           # postgres, redis, etc.
│
├── ui-test-agent/                        # exists — no structural change
│   └── app/agent/                        # planner, executor, reporter
│
├── docs/agent-platform/
│   ├── decisions/
│   │   └── ADR-006-qa-catalog.md         # planned
│   └── ...
│
└── .github/workflows/
    └── am-support-agent.yml              # exists — add qa contract tests later
```

---

## 2. `support-agent/` module tree (reference)

Matches the canonical support-agent target tree from `support-agent/README.md`:

```text
support-agent/
├── README.md
├── contracts/                 # A2A, capabilities, evidence
├── registry/agents.yaml       # specialist endpoints
├── adapters/                  # tool_agent, ui_test_agent, db_agent
├── orchestrator/
│   ├── workflows/             # SptRun, QaRun, AlertIncident, A2A
│   ├── activities/
│   ├── router/
│   └── hitl/
├── intelligence/              # catalog reader, verification
├── memory/                    # procedural → catalog refs
├── stores/                    # RunStore
├── observability/
├── tests/
│   ├── contract/
│   ├── parity/
│   └── e2e/
└── deploy/helm/
```

QA work plugs into **orchestrator**, **intelligence**, and **catalog** — not a parallel tree.

---

## 3. This `.github` repo (plan only)

```text
.github/                         # org config repo
├── .github/workflows/
│   └── pr-agent.yml             # exists — Gemini PR review
└── spt-qa-agent/                # this keep plan (review draft)
    ├── PLAN.md
    ├── FOLDER_STRUCTURE.md
    ├── ALIGNMENT.md
    └── ...
```

After review approval, copy the approved plan into `am-agents/docs/agent-platform/` and implement there.

---

## 4. Import / coupling rules (unchanged)

```text
support-agent orchestrator  →  am_platform_ports (schemas)
support-agent runtime       →  specialists via HTTP (registry/agents.yaml)
tool-agent / ui-test-agent  →  no import of support-agent
catalog/qa/                 →  data only — no Python logic
```

---

## 5. What must not appear

| Forbidden | Why |
|-----------|-----|
| New `qa-agent/` orchestrator package | Duplicates support-agent |
| Service names in workflow Python | ADR-004 — use catalog |
| QA logic in `.github` repo | Wrong repo — am-agents only |
| Using `fin-agent` for QA scope | fin-agent = finance product |
| Forked platform-ports schemas | ADR-003 extractable SDK |

---

## 6. Phase → folders

| Phase | Adds |
|-------|------|
| 0 | Keep plan in `.github/spt-qa-agent/` (this review) |
| 1 | `catalog/qa/` + schema; `list_qa()` in CatalogReader |
| 2 | `qa_run` workflow/activities; am-pipelines PR trigger |
| 3 | system + network catalog entries; verify integration |
| 4 | RunStore analytics; flaky + risk-based selection |

See [phases/PHASES.md](./phases/PHASES.md).
