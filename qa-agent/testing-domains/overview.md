# Testing domains — coverage map

How each testing type maps after **qa-agent extract**.

---

## Summary matrix

| Type | Domain | Specialist | Catalog |
|------|--------|------------|---------|
| Unit / integration / API smoke | Backend | **qa-agent** | `catalog/qa/backend/` |
| E2E UI | Frontend | ui-test-agent and/or **qa-agent** | `catalog/qa/frontend/` |
| Multi-service journey | System | **qa-agent** | `catalog/qa/system/` |
| DNS / TLS / latency | Network | **qa-agent** | `catalog/qa/network/` |
| Metrics / logs gate | Verify | tool-agent observe | `catalog/verify/` |
| Load / performance | SPT | existing SPT path | `catalog/spt/` |

---

## Domain → specialist routing

```text
backend   ──► qa-agent
system    ──► qa-agent
network   ──► qa-agent
frontend  ──► ui-test-agent and/or qa-agent  (open decision)
verify    ──► tool-agent
spt/perf  ──► existing SPT path (not this extract)
system fan-out orchestration ──► support-agent only
```

---

## Priority rules (support-agent planner)

| Priority | When | On failure |
|----------|------|------------|
| **P0** | Auth, payments, PII, health smoke | Blocking |
| **P1** | Changed domain in PR | Policy |
| **P2** | Full regression / deep scans | Release / scheduled |

---

## Environment matrix

| Environment | Backend | Frontend E2E | Network | SPT load |
|-------------|---------|--------------|---------|----------|
| PR preview | yes | yes | allowlist | no |
| preprod | yes | yes | yes | limited |
| staging | yes | yes | yes | yes |
| production | read-only smoke | synthetic only | read-only | no |

---

## Partial failure (ADR-004 — enforced in support-agent)

| `failure_mode` | Behavior |
|----------------|----------|
| `continue` (default) | Other targets keep running |
| `fail_fast` | Cancel pending after first hard fail |

---

## Out of scope

| Item | Reason |
|------|--------|
| Orchestration inside qa-agent | Extract lock — execute only |
| fin-agent portfolio tests | Separate product |
| Unbounded load on every PR | ADR-004 guards in support-agent |

---

## References

- [PLAN.md](../PLAN.md)
- [agents/qa-agent.md](../agents/qa-agent.md)
