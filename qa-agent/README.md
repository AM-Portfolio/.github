# QA Agent — Keep Plan (Review Draft)

Planning package for full-stack **QA agent** testing inside the **AM-Portfolio agent ecosystem**.

**Implementation home:** `AM-Portfolio/am-agents`  
**This folder:** review-only keep plan — no code, no workflows.

---

## Start here

| Document | Read for |
|----------|----------|
| [PLAN.md](./PLAN.md) | Master strategy, agent roles, phases, safety |
| [FOLDER_STRUCTURE.md](./FOLDER_STRUCTURE.md) | Target folder layout aligned with `am-agents` |
| [ALIGNMENT.md](./ALIGNMENT.md) | Mapping to existing `support-agent`, `tool-agent`, `ui-test-agent` |

---

## Agent model

```
Trigger (PR / QA demand / /qa)
           │
           ▼
    support-agent          ← orchestrator (plan, route, verify, report)
     ┌─────┼─────┐
     │     │     │
tool-agent  ui-test-agent  db-agent (optional)
(execute)   (frontend E2E)  (data checks)
     │     │
     └─────┘
           ▼
    RunStore + QA verdict
```

**Note:** `fin-agent` (`am-fin-agent`) is the finance product agent — not part of QA.

---

## Folder index

```
qa-agent/
├── PLAN.md
├── FOLDER_STRUCTURE.md
├── ALIGNMENT.md
├── phases/PHASES.md
├── agents/
│   ├── support-agent.md
│   ├── tool-agent.md
│   └── ui-test-agent.md
├── catalog/README.md
├── contracts/README.md
├── registry/agents.yaml.example
└── testing-domains/overview.md
```

---

*Status: Draft for review · Owner: AM-Portfolio Infrastructure Team · Updated: 2026-07-20*
