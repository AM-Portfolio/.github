# tool-agent — QA executor plan

**Canonical name:** `tool-agent`  
**Path:** `am-agents/tool-agent/`  
**Role in QA:** Execute backend, network, load/perf, and observe/verify checks.

---

## Purpose

tool-agent is a **specialist executor**. support-agent calls it over HTTP; it does not orchestrate other agents.

Existing API: `discover` → `plan` → `execute` → `stream`  
Default port: **8141** (from `registry/agents.yaml`)

---

## QA domains handled

| Domain | tool-agent surface | Catalog source |
|--------|-------------------|----------------|
| Backend API smoke | `tools.execute` + sandbox curl/HTTP | `catalog/qa/backend/` |
| Backend test suites | `tools.execute` (allowed commands) | `catalog/qa/backend/` |
| Network (DNS, TLS, latency) | `tools.execute` | `catalog/qa/network/` |
| Performance (load) | `tools/spt/` capability plugin | `catalog/qa/perf/` |
| Metrics / logs checks | `tools/observe/` | `catalog/verify/` |
| Infra probes | grafana, vault, kafka, etc. | as needed |

---

## Existing plugins (reuse)

```text
tool-agent/tools/
├── spt/           # load / perf scenarios
├── observe/       # metrics + logs
├── postgres/      # read-only queries
├── redis/
├── kafka/
├── mongodb/
├── qdrant/
├── grafana/
├── alert/
└── ...
```

Capability plugins for support-agent orchestration: `work-item`, `chat`, `mail`, `document`, `directory`, `observe`, `spt` — see `tool-agent/docs/CAPABILITY_PLUGINS.md`.

---

## Safety (existing — apply to QA)

| Rule | Behavior |
|------|----------|
| Command allowlist | Only approved prefixes in sandbox |
| Host allowlist | Declared in catalog / demand |
| Writes blocked | Most adapters read-only by default |
| Secrets | Vault / SecretBroker — never logged |
| MCP | Optional per tool manifest |

---

## QA catalog entry → execute (planned)

Example backend target (conceptual YAML):

```yaml
id: api-health-preview
kind: backend
enabled: true
tags: [backend, smoke, preview]
specialist: tool-agent
capability: tools.execute
params:
  method: GET
  url_secret_ref: preview-base-url
  path: /health
  expect_status: 200
```

support-agent passes resolved params; tool-agent executes inside sandbox.

---

## Performance execution (reuse existing plugin)

- Entries under `catalog/qa/perf/`
- Runner: k6 via ToolSandbox (tool-agent `tools/spt/`)

QA plan routes perf targets through the same plugin — no duplicate runner.

---

## What not to add

- QA orchestration logic in tool-agent
- New HTTP server for QA
- Hardcoded service names in Python

---

## Tests to add (Phase 1–2)

| Test | Type |
|------|------|
| QA backend smoke manifest executes | integration |
| Network probe allowlist enforced | unit |
| SPT plugin unchanged by QA work | regression |

---

## References

- [tool-agent/docs/ADDING_A_TOOL.md](https://github.com/AM-Portfolio/am-agents/blob/main/tool-agent/docs/ADDING_A_TOOL.md)
- [ADR-004 SPT catalog](https://github.com/AM-Portfolio/am-agents/blob/main/docs/agent-platform/decisions/ADR-004-spt-catalog-selectors.md)
