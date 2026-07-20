# catalog/ — QA data plan

Catalog holds **targets and check templates** — no Python logic (ADR-004, platform design SoT).

---

## Existing catalogs (am-agents)

| Path | Purpose |
|------|---------|
| `catalog/verify/` | Metrics, logs, health check templates |
| `catalog/prompts/` | Prompt bodies for agents |
| `catalog/spt/` | Legacy perf targets — migrate refs into `catalog/qa/perf/` |

---

## Planned: `catalog/qa/`

```text
catalog/qa/
├── README.md
├── selectors.schema.json       # { ids, tags } — mirror ADR-004
├── target.schema.json          # QA target shape
├── backend/
│   └── *.yaml                  # API smoke, pytest refs
├── frontend/
│   └── *.yaml                  # ui-test-agent scenario refs
├── system/
│   └── *.yaml                  # multi-step journeys (fan-out)
├── network/
│   └── *.yaml                  # DNS, TLS, latency probes
└── perf/
    └── *.yaml                  # load/k6 scenario refs
```

---

## Target schema (planned fields)

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Unique target id |
| `kind` | yes | `backend` \| `frontend` \| `system` \| `network` |
| `enabled` | yes | Skip if false |
| `tags` | yes | Selector tags |
| `specialist` | yes | `tool-agent` \| `ui-test-agent` |
| `capability` | yes | e.g. `tools.execute`, `ui.test.run` |
| `params` | yes | Opaque to catalog — resolved at runtime |
| `verify_ref` | no | Link to `catalog/verify/` check |
| `priority` | no | `P0` \| `P1` \| `P2` — default P1 |

Secrets: `*_secret_ref` keys only — resolved via SecretBroker (ADR-002).

---

## Selector rules (locked — same as SPT)

| Rule | Behavior |
|------|----------|
| Empty selector | **Fatal** — no implicit run-all |
| `{ ids: [...] }` | Run listed targets |
| `{ tags: [...] }` | Run targets matching any tag |
| `{ all: true }` | Rejected unless lab + approval + under max count |

Env guards: `QA_MAX_TARGETS_PER_RUN` (default 20; prod 5) — same rules as ADR-004.

---

## Example entries

**Backend smoke**

```yaml
id: api-health-preprod
kind: backend
enabled: true
tags: [backend, smoke, preprod]
specialist: tool-agent
capability: tools.execute
params:
  check_ref: verify/http-200
  url_secret_ref: preprod-api-base
  path: /health
priority: P0
```

**Frontend E2E**

```yaml
id: auth-login-e2e
kind: frontend
enabled: true
tags: [frontend, e2e, auth]
specialist: ui-test-agent
capability: ui.test.run
params:
  spec_ref: e2e/auth-login.spec.ts
  profile: modern_ui
  base_url_secret_ref: preprod-ui-base
priority: P0
```

**Network TLS**

```yaml
id: tls-api-preprod
kind: network
enabled: true
tags: [network, tls, preprod]
specialist: tool-agent
capability: tools.execute
params:
  probe: tls
  host_secret_ref: preprod-api-host
priority: P1
```

---

## Resolution flow

```text
QaDemandRequest.selector
    → CatalogReader.list_qa()
    → TargetResolver.expand()
    → TargetSet
    → support-agent router
```

Implementation: extend `support-agent/intelligence/catalog.py` (Phase 1).

---

## Related

- ADR-004 selector pattern: `docs/agent-platform/decisions/ADR-004-spt-catalog-selectors.md`
