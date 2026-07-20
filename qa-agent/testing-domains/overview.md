# Testing domains — coverage map

How each testing type maps to **am-agents** specialists and **catalog** paths.

---

## Summary matrix

| Type | Domain | Specialist | Catalog | Phase |
|------|--------|------------|---------|-------|
| Unit / integration | Backend | tool-agent | `catalog/qa/backend/` | 2 |
| API smoke / contract | Backend | tool-agent | `catalog/qa/backend/` | 2 |
| Component / unit UI | Frontend | repo CI (optional) | — | 2 |
| E2E UI | Frontend | ui-test-agent | `catalog/qa/frontend/` | 2 |
| Visual baseline | Frontend | ui-test-agent | `catalog/qa/frontend/` | 2 |
| Accessibility smoke | Frontend | ui-test-agent | `catalog/qa/frontend/` | 3 |
| Multi-service journey | System | support-agent fan-out | `catalog/qa/system/` | 3 |
| Service health matrix | System | tool-agent + verify | `catalog/verify/` | 3 |
| Queue / webhook flow | System | support-agent + tool-agent | `catalog/qa/system/` | 3 |
| DNS / TLS / latency | Network | tool-agent | `catalog/qa/network/` | 3 |
| Load / performance | SPT | **spt-agent** | `catalog/spt/` | extract |
| Metrics / logs gate | Verify | tool-agent (observe) | `catalog/verify/` | 3 |
| Data sanity | Backend | db-agent (optional) | demand-only | 4 |
| Dependency audit | Security | repo CI / tool-agent | TBD | 4 |

---

## Priority rules (support-agent planner)

| Priority | When | On failure |
|----------|------|------------|
| **P0** | Auth, payments, PII, health smoke | Blocking |
| **P1** | Changed domain in PR diff | Warning or blocking (policy) |
| **P2** | Full regression, load, deep security | Release / scheduled only |

Tag convention in catalog: `priority:P0`, or explicit `priority` field.

---

## Domain → specialist routing

```text
backend   ──► tool-agent
network   ──► tool-agent
verify    ──► tool-agent     (tools/observe/)
frontend  ──► ui-test-agent
spt/perf  ──► spt-agent      ★ extract (no orchestration)
system    ──► support-agent  (fan-out → specialists)
```

---

## Environment matrix

| Environment | Backend | Frontend E2E | Network | Load/perf |
|-------------|---------|--------------|---------|----------|
| PR preview | yes | yes | allowlist | no |
| preprod | yes | yes | yes | limited |
| staging | yes | yes | yes | yes |
| production | read-only smoke | synthetic only | read-only | no |

---

## Partial failure (ADR-004 — applies to QA)

| `failure_mode` | Behavior |
|----------------|----------|
| `continue` (default) | Other targets keep running |
| `fail_fast` | Cancel pending after first hard fail |

`skipped` (policy deny / disabled target) is not a hard fail.

Notify and PR comment must show counts — never false all-green on partial.

---

## Example: one PR, multiple domains

**Change:** payment API + checkout UI + CDN hostname.

| Step | Agent | Action |
|------|-------|--------|
| 1 | support-agent | Selector `{ tags: [payments, checkout, network] }` |
| 2 | tool-agent / **spt-agent** | API smoke + TLS; load via spt-agent |
| 3 | ui-test-agent | checkout E2E spec |
| 4 | support-agent | Verify + `QaRunSummary` |
| 5 | am-pipelines | PR comment + label |

---

## Out of scope

| Item | Reason |
|------|--------|
| fin-agent portfolio tests | Separate product (`am-fin-agent`) |
| Unbounded load on every PR | ADR-004 runaway guards (enforced in support-agent) |
| Orchestration inside spt-agent | Extract lock — execute only |
| Hardcoded service names in code | Use catalog only |

---

## References

- [PLAN.md](../PLAN.md)
- [catalog/README.md](../catalog/README.md)
- [agents/support-agent.md](../agents/support-agent.md)
