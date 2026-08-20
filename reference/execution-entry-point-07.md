<!-- PRESERVED VERBATIM (part 7 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
# 7. Invariants

| Invariant | Source |
|---|---|
| **Deterministic** — identical inputs produce equivalent outputs | `01` §3.1 |
| **Observable** — every decision explainable; no silent decisions | `01` §3.2 |
| **Modular** — one responsibility per subsystem; no cross-module mutation | `01` §3.3 |
| **Safe by default** — destructive operations require explicit opt-in | `01` §3.5 |
| **Evidence before action** — insufficient evidence means refuse, not guess | `02` §5.2, §5.5 |
| **Immutable contracts** — versioned, with schema, timestamp, correlation ID, producer, consumer, validation status | `01` §14 |
| **Immutable snapshots** — never overwrite a discovery, graph, plan, report, or learning version | `03` §31, `04` §17, `05` §33, `09` §29, `10` §28 |
| **Traceable** — Execution ID, Correlation ID, framework/config/graph version on everything | `01` §30 |
| **Masked** — secrets never reach logs, reports, screenshots, traces, or AI prompts | `01` §30, `07` §41 |
| **Honest AI** — never invent facts, fabricate results, mark failures as passes, or suppress defects | `01` §31 |
| **Gates are hard** — required-engine FAIL stops; optional-engine FAIL degrades | `01` §15, §26 |
| **One owner per capability** — consumers never become owners; algorithms exist once | Matrix, `01` §42 |
| **Optional means removable** — absence degrades cost or assurance, never correctness | `01` §16, §42 |
| **Advisory means advisory** — 11, 12, 14, 15, 16 recommend; `02` decides; `07` executes | Matrix, `01` §42 |
| **Derived is never canonical** — `11`'s projection never displaces `04` | Matrix, `01` §37 |
| **One Framework Confidence** — owned by `02`; engine-local scores are evidence | Matrix, `01` §37 |
| **Runtime layering** — same-execution dependencies flow downward only; cross-execution historical reads are exempt and never block | `01` §41 |
| **Artifact ownership ≠ runtime authority** — `12` owns a proposal; `07` alone schedules | Matrix, `01` §40 |
| **Reasoning precedes routing** — the QA Reasoning Layer (§3A) reasons before a request is raised; it owns nothing, decides nothing, and persists nothing | §3A |
| **Evidence is observed, never authored** — Evidence Objects come from real execution observations; each applicable one carries a measured confidence | `02` §18, `PLAYBOOK` §19 |
| **One Evidence Quality measurement** — derived from measured Evidence Objects; an input term of `02` §21, not a second model | `PLAYBOOK` §19 |
| **Unavailable is a state, not a number** — no term is ever substituted with `0`, `50`, `100`, a default, a previous value, or an inference | `02` §5.5, `01` §31 |
| **Confidence is all-or-nothing** — computed only when all four `02` §21 terms are available; otherwise refused with every absent term named | `02` §21 |
| **Specification is not runtime** — a documented requirement is never evidence that it happened | §7A |

---

# 7A. Validation Tiers — Specification Is Not Runtime

Four distinct claims. They are **not** interchangeable, and conflating them is a
reporting defect (`01` §31, `PLAYBOOK` §16).

| Tier | Question | Sufficient evidence |
|---|---|---|
| **Specification validation** | Does the architecture *say* this must happen? | A document and section reference |
| **Implementation validation** | Does the implementation *contain* the behaviour? | Source plus a passing unit/integration test |
| **Runtime validation** | Did the framework *do* it against a real authorized target? | Artifacts produced by that execution |
| **Evidence validation** | Can a persisted value be *traced and recomputed*? | The artifact path, its inputs, and a reproduction |

> **Specification PASS is not Runtime PASS.**
>
> **Implementation test PASS is not Runtime PASS.**

Report the tier you actually have:

- `PASS` — observed at the tier claimed, with the artifact path.
- `NOT DEMONSTRATED` — the framework ran, but this property was not observable.
- `BLOCKED` — execution could not legitimately proceed.
- `FAIL` — execution proceeded and a required invariant was violated.

`NOT PRODUCED` is the honest answer for an artifact that does not exist. Never
infer it from another artifact, and never create it to satisfy a check.

---

# 8. Non-Goals

Out of scope by architecture (`01` §2):

penetration testing · production exploitation · destructive automation ·
unsupported browser automation · CAPTCHA bypass · authentication bypass

Security validation is a **secondary, opt-in** goal — passive by default. Active
security scanning requires explicit authorization and a dedicated test
environment (Playbook §9). Native mobile automation is a future capability
(`01` §35), not a current one.

---

# 9. On Invocation

## Before anything else — authorization

0. **Establish the authorized target.** Use only a target the user has explicitly
   authorized. If the target or its authorization is **missing or ambiguous**,
   **STOP and report `BLOCKED`**. SHALL NEVER discover, select, invent, or
   substitute a target, and SHALL NEVER carry authorization forward from a
   previous run to a new one.

Then:

1. Read **`01`** in full. Highest architectural authority.
2. Read the **Ownership Matrix**. Never implement a capability another engine owns.
3. Read **`02`**. Nothing decides for itself.
4. Read the document owning the phase you are entering, **before** entering it.
5. Before raising any request listed in §3A, run the **QA Reasoning Layer** and
   attach its package. Silent unless verbose or debug output is enabled.
6. Consult **`16`** for operational judgement and the **Implementation Playbook**
   for tactics, once you know what the phase requires.
7. Walk the phases in order, including the optional ones enabled. Log every state
   transition and every skip with its reason.
8. At each gate: **PASS** → advance · **WARNING** → advance, record ·
   **FAIL (required)** → stop, diagnose · **FAIL (optional)** → degrade, report.
9. If a mandatory input is missing, stop at the gate and report it.

## Minimum Viable Run

Required engines only — `02`–`07` and `09`. This is exactly the v3.0 pipeline and
is always valid. Enable `08` and `10`–`15` as the target's scale justifies.

## Governance The Executing Agent Enforces

These hold in every run, and are the reason a run may legitimately stop:

- **Authorized targets only.** Missing or ambiguous authorization → `BLOCKED`.
- **Never substitute a target**, and never scan an unrelated system.
- **Preserve historical artifacts.** A previous execution's output is immutable
  evidence. Never modify, overwrite, or append to it.
- **Keep runtime artifacts inside the current execution's own boundary.**
- **Preserve ownership.** Never implement a capability another engine owns, and
  never become the owner of an artifact you consume (§3, Matrix).
- **Preserve confidence ownership** — `02` §21 only — and **Evidence Quality
  ownership** — `PLAYBOOK` §19 as `02`'s term (§5A).
- **Preserve identity ownership** — `06` §22.1, `01` §30, `01` §30.1 (§5B).
- **Introduce nothing new**: no engine, dataset, lifecycle state, phase,
  identifier authority, ownership authority, confidence model, or Evidence
  Quality model.
- **Never modify a frozen document** unless the current task explicitly
  authorizes it.
- **Never fabricate evidence** to satisfy a criterion, and never convert
  specification evidence into runtime evidence (§7A).
- Report `NOT DEMONSTRATED` when runtime evidence does not exist, `BLOCKED` when
  execution cannot legitimately proceed, and `NOT PRODUCED` for an artifact that
  was never created.

If satisfying a requirement would need any of the prohibited changes above,
**STOP and report the exact conflict.** Do not invent a resolution.

## Before Reporting

State which optional engines ran, which were skipped, and what was not verified.
Reconcile every number against the runner's own output. Separate target defects
from suite defects, and environment failures from application failures.

State the **validation tier** of every claim (§7A), and give the artifact path
for each reported value. For the evidence chain, report the Evidence Object
count, the per-item confidences, `evidenceQuality`, and either Framework
Confidence or the named unavailable terms (§5A). A capped, optimized,
early-stopped, incrementally-discovered, single-browser or credential-limited run
SHALL NEVER be presented as complete.

---

# End of Execution Entry Point

---

<!-- nav -->
*← [part 6](execution-entry-point-06.md)  ·  part 7 of 7*  ·  [reference index](INDEX.md)
