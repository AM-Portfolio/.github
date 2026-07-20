# QA Agent — Keep Plan (Review Draft)

Planning package for extracting **`qa-agent`** as a specialist module in AM-Portfolio.

**Implementation home:** `AM-Portfolio/am-agents`  
**This folder:** review-only keep plan — no code, no workflows.

---

## Start here

| Document | Read for |
|----------|----------|
| [PLAN.md](./PLAN.md) | Master strategy |
| [agents/qa-agent.md](./agents/qa-agent.md) | ★ Extract — execute only, no orchestration |
| [FOLDER_STRUCTURE.md](./FOLDER_STRUCTURE.md) | Target layout in am-agents |
| [ALIGNMENT.md](./ALIGNMENT.md) | Mapping to existing agents |

---

## Agent model

```
Trigger (PR / QA demand / /qa)
           │
           ▼
    support-agent          ← ALL orchestration (plan, route, verify, report)
     ┌─────┼──────┬────────┐
     │     │      │        │
 qa-agent  ui-test  tool-agent  db-agent
 (QA exec) (E2E*)  (observe /   (optional)
                    tools / SPT)
           │
           ▼
    RunStore + verdict
```

**Decision:** Extract **`qa-agent`** as a new specialist module.  
**Orchestration stays out** of qa-agent — only in **support-agent**.

\* Frontend may route to ui-test-agent and/or qa-agent (open decision).

**Note:** `fin-agent` is finance product — not part of QA.

---

## Folder index

```
qa-agent/                         # this keep-plan folder (.github)
├── PLAN.md
├── FOLDER_STRUCTURE.md
├── ALIGNMENT.md
├── phases/PHASES.md
├── agents/
│   ├── qa-agent.md               ★ extract spec
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
