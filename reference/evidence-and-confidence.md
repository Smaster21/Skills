# Evidence, Evidence Quality, and Framework Confidence

> Executor / reviewer reference. This file **routes** to the authoritative
> procedures; it deliberately copies **no formula**. The algorithms live in
> `docs/` and are owned there. Read this to know *what must happen and where the
> rule lives* — not to reimplement it.

This chain is operationally mandatory. A QA run that emits tests and reports but
not this chain has not satisfied the framework.

```
Real execution observation
        → Evidence Object            docs/02_Decision_Engine.md §18   (id · type · source · confidence · timestamp · payload)
        → Per-item confidence        docs/IMPLEMENTATION_PLAYBOOK.md §19
        → Evidence Quality           docs/IMPLEMENTATION_PLAYBOOK.md §19 → the 0.35 term of 02 §21
        → Framework Confidence       docs/02_Decision_Engine.md §21   (SOLE authority)
        → Decision / report artifacts  qa/raw/decision-history.jsonl, qa/report/
```

## Evidence Objects — observed, never authored

- Produced from **real** execution observations (a live DOM read, a probe
  result, a measured latency, a persisted artifact).
- Six fields, per `02` §18: `id · type · source · confidence · timestamp · payload`.
- Each applicable object carries a **measured** `confidence`. A hand-authored
  confidence is a defect, not evidence.
- Never fabricated, back-filled, or derived from specification text. Specification
  is not runtime evidence.

## Evidence Quality — one input term, not a second model

- Derived from the **measured** Evidence Objects of the decision it belongs to.
- Procedure (completeness · directness · corroboration · single rounding ·
  clamp · order-independent mean): **`docs/IMPLEMENTATION_PLAYBOOK.md §19`**. Do
  not recompute it a second way.
- Empty evidence fails safe: `evidenceQuality` is **undefined** and the decision
  is refused `INSUFFICIENT_EVIDENCE` (`02` §5.5) — never `0`, never defaulted.

## Framework Confidence — `02` §21 only

Single model, fixed terms and weights (stated for identification; do not
reweight, duplicate, or relocate):

| Term | Weight | Measurement procedure |
|---|---:|---|
| Evidence Quality | 0.35 | `PLAYBOOK` §19 |
| Historical Reliability | 0.25 | `PLAYBOOK` §20.1 |
| Rule Agreement | 0.20 | `PLAYBOOK` §20.2 |
| Environment Stability | 0.20 | `PLAYBOOK` §20.3 |

- Every other component **supplies observations** to `02`; none computes
  confidence. Engine-local scores (`08`, `14`, `15`) are evidence submitted to
  `02`, never confidence.
- Computed **only when all four terms are available**. If any is unavailable,
  `02` **refuses** and **names every** unavailable term. No partial score, ever.

### The three §20 term measurements — what they are, and are NOT

| Term | Measures | NOT |
|---|---|---|
| Historical Reliability (`§20.1`) | Framework **lifecycle** reliability across eligible prior executions of the current Execution Scope Identity (`01` §30.1), from the pinned snapshot (`10` §7.5) | **Not** test pass rate. A target defect never lowers framework reliability |
| Rule Agreement (`§20.2`) | How consistently applicable evaluated rules (`02` §39) agree with the selected candidate | **Not** a new rule system. `02` §19's `Confidence Modifier` is never read |
| Environment Stability (`§20.3`) | How nominally the target behaved in the **pre-execution** verification window (`07` §17/§31, `PLAYBOOK` §18) | **Not** during-execution health — that would feed the outcome back into the term that scores it |

## Unavailable measurements — named states, never numbers

| State | Term | Source |
|---|---|---|
| `NO_HISTORY` / `INSUFFICIENT_HISTORY` / `NO_HISTORY_CONFIGURATION` | Historical Reliability | `PLAYBOOK` §20.1 |
| `NO_APPLICABLE_RULES` / `RULE_OUTCOMES_NOT_RECORDED` | Rule Agreement | `PLAYBOOK` §20.2 |
| `NO_ENVIRONMENT_OBSERVATION` / `NO_LATENCY_REFERENCE` | Environment Stability | `PLAYBOOK` §20.3 |
| `INSUFFICIENT_EVIDENCE` | Evidence Quality + the decision | `PLAYBOOK` §19 · `02` §5.5 |
| `MISSING_TERM` | Framework Confidence, naming each absent term | `02` §21 · `02` §5.5 |

### An unavailable term MUST report its distance from availability

Naming the term is necessary but not sufficient. `REFUSED — NO_HISTORY` reads like a
fault; in most runs it is simply a term that has not accumulated yet. Report the
**progress toward availability** alongside the state:

```
Historical Reliability : UNAVAILABLE (INSUFFICIENT_HISTORY)
                         1 of 3 eligible executions recorded for this scope
                         (recency window W=5, minimumExecutions=3)
                         → available after 2 further executions of this scope
```

This distinguishes the two cases that a bare state name conflates:

| Case | Meaning | Action |
|---|---|---|
| Accumulating | The store exists, `W` and `minimumExecutions` are configured, history is building | None — report the count and wait |
| **Configuration gap** | No store resolved for the Execution Scope Identity, or `W` / `minimumExecutions` never configured | **Report as a configuration defect** (FM-8), not as a neutral unavailable measurement |

A term that can *never* become available is a defect. A term that is two runs away
is not. The report MUST make clear which it is.

### Eligible history includes non-`COMPLETED` terminal states

Every execution persists its **terminal lifecycle state** (`01` §17) and its scope
identity — `COMPLETED`, `FAILED`, `BLOCKED`, `INTERRUPTED` alike — written at the
point the state is reached, **not** only on a successful reporting phase.

> Persisting only at the end of reporting means an interrupted run contributes
> nothing. History then accrues solely from clean runs, so the term arrives late
> *and* measures a biased sample — reliability computed only over runs that already
> succeeded. Since Historical Reliability measures **framework lifecycle**
> reliability, an interrupted run is exactly the signal it exists to capture.

A cold-start scope with no history is a legitimate unavailable state, **not** an
error, and never halts execution (`10` §7.3).

## Persistence — no new artifact

| Value | Recorded in | Owner |
|---|---|---|
| Per-item confidence | the Evidence Object's `confidence` field | `02` §18 |
| Evidence Quality | the decision's evidence record | `02` §36 |
| Framework Confidence + each term's value/reason | the decision audit record | `02` §39 |

All three destinations already exist in `qa/raw/decision-history.jsonl`. No dataset,
file, or schema is created for this chain.

## Validation tiers — do not overclaim

`docs/` and this file are **specification**. A passing unit test is
**implementation**. Only a real run against an authorized target is **runtime**.
Report the tier you actually have: `PASS` (with artifact path) ·
`NOT DEMONSTRATED` (ran but not observable) · `BLOCKED` · `FAIL` ·
`NOT PRODUCED`. Specification PASS is not Runtime PASS; implementation test PASS
is not Runtime PASS.
