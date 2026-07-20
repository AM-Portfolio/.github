# Testing Domains Overview

Complete map of testing types the SPT / QA agent covers, with Fin Agent (scope) vs Tool Agent (execution) split.

---

## Matrix

| Type | Domain | Fin Agent finds | Tool Agent runs |
|------|--------|-----------------|-----------------|
| Unit tests | Backend / Frontend | Changed modules → test files | `pytest`, `jest`, `vitest`, etc. |
| Integration tests | Backend | API + DB touchpoints | Test suite + testcontainers |
| Contract tests | Backend | OpenAPI diff | Schema validator, Pact |
| API smoke | Backend | Public routes in diff | `curl` / HTTP client |
| Component tests | Frontend | Changed `.tsx` / `.vue` | Unit runner |
| E2E UI | Frontend | User journeys affected | Playwright / Cypress |
| Accessibility | Frontend | Form / nav changes | axe in Playwright |
| Synthetic monitoring | System | Critical paths | Scripted multi-step flows |
| Service health | System | `docker-compose`, k8s | Health endpoint matrix |
| Event / queue | System | Producers/consumers in diff | Publish + consume test msg |
| DNS | Network | Hostnames in infra diff | `dig`, `nslookup` |
| TLS | Network | HTTPS endpoints | `openssl s_client`, cert expiry |
| Latency | Network | SLO-defined URLs | `curl -w`, repeated samples |
| Connectivity | Network | Cross-region / VPC | Probe from Cloud Agent |
| Dependency audit | Security (P2) | Lockfile changes | `npm audit`, `pip-audit` |
| Load | Performance (P2) | Release only | k6 scenarios on staging |

---

## Priority rules (Fin Agent)

```
P0 — Must run on every relevant PR; failure blocks merge
  - Auth, payments, data deletion
  - Health of production-critical paths (in preview/staging)
  - Breaking API contract changes

P1 — Run when domain touched; failure warns
  - Non-critical UI
  - Secondary API endpoints
  - Network latency regression

P2 — Scheduled or release-only
  - Full regression E2E suite
  - Load tests
  - Deep security scan
```

---

## Example: single PR, multiple domains

**Change:** Update checkout API + payment form + CDN hostname in terraform.

| Step | Agent | Action |
|------|-------|--------|
| 1 | Fin | Detect API + UI + infra; assign P0 backend, frontend, network |
| 2 | Tool | `pytest tests/payments`; Playwright checkout spec |
| 3 | Tool | `dig cdn.example.com`; TLS check |
| 4 | Fin | Fail if E2E or TLS fails; pass with warning if latency +20% |
| 5 | Orchestrator | Post combined report; label `qa:fail` or `qa:pass` |

---

## Environment targets

| Environment | Backend | Frontend E2E | Network probes | Load |
|-------------|---------|--------------|----------------|------|
| Local / CI | ✅ | Mock or skip | Skip external | ❌ |
| PR preview | ✅ | ✅ | Limited allowlist | ❌ |
| Staging | ✅ | ✅ | Full declared hosts | ✅ |
| Production | Read-only smoke | Synthetic only | Read-only | ❌ |

---

## Glossary

- **SPT** — Software Product Testing; org name for unified QA agent
- **Smoke** — Minimal fast checks that services respond
- **Synthetic** — Scripted user journey across services
- **Verdict** — Final pass/fail from Fin Agent after Tool Agent completes

Parent plan: [../PLAN.md](../PLAN.md)
