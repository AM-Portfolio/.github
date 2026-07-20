# ui-test-agent — frontend QA executor plan

**Canonical name:** `ui-test-agent`  
**Path:** `am-agents/ui-test-agent/`  
**Role in QA:** Execute frontend / UI E2E tests via Playwright.

---

## Purpose

ui-test-agent is the **frontend specialist**. support-agent routes all UI/E2E QA targets here — not to tool-agent.

Default port: **8130** (from `registry/agents.yaml`)  
Capabilities: `ui.test.run`, `ui.test.report`

---

## QA domains handled

| Domain | ui-test-agent feature | Notes |
|--------|----------------------|-------|
| E2E regression | Playwright walks | `app/agent/executor.py` |
| Auth flows | Keycloak profiles | `app/profiles/` |
| Visual baseline | Screenshot compare | preprod baseline commands |
| Design review | LLM-assisted review | `app/agent/design_review.py` |
| Accessibility smoke | Optional axe in Playwright | Phase 3 |
| Reporting | JSON + screenshots | `app/agent/reporter.py` |

---

## Existing stack (reuse)

```text
ui-test-agent/
├── app/
│   ├── agent/          # planner, executor, reporter, graph (LangGraph)
│   ├── browser/        # Playwright controller
│   ├── profiles/       # auth flows (modern_ui, etc.)
│   ├── llm/            # LiteLLM / gateway client
│   └── memory/qdrant/  # visual element matching
├── scripts/            # e2e preprod runners
└── tests/
```

NPM workspace: `@am/ui-test-agent`

Commands (existing):

- `npm run test:e2e:preprod`
- `npm run test:e2e:preprod:compare`

---

## QA catalog entry → execute (planned)

Example frontend target:

```yaml
id: checkout-flow-preview
kind: frontend
enabled: true
tags: [frontend, e2e, checkout]
specialist: ui-test-agent
capability: ui.test.run
params:
  spec_ref: e2e/checkout.spec.ts
  base_url_secret_ref: preview-base-url
  profile: modern_ui
```

support-agent sends demand; ui-test-agent returns status + artifact refs.

---

## Environment requirements

| Variable | Purpose |
|----------|---------|
| Preview / preprod URL | Target app (from secret ref) |
| `KEYCLOAK_TOKEN_URL` | Auth flows |
| `QDRANT_HOST` | Visual matching (optional) |
| `LANGFUSE_ENABLED` | Trace planner decisions |

Preview URL discovery for PR QA is an **open decision** (see [PLAN.md](../PLAN.md)).

---

## Integration with support-agent

```text
support-agent
  POST ui-test-agent /... (ui.test.run)
    ← plan + execute result
    ← screenshots / trace on failure
  verify step merges into QaRunSummary
```

No changes to ui-test-agent core architecture — add catalog refs and optional API wrapper for demand shape from support-agent.

---

## What not to add

- Backend or network tests in ui-test-agent
- Orchestration / RunStore ownership
- Duplicate Playwright runner outside this package

---

## Tests to add (Phase 2)

| Test | Type |
|------|------|
| Catalog spec_ref resolves and runs | integration (staging) |
| Failure artifacts returned to support-agent | contract |

---

## References

- [ui-test-agent/README.md](https://github.com/AM-Portfolio/am-agents/blob/main/ui-test-agent/README.md)
