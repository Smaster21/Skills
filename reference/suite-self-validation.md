# Suite Self-Validation — provenance and falsifiability

> Executor reference. **Mandatory gate — Phases 5–6 (and 9 for Gate C).** This is the structural fix for the root
> cause behind the ten entries in `known-failure-modes.md`.

## The root cause

The framework is uncompromising about evidence **it reports**. `Rule 4` —
*"Evidence is observed, never authored"* — governs Evidence Objects. `02` §21
**refuses** to emit Framework Confidence rather than substitute a default term.
That discipline is the best thing in this skill.

It was never applied to the things the suite is **made of**.

| Artifact | Discipline applied | Result |
|---|---|---|
| Evidence Object → confidence | measured, never authored, refuses when absent | correct in every run |
| **Locator** | verified to resolve to 1 element — but not to be the *right* element | a passing test drove an unrelated control |
| **Assertion** | none — could be authored from nothing | `200` asserted on endpoints never called; a one-route count asserted globally |
| **Classification** | none — inferred from failure text | 22 suite defects filed as target defects |

And the only gate on the suite itself, Phase 6 `VALIDATING`, checks that the code
**compiles**. A suite of 121 tests that all assert the wrong thing compiles
perfectly.

> **Root cause, stated once:** the skill's evidence discipline governs what it
> *reports* but not what it *builds*, and nothing verifies the suite can fail.

Everything else follows. Nine of ten known failure modes produced a clean type
check and a green pipeline while being wrong.

You cannot enumerate the failure modes of the next target in advance. You can
require **provenance** and prove **falsifiability**. Those two generalise; a list
of remembered mistakes does not.

---

## Gate A — Assertion Provenance

> **Every generated assertion MUST cite the measurement it was derived from.
> An assertion with no provenance is a defect and FAILS the validation gate.**

This is `Rule 4` extended from evidence to expectations. An expectation is a claim
about the target; a claim with no measurement behind it is authored, not observed.

Each assertion carries:

| Field | Meaning |
|---|---|
| `derivedFrom` | artifact + path of the measurement (e.g. `network/api-inventory.json#rows[12].measuredStatus`) |
| `measuredValue` | the value actually observed |
| `scope` | where the measurement is valid — **which route**, which auth state, which trigger state |

An **input value** is a claim about the target too, and carries the same burden:
`valueSource` citing the measurement it came from. Keep any human explanation in a
separate `valueRationale` field — **never let prose leak into the value itself**. A
form posted with `"user@example.invalid (RFC 6761 reserved)"` is not the value that
was measured, and the test no longer sends what the probe observed.

### Consequences, by construction

- An endpoint expectation must come from a **replayed response**, so `200` can
  never be assumed (FM-3).
- A baseline carries the route it was measured on, so it cannot be asserted on a
  different route (FM-2).
- An assertion about a triggered state (error banner, validation message) must
  cite the probe that observed it triggered (FM-6).

### Permitted provenance-free assertions

Exactly one class: **invariants of the protocol or the framework**, not of the
target — for example "HTTP status is a number", "the page has a URL". These carry
`derivedFrom: "PROTOCOL_INVARIANT"` and must be rare. Anything describing the
application's behaviour needs a measurement.

### A generated artifact may not embed the target

Alongside the assertion checks, Phase 6 scans **every generated file** — runner config,
fixtures, page objects, specs — for an embedded target origin. A literal scheme+host
that also appears in the measured evidence is a **FAIL**, not a convenience: the run
must read its target from the environment and abort when it is absent
(`scope-enforcement.md`). This is the same rule as target-agnosticism, applied to
configuration rather than to locators.

### Provenance in the metadata is not provenance in the test

Everything above inspects the **assertion**, not whether the emitted test *honours*
it. A test can cite a real measurement, value and scope and never enter the state it
was measured in — and this Gate passes it (FM-11). Two further checks close that gap,
both comparing the **plan against the emitted source**: **A2 interaction-emitted**
and **A3 trigger-state-entered**, with the no-prose-dispatch rule and the closed
`interaction.kind` vocabulary: [`suite-gates-extended.md`](suite-gates-extended.md).

## Gate B — Negative Control (falsifiability)

> **A test that cannot fail is not a test. Before a suite is trusted, prove each
> test is capable of failing.**

The locator gate proves a locator resolves to exactly one element. It cannot prove
it resolves to the **right** element — and that gap produced a test which passed
while driving an unrelated control, reporting coverage that did not exist. It was
caught only because a sibling test happened to fail. That is luck, not process.

### Procedure

For each generated test, run it once under **perturbation** — invalidate the thing
it claims to exercise, without touching the application:

| Test kind | Perturbation |
|---|---|
| Element assertion | rebind the locator to a deliberately non-matching target |
| API status/shape | request a deliberately invalid variant of the endpoint |
| Auth-dependent test | run with the session removed |
| Workflow (fill → submit → assert) | skip the submit step |

Record the outcome:

| Result under perturbation | Verdict |
|---|---|
| Test **fails** | `FALSIFIABLE` — it *can* fail |
| Test **passes** | `VACUOUS` — **quarantine it**; it proves nothing |

A `VACUOUS` test SHALL NOT be counted as coverage. It is reported as a suite
defect and ledgered, never silently kept because it is green.

### Falsifiable is not the same as sensitive

The table above reads **only the perturbed run**. A test that fails *unconditionally*
also fails when perturbed, so it scores `FALSIFIABLE` while being broken — exactly how
FM-11 passed this Gate. Sensitivity needs **both** runs and is resolved after
execution: [`suite-gates-extended.md`](suite-gates-extended.md) → Gate B2.

### Cost control

Negative control is cheap relative to what it prevents. Perturbation runs need no
retries, no artifacts, and no full-suite parallelism. Where a full pass is too
expensive, run it across a **risk-weighted sample plus every test whose locator
was bound by a fallback strategy** — those are exactly where false passes live.
Record the sampled fraction in the ledger. **Never report a suite as validated on
an unstated sample.**

---

## Gate C — Classification Provenance

> **A failure may be attributed to the target only when a measurement supports
> it.** The classification record MUST name the artifact proving the cause.

Failure text cannot distinguish a wrong expectation from a real defect: an API
`401` and a genuine authorization bug produce the same assertion string. Absent a
probe, classify `UNCLASSIFIED`. Never default to blaming the target — that is how
22 suite defects were nearly published as defects in someone's application.

### The probe MUST reproduce the test's conditions, not merely its route

A probe is evidence about the target only if it puts the target in the state the
assertion was measured in — so it MUST replay the failing case's recorded
`interaction` before re-measuring, not merely re-open `scope.route`. A probe that
skips the action reproduces the suite's own omission and reports it as the target's
behaviour (FM-11). Full rule and the `UNCLASSIFIED` fallback:
[`suite-gates-extended.md`](suite-gates-extended.md) → Gate C.

## Where these gates run

| Gate | Phase | On failure |
|---|---|---|
| A · Assertion provenance | 6 `VALIDATING`, with the compile gate | **FAIL — stops the run.** Poor automation never executes |
| A2 · Interaction emitted | 6 `VALIDATING`, with Gate A | **FAIL — stops the run.** A dropped interaction is never emitted as a simpler test |
| A3 · Trigger state entered | 6 `VALIDATING`, with Gate A | **FAIL — stops the run** |
| B · Negative control | 6 `VALIDATING`, after compile passes | `VACUOUS` tests quarantined and ledgered; suite proceeds without them |
| B2 · Sensitivity | 7 → post-execution (needs both runs) | non-`SENSITIVE` reported in its own row; never counted as proven coverage |
| C · Classification provenance | 9 `DIAGNOSTICS` | unproven attribution downgraded to `UNCLASSIFIED`; a probe that did not replay the interaction is not evidence |

Phase 6 therefore stops being a syntax check and becomes what its name claims:
validation that the suite is **sound**, not merely that it **parses**.

---

## What to report

Alongside pass/fail and the coverage ledger:

```
Assertions            412 total · 412 with provenance · 0 authored
Interactions          14 planned · 14 emitted · 0 dropped
Trigger states        412 checked · 0 asserting a state the test never entered
Negative control      121 tested · 119 FALSIFIABLE · 2 VACUOUS (quarantined)
Sensitivity           119 SENSITIVE · 0 fails-regardless · sampled 1.00
Classification        5 failures · 5 evidence-backed (5 with interaction replayed)
                      · 0 unproven target claims
```

A suite with authored assertions or vacuous tests is **not** validated, whatever
its pass rate. Report the pass count and these figures together — the pass count
alone has already proven capable of describing a suite that tested the wrong
thing 121 times over.
