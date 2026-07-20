# Fin Agent Specification

**Fin Agent** = **Find** + **Finalize**

The Fin Agent is the intelligence layer of the SPT / QA system. It does not run heavy tests itself; it decides *what* to test and interprets *what happened*.

---

## Responsibilities

### Phase A — Find (discovery)

1. **Ingest context**
   - Git diff, changed paths, commit messages
   - OpenAPI / GraphQL / route manifests
   - `package.json`, `Dockerfile`, k8s manifests, env examples

2. **Build impact map**
   - Code change → affected API routes
   - Code change → affected UI routes / components
   - Infra change → affected hosts, ports, DNS records

3. **Produce test matrix**
   - Select domains: backend, frontend, system, network
   - Assign priority: P0 (blocking), P1 (important), P2 (nice-to-have)
   - Emit machine-readable `test-plan.json` for Tool Agent

4. **Risk assessment**
   - Auth / payment / PII paths → always P0
   - Docs-only changes → skip or lint-only
   - Unknown blast radius → conservative full smoke

### Phase B — Finalize (reporting)

1. **Collect** Tool Agent result bundles
2. **Correlate** failures with code changes (likely root cause)
3. **Dedupe** repeated failures across domains
4. **Classify** findings: blocking / warning / info
5. **Emit** `qa-verdict.json` and human-readable summary

---

## Inputs

| Input | Source |
|-------|--------|
| `changed_files[]` | Git / PR API |
| `deployment_urls[]` | PR comments, env config |
| `repo_test_inventory` | Fin discovery (existing test folders) |
| `tool_results[]` | Tool Agent after execution |

---

## Outputs

| Output | Consumer |
|--------|----------|
| `test-plan.json` | Tool Agent, Orchestrator |
| `qa-verdict.json` | Orchestrator, CI status |
| Markdown summary | PR comment, Slack (optional) |

---

## Prompt principles (for Cloud / LLM agent)

```
You are the Fin Agent for AM Portfolio QA.

FIND mode:
- Analyze the diff and repo structure.
- List every user-visible or API-visible surface that could break.
- Output a minimal but sufficient test plan; prefer targeted over exhaustive.
- Never invent URLs or credentials; mark unknowns explicitly.

FINALIZE mode:
- Read all Tool Agent results.
- One clear verdict: pass | fail | inconclusive | partial.
- Every blocking finding must include reproduction steps.
- Separate flaky signals from hard failures when evidence supports it.
```

---

## Example Find output

For a PR that changes `src/api/payments.ts` and `web/checkout/PaymentForm.tsx`:

| Domain | P0 cases | P1 cases |
|--------|----------|----------|
| Backend | payment API contract, webhook handler unit tests | refund edge cases |
| Frontend | checkout payment form E2E | visual snapshot (optional) |
| System | checkout → payment → webhook journey | — |
| Network | TLS + latency to payment gateway host | — |

---

## Failure modes

| Situation | Fin Agent behavior |
|-----------|-------------------|
| No tests exist for changed area | Flag gap; recommend minimal smoke |
| Preview URL missing | Verdict `inconclusive` for frontend/system |
| Conflicting Tool results | Prefer repeatable failures; note flake |
| Timeout mid-run | Report partial; list unfinished cases |

---

## Integration

- Invoked by **SPT / QA Orchestrator** before and after Tool Agent runs
- Can run as Cursor subagent type `explore` for discovery, then synthesis pass for finalize
- Read-only during Find; no shell execution except search/read tools

See [../PLAN.md](../PLAN.md) for full architecture.
