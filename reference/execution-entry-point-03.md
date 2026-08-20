<!-- PRESERVED VERBATIM (part 3 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
## What It SHALL NEVER Do

It SHALL NEVER own decisions, planning, execution, healing, reporting, or
learning.

It SHALL NEVER calculate **Framework Confidence** or **Framework Risk**.

It SHALL NEVER bypass the Decision Engine.

Every value it reasons with is **consumed from its owner**:

| Value | Consumed from | Contract |
|---|---|---|
| Framework Risk | **02** | §20 Risk Engine |
| Framework Confidence | **02** | §21 Confidence Engine |
| Canonical coverage | **05** | §18 Coverage Analysis |
| Coverage gaps, test value | **14** | Coverage Recommendations |
| Execution cost, runtime prediction | **12** | §26 Runtime Predictor |
| Application model | **04** | Graph query contract |
| Historical evidence | **10** / **09** | Learning and analytics contracts |
| Duplicate detection | **06** | §32 |

Where a required value is unavailable, the layer SHALL state the gap. It SHALL
NOT substitute an estimate of its own for an owner's value.

---

## Output — The Reasoning Package

Every recommendation SHALL contain exactly these five fields:

| Field | Content |
|---|---|
| **Reasoning** | Why this request, in terms of the objective it serves |
| **Supporting evidence** | Concrete observations, cited by source and owner |
| **Uncertainty** | What remains unknown, and what would resolve it |
| **Expected confidence improvement** | Estimated, as evidence for `02` — never a Framework Confidence value |
| **Expected execution cost** | Consumed from `12` where available; otherwise stated as unavailable |

A package with an empty evidence field SHALL NOT be submitted.

### Not A Canonical Artifact

The package is **transient**. It is submitted to the receiving owner and absorbed
into that owner's existing evidence and audit record — `02` §36 Evidence Model
and §39 Decision Audit for decisions, `09` for reporting conclusions.

It is **not** persisted as a dataset of its own, appears in no Canonical Data
Ownership row, and creates no new artifact. The Matrix is unchanged.

### Estimates Are Evidence, Not Values

The two estimated fields follow the engine-local score precedent already
established for `08`, `14`, and `15`:

- They are **evidence submitted to an owner**
- They SHALL NEVER be reported as Framework Confidence or Framework Risk
- They SHALL NEVER override an owner's value
- Where an estimate and an owner's value disagree, **the owner's value prevails**

---

## Visibility

The layer is **invisible by default**. It produces no user-facing output.

Its reasoning becomes visible only when verbose or debug output is enabled,
resolved through the configuration hierarchy owned by the Configuration Layer
(`01` §29). This orchestrator defines no configuration option of its own.

When visible, reasoning SHALL be shown as the package's five fields — never as
narration, and never as a substitute for the owner's recorded decision.

---

## Determinism

Identical inputs SHALL produce an identical package (`01` §3.1).

The layer SHALL NOT introduce randomness, wall-clock dependence, or
completion-order dependence. Where it reasons during execution, it does so at the
same deterministic checkpoints `14` uses (§5, Phase 7) — never per test
completion.

## Failure Mode

Where evidence is insufficient to reason, the layer SHALL **refuse to
recommend** — consistent with `02` §5.5 Safe Failure.

It SHALL then route the underlying request with the evidence gap stated, so the
owner decides on a known-incomplete picture rather than a fabricated one.

Refusing to recommend SHALL NEVER block a request from reaching its owner.

---

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

---

<!-- nav -->
*← [part 2](execution-entry-point-02.md)  ·  part 3 of 7  ·  [part 4](execution-entry-point-04.md) →*  ·  [reference index](INDEX.md)
