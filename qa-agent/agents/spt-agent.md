# spt-agent — specialist module plan

**Canonical name:** `spt-agent`  
**Path:** `am-agents/spt-agent/` (new module)  
**Role:** Execute performance / load / synthetic tests only.

---

## Hard boundary

| In spt-agent | Out of spt-agent (stays in support-agent) |
|--------------|-------------------------------------------|
| Prep / execute / status / cancel | Temporal workflows (`SptRunWorkflow`) |
| k6 (and other engines) in sandbox | Catalog resolve + selector expand |
| Scenario runner + metrics export | Fan-out, failure_mode, budgets |
| Child result payload | RunStore parent run + final verdict |
| Health / ready endpoints | PR comments, notify, HITL |
| Local safety allowlists | Target policy / max targets |

**Rule:** spt-agent never imports support-agent, never starts sibling targets, never owns orchestration.

---

## Why extract

Today SPT lives as `tool-agent/tools/spt/` capability plugin. That couples load engines to the generic tool runtime.

Extracting `spt-agent/`:

- Matches `ui-test-agent` pattern (domain specialist)
- Lets SPT scale (CPU/memory for k6) independently
- Keeps tool-agent focused on data/infra tools
- Clarifies: support-agent orchestrates; specialists execute

---

## Target tree

```text
spt-agent/
├── README.md
├── pyproject.toml                 # package am_spt_agent
├── Dockerfile
├── helm/
│   ├── values.yaml
│   ├── values.dev.yaml
│   ├── values.preprod.yaml
│   └── values.prod.yaml
├── .env.example
├── observability.yaml
├── app/                           # or src/am_spt_agent/
│   ├── main.py                    # FastAPI: health, ready, A2A-ish ops
│   ├── config.py
│   ├── api/
│   │   ├── prepare.py             # POST .../prepare
│   │   ├── execute.py             # POST .../execute
│   │   ├── status.py              # GET  .../status/{ref}
│   │   └── cancel.py              # POST .../cancel
│   ├── engines/
│   │   ├── base.py
│   │   ├── memory.py              # lab stub
│   │   └── k6.py                  # sandbox-gated k6
│   ├── safety.py                  # sandbox required, host allowlist
│   ├── schemas.py                 # request/response DTOs (or am_platform_ports)
│   └── observability/
├── tests/
│   ├── contract/
│   └── unit/
└── scripts/
```

Mirror layout style of `ui-test-agent/` / `db-agent/` — specialist app, not platform_worker.

---

## HTTP contract (called by support-agent)

| Op | Purpose | Returns |
|----|---------|---------|
| `prepare` | Ensure dataset / warm scenario (`prep_ref`) | `{ prep_ref, ready }` |
| `execute` | Start load run (sandbox required) | `{ async_operation_ref, status }` |
| `status` | Poll run | `{ status, metrics?, error? }` |
| `cancel` | Abort run | `{ status: cancelled }` |

Auth: `X-Agent-Caller: support-agent` (same pattern as tool-agent / db-agent).

**Does not expose:** plan selector, expand catalog, create parent RunStore, notify PR.

---

## Migration from tool-agent

| From | To |
|------|----|
| `tool-agent/tools/spt/` | `spt-agent/engines/` + API |
| `SPT_PROVIDER=memory\|k6` | spt-agent env |
| Capability `spt.*` via tool-agent | Direct HTTP to spt-agent |
| `TOOL_AGENT_CAPABILITY_PLUGINS=spt` | Remove after cutover |

Phased cutover:

1. Stand up `spt-agent` beside plugin (parity tests)
2. Point support-agent registry at `spt-agent`
3. Deprecate `tool-agent/tools/spt/`

---

## Registry entry (support-agent)

```yaml
  - agent_id: spt-agent
    display_name: SPT Agent
    base_url_env: SPT_AGENT_BASE_URL
    default_base_url: http://127.0.0.1:8150
    preferred: true
    health_path: /health
    ready_path: /ready
    capabilities:
      - id: spt.prepare
        ops: [execute]
      - id: spt.execute
        ops: [execute, status, cancel]
      - id: spt.status
        ops: [status]
```

---

## Orchestration stays in support-agent

```text
PR / SPT demand
    → support-agent SptRunWorkflow
         ├─ resolve catalog/spt (or catalog/qa/perf)
         ├─ expand selector
         ├─ policy / sandbox / max targets
         ├─ for each target (bounded fan-out):
         │      HTTP → spt-agent prepare / execute / status
         ├─ optional observe via tool-agent
         └─ RunStore summary + PR notify
```

spt-agent sees **one target’s execute request**, not the whole demand matrix.

---

## LLM

| In spt-agent | LLM? |
|--------------|------|
| prepare / execute / status / cancel | **No** |
| k6 run | **No** |
| Free-text chat intent | Out of scope for PR→SPT path |

LLM for “which tags from this PR?” stays in support-agent planner (optional, later).

---

## Safety

- `sandbox=true` required for k6 (same rule as current plugin)
- No prod writes; staging/preprod hosts from secret refs only
- Secrets never logged; SecretBroker at caller or local vault mapping
- Rate / concurrency limits local to the pod; org fan-out limit in support-agent

---

## Tests

| Test | Owner |
|------|-------|
| Contract: prepare→execute→status | spt-agent |
| Sandbox deny without flag | spt-agent |
| support-agent routes to spt-agent mock | support-agent |
| Parity vs old tool-agent spt plugin | cutover phase |

---

## Non-goals for this module

- Becoming the QA orchestrator
- Owning `catalog/spt` writer UI
- Calling ui-test-agent or other specialists
- Temporal workflows inside spt-agent

See [../PLAN.md](../PLAN.md) and [../FOLDER_STRUCTURE.md](../FOLDER_STRUCTURE.md).
