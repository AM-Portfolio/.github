# qa-agent — specialist module plan

**Canonical name:** `qa-agent`  
**Path:** `am-agents/qa-agent/` (new module)  
**Role:** Execute QA tests only — backend, frontend, system, network (and related checks).

---

## Hard boundary

| In qa-agent | Out of qa-agent (stays in support-agent) |
|-------------|------------------------------------------|
| Run concrete QA cases | Temporal workflows (`QaRunWorkflow`) |
| Backend / API smoke & suite runners | Catalog resolve + selector expand |
| Frontend E2E runners (or call ui-test-agent as a library — prefer HTTP via orchestrator) | Fan-out, failure_mode, budgets |
| Network probes (DNS, TLS, latency) | Parent RunStore + final verdict |
| Collect evidence (logs, screenshots, metrics) | PR comments, notify, HITL |
| Health / ready + execute API | Target policy / max targets / empty-selector fatal |

**Rule:** qa-agent never imports support-agent, never expands selectors, never fans out sibling targets, never owns orchestration.

---

## Why extract

Today QA is fragmented (repo CI, ui-test-agent, tool-agent, SPT plugin). A dedicated **`qa-agent`** specialist:

- Matches `ui-test-agent` / `db-agent` pattern (domain specialist)
- Scales test runners independently of support-agent
- Keeps support-agent thin: plan → route → verify → report
- Clarifies: **support-agent orchestrates; qa-agent executes**

> **Note:** This plan extracts **qa-agent**, not a separate spt-agent. Load/perf (SPT) may remain on existing `tool-agent/tools/spt/` + `SptRunWorkflow` unless later folded under qa-agent `kind: perf` — open decision.

---

## Target tree

```text
am-agents/qa-agent/
├── README.md
├── pyproject.toml                 # package am_qa_agent
├── Dockerfile
├── helm/
│   ├── values.yaml
│   ├── values.dev.yaml
│   ├── values.preprod.yaml
│   └── values.prod.yaml
├── .env.example
├── observability.yaml
├── app/                           # or src/am_qa_agent/
│   ├── main.py                    # FastAPI: health, ready, execute, status
│   ├── config.py
│   ├── api/
│   │   ├── execute.py             # POST run one target / case
│   │   ├── status.py              # GET  status/{ref}
│   │   └── cancel.py              # POST cancel
│   ├── runners/
│   │   ├── backend.py             # pytest / HTTP smoke / contract
│   │   ├── frontend.py            # Playwright bridge or local runner
│   │   ├── system.py              # multi-step journey scripts
│   │   ├── network.py             # dig / TLS / latency
│   │   └── base.py
│   ├── safety.py                  # allowlists, sandbox, host allowlist
│   ├── schemas.py                 # request/response (or am_platform_ports)
│   └── observability/
├── tests/
│   ├── contract/
│   └── unit/
└── scripts/
```

Layout style: same family as `ui-test-agent/` / `db-agent/` — specialist app, **not** platform_worker.

---

## HTTP contract (called by support-agent)

| Op | Purpose | Returns |
|----|---------|---------|
| `execute` | Run one resolved QA target | `{ async_operation_ref \| result, status }` |
| `status` | Poll async run | `{ status, evidence_refs?, error? }` |
| `cancel` | Abort run | `{ status: cancelled }` |

Request carries **already-resolved** params from support-agent (no catalog expand inside qa-agent):

```json
{
  "target_id": "api-health-preprod",
  "kind": "backend",
  "params": { "path": "/health", "url_secret_ref": "..." },
  "sandbox": true,
  "demand_ref": "qa-pr-42-abc"
}
```

Auth: `X-Agent-Caller: support-agent`.

**Does not expose:** plan selector, expand catalog, create parent RunStore, notify PR.

---

## Domains (runners)

| `kind` | Runner | Examples |
|--------|--------|----------|
| `backend` | backend.py | pytest, API smoke, contract |
| `frontend` | frontend.py | Playwright E2E (or delegate URL only — orchestrator may still prefer ui-test-agent) |
| `system` | system.py | Synthetic multi-step scripts |
| `network` | network.py | DNS, TLS, latency |
| `perf` | optional later | Only if folding SPT into qa-agent |

Open decision: frontend via **qa-agent** vs keep routing to existing **ui-test-agent**. Plan default: support-agent may route `frontend` → ui-test-agent **or** qa-agent; prefer one path after pilot.

---

## Registry entry (support-agent)

```yaml
  - agent_id: qa-agent
    display_name: QA Agent
    base_url_env: QA_AGENT_BASE_URL
    default_base_url: http://127.0.0.1:8160
    preferred: true
    health_path: /health
    ready_path: /ready
    capabilities:
      - id: qa.execute
        ops: [execute, status, cancel]
      - id: qa.status
        ops: [status]
```

---

## Orchestration stays in support-agent

```text
PR / QA demand
    → support-agent QaRunWorkflow
         ├─ resolve catalog/qa
         ├─ expand selector
         ├─ policy / sandbox / max targets
         ├─ for each target (bounded fan-out):
         │      HTTP → qa-agent.execute / status
         │      (optional) HTTP → ui-test-agent / tool-agent
         ├─ optional observe via tool-agent
         └─ RunStore summary + PR notify
```

qa-agent sees **one target execute request**, not the whole demand matrix.

---

## LLM

| In qa-agent | LLM? |
|-------------|------|
| execute / status / cancel | **No** (default) |
| Scripted runners | **No** |
| Optional design-review style hooks | Only if explicitly enabled later |

LLM for “which tags from this PR?” stays in **support-agent** planner (optional, later).

---

## Safety

- Host / command allowlists
- Sandbox for destructive or load-adjacent ops
- No prod writes by default
- Secrets via SecretBroker / vault mappings — never logged

---

## Relationship to other specialists

| Specialist | After qa-agent extract |
|------------|------------------------|
| **support-agent** | Orchestrator only |
| **qa-agent** ★ | Primary QA executor |
| **ui-test-agent** | May remain frontend specialist or be called only for advanced UI |
| **tool-agent** | Infra tools + observe; SPT plugin until separate decision |
| **db-agent** | Optional data checks |

---

## Non-goals for this module

- Becoming the QA **orchestrator**
- Temporal workflows inside qa-agent
- Catalog resolve / selector expand
- Fan-out to other specialists
- PR comment / Cliq notify ownership

See [../PLAN.md](../PLAN.md) and [../FOLDER_STRUCTURE.md](../FOLDER_STRUCTURE.md).
