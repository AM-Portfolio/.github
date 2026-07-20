# SPT / QA Agent

Organization-wide plan for an **SPT (Software Product Testing) / QA agent** that runs backend, frontend, system, and network testing through specialized sub-agents.

## Contents

| Document | Purpose |
|----------|---------|
| [PLAN.md](./PLAN.md) | Master keep plan: goals, architecture, phases, and operating model |
| [agents/tool-agent.md](./agents/tool-agent.md) | Tool Agent: executes tests and collects evidence |
| [agents/fin-agent.md](./agents/fin-agent.md) | Fin Agent: finds scope and produces final QA verdict |
| [testing-domains/overview.md](./testing-domains/overview.md) | Coverage map across all testing types |

## Quick summary

```
User / PR / CI trigger
        │
        ▼
   SPT / QA Orchestrator
    ┌───┴───┐
    │       │
Fin Agent  Tool Agent
(find)     (execute)
    │       │
    └───┬───┘
        ▼
   Unified QA report
```

Start with [PLAN.md](./PLAN.md) for the full strategy.
