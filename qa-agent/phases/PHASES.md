# Phased rollout

Implementation phases for **QA agent** inside **`am-agents`**.  
Plan-only — no dates; ordered by dependency.

---

## Phase 0 — Review (current)

**Goal:** Align stakeholders on architecture before code.

| Deliverable | Location |
|-------------|----------|
| Keep plan | `.github/qa-agent/` |
| Alignment with am-agents | [ALIGNMENT.md](../ALIGNMENT.md) |
| Folder structure target | [FOLDER_STRUCTURE.md](../FOLDER_STRUCTURE.md) |

**Exit criteria:** Review checklist in [PLAN.md](../PLAN.md) signed off.

---

## Phase 1 — Catalog and contracts

**Goal:** Define QA targets as data; no workflow changes yet.

| Task | Owner module |
|------|--------------|
| Add `catalog/qa/` directory + README | am-agents/catalog |
| Define `selectors.schema.json` (mirror ADR-004) | am-agents/catalog/qa |
| Sample entries: backend, frontend, network | am-agents/catalog/qa |
| Extend `CatalogReader.list_qa()` | support-agent/intelligence |
| Draft ADR-006 QA catalog | docs/agent-platform/decisions |
| Contract tests for catalog parse | support-agent/tests |

**Exit criteria:** Catalog loads in support-agent; selector resolves to sample TargetSet in unit test.

---

## Phase 2 — QA workflow + PR trigger

**Goal:** End-to-end QA run from support-agent; optional PR hook.

| Task | Owner module |
|------|--------------|
| Add `qa_run` workflow + activities | support-agent/orchestrator |
| Route QA targets → tool-agent / ui-test-agent | support-agent/router |
| RunStore `kind=qa` rows | platform-ports + postgres adapter |
| Reusable workflow in am-pipelines | am-pipelines |
| PR labels `qa:pass` / `qa:fail` / `qa:partial` | am-pipelines |
| `/qa` comment commands | am-pipelines or support-agent gateway |

**Exit criteria:** Manual QA demand completes; PR trigger runs smoke catalog on pilot repo.

---

## Phase 3 — System + verify integration

**Goal:** Multi-service journeys and observability checks.

| Task | Owner module |
|------|--------------|
| `catalog/qa/system/` journey entries | catalog |
| Wire `catalog/verify/` into QA verify step | support-agent + tool-agent observe |
| Network probe entries in `catalog/qa/network/` | catalog |
| Partial-failure reporting in PR comment | support-agent |
| Enable `SUPPORT_AGENT_QA_PARITY` in staging | ops |

**Exit criteria:** System catalog run produces partial-safe summary; verify checks attach to RunStore steps.

---

## Phase 4 — Intelligence and ops

**Goal:** Reduce noise; improve signal over time.

| Task | Owner module |
|------|--------------|
| Flaky test detection from RunStore history | support-agent/intelligence |
| Risk-based selection (small diff → targeted QA) | support-agent/planner |
| Scheduled full regression | am-pipelines cron |
| Grafana dashboard from RunStore metrics | am-obs-platform |
| Gated learning from QA feedback | support-agent/learning |

**Exit criteria:** False positive rate measured; dashboard live for pilot repos.

---

## Dependency graph

```text
Phase 0 (review)
    │
    ▼
Phase 1 (catalog)
    │
    ▼
Phase 2 (workflow + PR)
    │
    ├──────────────┐
    ▼              ▼
Phase 3        (optional parallel)
(system+verify)
    │
    ▼
Phase 4 (intelligence)
```

---

## Out of scope for all phases

- New orchestrator package outside support-agent
- fin-agent changes
- Production load tests on every PR
- Auto-merge without human policy
