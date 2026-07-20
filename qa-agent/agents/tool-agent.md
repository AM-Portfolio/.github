# tool-agent — QA executor plan (post SPT extract)

**Canonical name:** `tool-agent`  
**Path:** `am-agents/tool-agent/`  
**Role:** Backend, network, observe/verify execution.

---

## After spt-agent extract

| Domain | Owner |
|--------|-------|
| Backend API smoke / sandbox commands | **tool-agent** |
| Network DNS / TLS / latency | **tool-agent** |
| Observe / metrics / logs | **tool-agent** |
| Load / perf / k6 | **spt-agent** (not tool-agent) |

Legacy `tools/spt/` remains only until Phase 3 cutover, then **deprecated**.

---

## QA domains handled (steady state)

| Domain | Surface | Catalog |
|--------|---------|---------|
| Backend | `tools.execute` | `catalog/qa/backend/` |
| Network | `tools.execute` | `catalog/qa/network/` |
| Verify | `tools/observe/` | `catalog/verify/` |

Default port: **8141**.

---

## Cutover from `tools/spt/`

1. Dual-run: support-agent can hit tool-agent plugin **or** spt-agent  
2. Parity green in staging  
3. Remove `TOOL_AGENT_CAPABILITY_PLUGINS=spt` and delete/archive `tools/spt/`

Do **not** add new SPT features to the plugin during extract.

---

## Safety

- Command allowlist + host allowlist  
- Writes blocked by default on data adapters  
- Secrets via Vault / SecretBroker  

---

## LLM

Structured plan/execute from support-agent: **no LLM** on PR→QA path.  
Free-text intent prompts may exist for chat ops — out of scope for orchestrated QA.

---

## References

- [spt-agent.md](./spt-agent.md)  
- [../PLAN.md](../PLAN.md)  
