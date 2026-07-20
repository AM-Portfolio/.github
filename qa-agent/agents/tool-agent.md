# tool-agent — role after qa-agent extract

**Canonical name:** `tool-agent`  
**Path:** `am-agents/tool-agent/`  

---

## Steady state (with qa-agent)

| Domain | Owner |
|--------|-------|
| Backend / system / network QA runners | **qa-agent** |
| Observe / metrics / logs | **tool-agent** |
| Generic infra tools | **tool-agent** |
| SPT load plugin | **tool-agent** `tools/spt/` (unchanged in this plan) |

tool-agent is **not** the QA orchestrator and **not** the primary QA executor after extract.

---

## LLM

Structured calls from support-agent: no LLM on PR→QA path.

---

## References

- [qa-agent.md](./qa-agent.md)  
- [../PLAN.md](../PLAN.md)  
