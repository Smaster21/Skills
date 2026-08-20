<!-- PRESERVED VERBATIM (part 6 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
## Evidence Quality

- Evidence Quality is **derived from the measured Evidence Objects** of the
  decision it belongs to.
- It is **not a second confidence model**. It is one **input term** of `02` §21's
  existing model.
- The measurement procedure — completeness, directness, corroboration, rounding,
  clamping, aggregation — belongs to **`PLAYBOOK` §19**. Do not reimplement it
  here and do not compute it a second way.
- Empty evidence SHALL fail safely per `PLAYBOOK` §19 and `02` §5.5:
  `evidenceQuality` is **undefined** and the decision is refused
  `INSUFFICIENT_EVIDENCE`. It is never zero and never defaulted.

## Framework Confidence — `02` §21 is the sole authority

`02` §21 owns the framework's **single** confidence model. Its four terms and
their existing weights are:

| `02` §21 term | Weight | Measurement procedure |
|---|---:|---|
| Evidence Quality | 0.35 | `PLAYBOOK` §19 |
| Historical Reliability | 0.25 | `PLAYBOOK` §20.1 |
| Rule Agreement | 0.20 | `PLAYBOOK` §20.2 |
| Environment Stability | 0.20 | `PLAYBOOK` §20.3 |

These are **`02` §21's existing terms and weights — not a new model.** They sum
to 1.00 and SHALL NOT be reweighted, replaced, duplicated, or relocated.

- Every other component **supplies observations** to `02` through its existing
  contract. None of them computes confidence.
- No component may publish a competing Framework Confidence value. Engine-local
  scores (`08`, `14`, `15`) are evidence submitted to `02` (§3).
- Framework Confidence is computed **only when all four terms are available**.
- Where any term is unavailable, `02` **refuses** the value and **names every**
  unavailable term. A partial confidence score SHALL NEVER be produced.

## The three `PLAYBOOK` §20 term measurements

Surface only. The procedures are owned by `PLAYBOOK` §20 through `02` §21's
contract; read them there before relying on a value.

| Term | What it measures | What it is NOT |
|---|---|---|
| **Historical Reliability** (`§20.1`) | Framework **lifecycle** reliability across eligible prior executions of the current **Execution Scope Identity** (`01` §30.1), read from the pinned snapshot (`10` §7.5) | **Not test pass rate.** A target defect SHALL NEVER reduce framework reliability — finding real defects is success, not unreliability |
| **Rule Agreement** (`§20.2`) | How consistently the **applicable evaluated rules** already recorded in `02` §39 agree with the selected candidate | **Not a new rule system.** `02` §19 remains the only Rule Engine. `02` §19's `Confidence Modifier` SHALL NEVER be read — it would be a second path into confidence |
| **Environment Stability** (`§20.3`) | How nominally the target behaved during the **pre-execution** verification window (`07` §17, §31 · `PLAYBOOK` §18) | **Not during-execution health.** Feeding `07` §40's continuous health back in would let the outcome being scored influence the term that scores it |

## Unavailable measurements

An unavailable term is a **named state**, never a number:

| State | Term | Source |
|---|---|---|
| `NO_HISTORY` | Historical Reliability | `PLAYBOOK` §20.1 |
| `INSUFFICIENT_HISTORY` | Historical Reliability | `PLAYBOOK` §20.1 |
| `NO_APPLICABLE_RULES` | Rule Agreement | `PLAYBOOK` §20.2 |
| `RULE_OUTCOMES_NOT_RECORDED` | Rule Agreement | `PLAYBOOK` §20.2 |
| `NO_ENVIRONMENT_OBSERVATION` | Environment Stability | `PLAYBOOK` §20.3 |
| `NO_LATENCY_REFERENCE` | Environment Stability | `PLAYBOOK` §20.3 |
| `INSUFFICIENT_EVIDENCE` | Evidence Quality, and the decision itself | `PLAYBOOK` §19 · `02` §5.5 |
| `MISSING_TERM` | Framework Confidence, naming each absent term | `02` §21 · `02` §5.5 |

`PLAYBOOK` §20.1's D-3 additionally requires a refusal when `W` or
`minimumExecutions` is not resolvable through `01` §29; the implementation
records this as `NO_HISTORY_CONFIGURATION`.

**An unavailable measurement SHALL NEVER be replaced with `0`, `50`, `100`, a
configured default, a previous execution's value, or an inferred value.** A
cold-start scope with no history is a legitimate unavailable state, not an error,
and SHALL NEVER halt execution (`10` §7.3).

## Persistence — no new artifact

All three destinations already exist:

| Value | Recorded in | Owner |
|---|---|---|
| Per-item evidence confidence | the `confidence` field of the Evidence Object | `02` §18 |
| Evidence Quality | the decision's evidence record | `02` §36 |
| Framework Confidence, and each term's value or unavailable reason | the decision audit record | `02` §39 |

No dataset, file, or schema is created for this chain.

---

# 5B. Artifact Ownership And Identity

Artifacts persist under their **existing** owners. Never invent a destination,
and never claim ownership of another engine's artifact.

| Artifact | Owner | Canonical document |
|---|---|---|
| Discovery snapshot, component inventory, verified inventory | **03** | `03` §11, §29, §31 |
| Knowledge Graph, graph version, graph diff | **04** | `04` §30 |
| Test Plan, canonical coverage | **05** | `05` §32 |
| Generated tests, Page Objects, fixtures | **06** | `06` §30, §40 |
| Runtime Schedule, execution history, deviation record | **07** | `07` §39 |
| Locator History | **08** | `08` |
| Reports, analytics | **09** | `09` §33 |
| Learning Database | **10** | `10` §13, §7.1 |
| Decision history, evidence record, decision audit | **02** | `02` §36, §38, §39 |

## Identity — consumed, never redefined

| Identity | Owner | Used for |
|---|---|---|
| **Test Case Identity** | **06** §22.1 | Stable identity of a test across regeneration |
| **Execution ID / Correlation ID** | **01** §30 | One run, one causal chain |
| **Execution Scope Identity** | **01** §30.1 | *Which application* this run is about; the historical partition key for `10` |

This file, and every engine, **consumes** these identities. None generates,
redefines, or substitutes one, and no second identity system exists. `projectId`
from the test runner identifies a browser project and SHALL NEVER be used as a
scope identity (`01` §30.1).

## Event model

Components communicate through the **Event Bus** (`01` §27), which `07` §38
integrates with published and subscribed lifecycle events. Publish and subscribe
through that contract rather than invoking engines directly. This file defines no
event type; the Core Event set is `01` §27's.

## Learning

Learning is **downstream of real observations** (`10`). It accumulates per
Execution Scope Identity (`10` §7.2), is pinned immutable within an execution
(`10` §7.5), and treats a cold start as normal (`10` §7.3).

`10` **recommends only.** It SHALL NEVER fabricate an observation, become a
second decision authority, compute confidence, modify tests, Page Objects or
configuration, or introduce a lifecycle state. `02` decides whether a
recommendation applies.

---

# 6. Failure Handling

| Situation | Action |
|---|---|
| **Required engine fails** | **STOP.** Produce a diagnostic report. Never emit partial results as complete. |
| **Optional engine fails** | **DEGRADE, do not stop** (`01` §15). Apply the phase's degrade rule. Report the degradation. |
| **Mandatory gate FAIL** | Stop; diagnostic report. |
| **Gate WARNING** | Advance; record the finding. |
| **Mandatory input missing** (`BASE_URL`, environment, capability) | Stop at the gate and report. Never substitute a guess for evidence. |
| **Degraded mode** | Only `02` may approve. Log the reason; `09` reports it. |
| **Abort** | `FAILED` or `CANCELLED`. Preserve all collected evidence. |
| **Rollback** | Framework state only, forward-only except the 02-approved `PLANNING_REVIEW → PLANNING` loop. Never undo executed browser actions. |
| **Re-plan** | Observation → Evidence → **Re-plan Request** → `02` → `05` → `15` → `06` → Validation → Execution. Runtime SHALL NEVER generate tests directly. |
| **Recovery** | Classify first (`08` §16), then `02` approves. Preserve evidence across recovery. |
| **Environment interstitial** | Classified as an **environment issue**, ordered ahead of every other rule. Never scored as a regression (Playbook §15). |
| **Evidence set empty** | `evidenceQuality` undefined → refuse the decision `INSUFFICIENT_EVIDENCE` (`02` §5.5). Never zero, never defaulted (§5A). |
| **A `02` §21 term unavailable** | `02` refuses Framework Confidence with `MISSING_TERM`, naming every absent term. The decision still proceeds on rules, policy and risk; only the confidence value is refused (§5A). |
| **No history for this scope** | Cold start is normal (`10` §7.3). Historical Reliability is `NO_HISTORY`; execution SHALL NOT halt and the absence SHALL NOT be reported as a failure. |
| **Required measurement input missing** | Refuse with the named state. Never substitute `0`, `50`, `100`, a default, a previous value, or an inferred value. |

## Failure Classification

Classify **before** considering any recovery (`08` §16). The classification
determines what the failure means and whether healing is even permitted:

| Classification | Meaning | Healing |
|---|---|---|
| `LOCATOR` | The element exists; its address drifted | Permitted, under `02` approval and `08` §32 limits |
| `ASSERTION` | An expectation was not met — usually a **target defect** | **Prohibited** (`08` §13) |
| `ENVIRONMENT` | The target or infrastructure misbehaved | Prohibited; not a regression |
| Business logic · backend · auth · security · data corruption | Real defects | **Prohibited** (`08` §13) |

A target defect and a suite defect are different findings and SHALL be reported
separately (§9). Evidence SHALL be preserved across every recovery attempt and
every terminal state, including `FAILED` and `CANCELLED`.

## Degradation Ledger

Every degraded run SHALL state, in the report: which optional engines ran, which
were skipped, why, and what the skip cost in assurance. An optimized,
early-stopped, incrementally-discovered, or credential-limited run SHALL NEVER be
presented as complete.

---

---

<!-- nav -->
*← [part 5](execution-entry-point-05.md)  ·  part 6 of 7  ·  [part 7](execution-entry-point-07.md) →*  ·  [reference index](INDEX.md)
