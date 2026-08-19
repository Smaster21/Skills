<!-- PRESERVED VERBATIM (part 3 of 6). Split ONLY to satisfy the 200-line
     reference cap; no sentence was altered, reordered, or removed. The original
     preservation note is retained at the top of part 1. -->
# 4. Execution State Machine

21 states, per `01` §17. Maintain and log explicit state. Every transition,
including every skip, SHALL be logged with its reason.

```
INITIALIZING
     ↓
CONFIGURING
     ↓
DETECTING_CHANGES °········┐
     ↓                     │
DISCOVERING ◄──────────────┘
     ↓
BUILDING_GRAPH
     ↓
INTELLIGENCE °·············┐
     ↓                     │
PLANNING ◄─────────────────┘
     ↓
PLANNING_REVIEW °··········┐
     ↓  ╰─ revise ─► PLANNING   (02-approved only, capped by 15 §28A)
GENERATING ◄───────────────┘
     ↓
VALIDATING
     ↓
OPTIMIZATION °·············┐
     ↓                     │
EXECUTING ◄────────────────┘
     ↓  ├─► RETRYING ──────┐   (02 approves)
     ↓  └─► SELF_HEALING ──┤   (02 approves)
     ↓ ◄───────────────────┘
COLLECTING_ARTIFACTS
     ↓
DIAGNOSTICS
     ↓
REPORTING
     ↓
LEARNING
     ↓
COMPLETED

°  skippable — transitions directly to its successor when its engine is
   absent or disabled.
```

Terminal failure states: **`FAILED`**, **`CANCELLED`**.

## Skip Rules

| State | Owner | Skips to |
|---|---|---|
| `DETECTING_CHANGES` | 13 | `DISCOVERING` |
| `INTELLIGENCE` | 11 | `PLANNING` |
| `PLANNING_REVIEW` | 15 | `GENERATING` |
| `OPTIMIZATION` | 12 | `EXECUTING` |

A skipped state SHALL be logged with its reason and reported by `09`.
No other state is skippable.

`14` is **not** a state — it evaluates at deterministic checkpoints *within*
`EXECUTING`.

## Rollback Rules

- Forward-only. A module may **never** transition backwards.
- **One exception:** `PLANNING_REVIEW → PLANNING`, permitted only on a
  **02-approved** revision decision, bounded by `15`'s iteration cap. On cap
  exhaustion, `15` escalates to `02`; the plan proceeds with findings disclosed
  by `09`, never silently.
- Rollback affects framework state only. It SHALL NEVER undo browser actions
  already executed (`02` §32).

## Backward Compatibility

Skipping all four `°` states traverses exactly the v3.0 sequence, in v3.0 order.

---

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
