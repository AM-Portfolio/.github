# Tool Agent Specification

The **Tool Agent** executes tests. It receives a structured plan from the Fin Agent and runs commands, browsers, and network probes — collecting logs, screenshots, and metrics as evidence.

---

## Responsibilities

1. Parse `test-plan.json` and validate commands against an allowlist
2. Execute cases in parallel where safe (unit tests) or sequentially (E2E)
3. Capture stdout/stderr, HTTP responses, HAR, screenshots
4. Enforce timeouts and resource limits
5. Return `results.json` per case and upload artifacts

---

## Domain playbooks

### Backend

```bash
# Examples — actual commands come from repo conventions
pytest tests/ -q --tb=short
npm test -- --runInBand
go test ./... -count=1
curl -sf "$PREVIEW_URL/health"
newman run postman/collection.json -e preview.json
```

**Validates:** status codes, JSON schema, auth headers, DB migrations (dry-run)

### Frontend

```bash
npm run test:unit
npx playwright test e2e/ --reporter=json
# Optional: axe accessibility scan in Playwright afterEach
```

**Validates:** routing, forms, critical UI flows, console errors, basic a11y

### System

```bash
# Health matrix — all services must respond
for url in $SERVICE_URLS; do curl -sf "$url/health"; done

# Synthetic script (repo-specific)
node scripts/qa/synthetic-checkout.js
```

**Validates:** cross-service flows, feature flags, queue/worker connectivity

### Network

```bash
dig +short "$HOST"
openssl s_client -connect "$HOST:443" -servername "$HOST" </dev/null
curl -w "%{time_connect} %{time_total}\n" -o /dev/null -s "$URL"
traceroute -m 15 "$HOST"   # staging only, rate-limited
```

**Validates:** DNS, TLS expiry, latency SLOs, reachability from Cloud Agent egress

---

## Tool allowlist (security)

Only approved command prefixes may run:

| Allowed | Blocked |
|---------|---------|
| `pytest`, `npm test`, `go test`, `playwright` | Arbitrary `rm`, `curl \| bash` |
| `curl`, `dig`, `openssl` with fixed hosts | Scanning undeclared IP ranges |
| `newman`, `k6` in staging | k6 against production |
| Repo `scripts/qa/*` | Ad-hoc scripts outside allowlist |

Hosts must appear in `test-plan.json` `allowed_hosts[]`.

---

## Execution environment

| Context | Where Tool Agent runs |
|---------|------------------------|
| PR CI | GitHub Actions job with repo checkout |
| Deep QA | Cursor Cloud Agent with tmux sessions |
| Release | Staging cluster + dedicated QA runner |

Cloud Agent notes:

- Use tmux for long-running E2E and servers
- Commit artifacts to run bundle path before finalize
- Respect egress policy from environment config

---

## Output format

Per case:

```json
{
  "case_id": "e2e-checkout",
  "domain": "frontend",
  "status": "failed",
  "started_at": "2026-07-20T10:00:00Z",
  "duration_ms": 45000,
  "exit_code": 1,
  "evidence": [
    "artifacts/e2e-checkout/trace.zip",
    "artifacts/e2e-checkout/screenshot.png"
  ],
  "error_summary": "Timeout waiting for #payment-success"
}
```

---

## Parallelism rules

| Safe parallel | Sequential only |
|---------------|-----------------|
| Unit test shards | E2E sharing one browser profile |
| Independent API probes | DB migration tests |
| Lint + unit in different dirs | Load tests |

Max concurrent jobs: **4** per run (configurable org-wide).

---

## Prompt principles (for Cloud / LLM agent)

```
You are the Tool Agent for AM Portfolio QA.

- Execute only cases listed in test-plan.json.
- Do not deviate from allowlisted commands.
- On failure: capture maximum evidence before exiting.
- On flaky failure: retry once for E2E/network only.
- Never print secrets; redact Authorization headers in logs.
- Report inconclusive if environment is unreachable.
```

---

## Integration

- Triggered by Orchestrator after Fin Agent publishes `test-plan.json`
- Results consumed by Fin Agent Finalize phase
- Failed P0 cases block PR when branch protection + `qa:required` enabled

See [../PLAN.md](../PLAN.md) for orchestration flow.
