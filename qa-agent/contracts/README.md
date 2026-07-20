# Data contracts — QA plan

Reuse **`am_platform_ports`** schemas where they exist. Do not fork DTOs in QA-specific packages (ADR-003).

---

## Existing schemas (platform-ports)

| Schema | Use in QA |
|--------|-----------|
| `SptDemandRequest` | Extend or mirror for QA demand |
| `QaRunSummary` | QA summary |
| `ChildRunResult` | Per-target specialist result |
| RunStore `create_run(kind=...)` | `kind=qa` |

Location: `am-agents/libs/platform-ports/src/am_platform_ports/schemas/`

---

## Planned: QA demand

Option A — extend existing demand shape:

```json
{
  "kind": "qa",
  "selector": { "tags": ["backend", "smoke"] },
  "environment": "preview",
  "failure_mode": "continue",
  "demand_ref": "qa-pr-123-abc",
  "context": {
    "pr_number": 123,
    "repo": "AM-Portfolio/example-api",
    "sha": "abc123"
  }
}
```

Option B — separate `QaDemandRequest` with same selector shape.

**Open decision:** see [PLAN.md](../PLAN.md) §13.

---

## TargetSet (resolver output)

```json
{
  "targets": [
    {
      "id": "api-health-preprod",
      "kind": "backend",
      "specialist": "tool-agent",
      "capability": "tools.execute",
      "params": { }
    }
  ],
  "expanded_count": 1,
  "skipped": []
}
```

---

## ChildRunResult (specialist → support-agent)

```json
{
  "target_id": "api-health-preprod",
  "status": "succeeded",
  "duration_ms": 340,
  "retryable": false,
  "evidence_refs": ["minio://runs/qa-abc/step-1.json"],
  "error_summary": null
}
```

Statuses: `succeeded` | `failed` | `skipped` | `cancelled`

---

## QaRunSummary (final verdict)

```json
{
  "demand_ref": "qa-pr-123-abc",
  "overall_status": "partial",
  "failure_mode": "continue",
  "counts": {
    "succeeded": 4,
    "failed": 1,
    "skipped": 0,
    "cancelled": 0
  },
  "domains": {
    "backend": "pass",
    "frontend": "fail",
    "system": "pass",
    "network": "pass"
  },
  "blocking_findings": [
    {
      "target_id": "checkout-e2e",
      "severity": "blocking",
      "summary": "Timeout on #payment-success",
      "reproduction": "ui.test.run spec_ref=e2e/checkout.spec.ts"
    }
  ]
}
```

`overall_status` rules (same as ADR-004 SPT):

| Condition | Status |
|-----------|--------|
| failed=0, succeeded≥1 | `succeeded` |
| failed≥1 and succeeded≥1 | `partial` |
| succeeded=0, failed≥1 | `failed` |
| empty TargetSet | fatal before run |

---

## RunStore mapping

| Field | Value |
|-------|-------|
| `kind` | `qa` |
| `demand_ref` | PR or manual ref |
| `overall_status` | from summary |
| steps | one row per target + verify step |

Secrets never stored in RunStore rows (ADR-002, ADR-005).

---

## PR comment format (planned)

Markdown summary posted by am-pipelines or support-agent notify adapter:

```markdown
## QA Agent — partial

| Domain | Result |
|--------|--------|
| backend | pass |
| frontend | fail |
| network | pass |

**Blocking:** checkout-e2e — Timeout on #payment-success

Run: `qa-pr-123-abc` · [RunStore link]
```

---

## Versioning

- Contract version in demand: `"schema_version": 1`
- Breaking changes → new ADR + bump version
