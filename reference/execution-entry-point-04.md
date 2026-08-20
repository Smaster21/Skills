<!-- PRESERVED VERBATIM (part 4 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
# 4A. Execution Flow — End To End

The same 21 states and 16 phases, read as one causal chain. **This section adds no
state and no phase** — it is a reading aid for the model above and §5 below.

```
Request + authorized target
        ↓                       ← authorization verified first (§9); missing → STOP
Scope + governance validation    01 §30.1 scope identity · 01 §29 configuration
        ↓
Discovery                        03 · Phase 1
        ↓
Application / knowledge model    04 · Phase 2
        ↓
Locator verification             03 §29 → 06 §26 · Phase 3
        ↓
Risk + test planning             05 · Phase 4   (risk consumed from 02)
        ↓
Test generation                  06 · Phase 5
        ↓
Suite validation                 01 §18 · 06 §38 · Phase 6
        ↓
Execution orchestration          07 — sole runtime authority · Phase 7
        ↓
Browser / tool execution          07 §26, §27
        ↓
Evidence collection              02 §18 · Phase 9
        ↓
Per-item evidence confidence     PLAYBOOK §19
        ↓
Evidence Quality                 PLAYBOOK §19 → 02 §21's 0.35 term
        ↓
Decision evaluation              02 §15 pipeline
        ↓
Framework Confidence             02 §21 — sole authority
        ↓
Diagnostics + reporting          01 §23 · 09 · Phase 10
        ↓
Learning                         10 · Phase 11
        ↓
Next execution's decisions       10 §7.5 pinned snapshot
```

Evidence flows **forward** into decisions and **across executions** into learning.
An execution never reads history it is concurrently writing (`10` §7.5), and
same-execution dependencies flow downward only (`01` §41).

---

---

<!-- nav -->
*← [part 3](execution-entry-point-03.md)  ·  part 4 of 7  ·  [part 5](execution-entry-point-05.md) →*  ·  [reference index](INDEX.md)
