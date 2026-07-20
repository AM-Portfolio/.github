# QA Agent — Keep Plan (Review Draft)

Planning package for full-stack **QA** testing and extraction of **`spt-agent`** inside the AM-Portfolio agent ecosystem.

**Implementation home:** `AM-Portfolio/am-agents`  
**This folder:** review-only keep plan — no code, no workflows.

---

## Start here

| Document | Read for |
|----------|----------|
| [PLAN.md](./PLAN.md) | Master strategy |
| [FOLDER_STRUCTURE.md](./FOLDER_STRUCTURE.md) | Target layout incl. new `spt-agent/` |
| [ALIGNMENT.md](./ALIGNMENT.md) | Mapping to existing agents |
| [agents/spt-agent.md](./agents/spt-agent.md) | SPT extract — execute only, no orchestration |

---

## Agent model

```
Trigger (PR / QA or SPT demand / /qa)
           │
           ▼
    support-agent          ← ALL orchestration (plan, route, verify, report)
     ┌─────┼──────┬────────┐
     │     │      │        │
tool-agent  ui-test  spt-agent  db-agent
(backend,   (E2E)   (load/perf) (optional)
 network,
 observe)
           │
           ▼
    RunStore + verdict
```

**Decision:** Extract **`spt-agent`** as a new specialist module.  
**Orchestration stays out** of spt-agent — only in **support-agent**.

**Note:** `fin-agent` is finance product — not part of QA/SPT.

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
│   ├── spt-agent.md          ★ extract plan
│   ├── tool-agent.md
│   └── ui-test-agent.md
├── catalog/README.md
├── contracts/README.md
├── registry/agents.yaml.example
└── testing-domains/overview.md
```

---

*Status: Draft for review · Owner: AM-Portfolio Infrastructure Team · Updated: 2026-07-20*
