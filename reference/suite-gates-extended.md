# Suite Gates — Extended Checks (A2 · A3 · B2 · C-replay)

> Executor reference. **Phases 6 VALIDATING, 7 EXECUTING, 9 DIAGNOSTICS.** Companion
> to [`suite-self-validation.md`](suite-self-validation.md), which defines Gates A, B
> and C. Every check here exists because FM-11 passed the original three: each of
> them inspected only **one side** of a comparison.
>
> The unifying rule: **a gate must compare two things.** Plan against emitted source.
> Perturbed run against normal run. Asserted state against the state reached.

### Provenance in the metadata is not provenance in the test

Everything above inspects the **assertion**. It says nothing about whether the
emitted test *honours* it. A test can cite a real measurement, a real value and a
real scope, and then never enter the state that measurement was taken in — and
this Gate passes it (FM-11). Two further checks close that gap, and both compare
the **plan against the emitted source**:

**A2 · Interaction emitted.** Every case carrying a planned `interaction` MUST
contain that interaction in its generated body. A planned interaction that reaches
no emitter is a **FAIL**, never a silently simpler test.

> Generation therefore dispatches on `interaction.kind` and **never on the case
> title, description, or any other generated prose.** Prose dispatch makes emitted
> behaviour depend on wording: rename a test and it silently stops doing what it
> claims. Where a case's interaction matches no emitter, generation **throws**; it
> never falls through to a route-only test.
>
> `kind` is a **closed set describing mechanical shape** — `fill_then_submit` ·
> `submit_empty` · `fill_only` · `activate` — never a business concept. A
> domain-named kind adds one branch per concept per target and drifts the generator
> back toward per-application code (FM-11). Which controls and which values are
> already carried by `fills[]` and `submit`.

**A3 · Trigger state entered.** An assertion may cite a measurement that required a
control to be driven **only** if its test drives one.

> Whether a measurement required interaction is recorded **by the probe, at
> measurement time** — `requiresInteraction: true | false` on the probe record. It
> is never inferred from the wording of a `triggerState` label: label text is
> language- and phrasing-bound, exactly the mistake FM-9 documents. A run that
> parsed the label instead produced 462 false positives; the recorded boolean
> produced one true positive and no false ones.

Both checks are cheap, and both catch a class the original Gate A cannot see: work
that **contradicts its own recorded intent**.

---


### Falsifiable is not the same as sensitive — the conjunction is mandatory

The table above reads **only the perturbed run**, and that is not sufficient. A test
that fails *unconditionally* also fails when perturbed, so it scores `FALSIFIABLE`
while being broken. This is precisely how FM-11 passed this Gate.

Sensitivity is a property of **two runs together**, so it is resolved after the
execution pass, not inside the perturbation pass:

| Unperturbed | Perturbed | Verdict |
|---|---|---|
| passes | **fails** | `SENSITIVE` — genuinely responds to its own subject |
| passes | passes | `VACUOUS` — proves nothing |
| **fails** | fails | `INCONCLUSIVE_FAILS_REGARDLESS` — the perturbation explains nothing; diagnose the failure first |
| fails | passes | `INCONCLUSIVE` — inverted and incoherent; investigate |

Only `SENSITIVE` counts as proven coverage. `INCONCLUSIVE_FAILS_REGARDLESS` is
reported in its own row and **never folded into the falsifiable count** — folding it
in is what let a broken test be presented as validated coverage.

Persist this as `execution/negative-control-sensitivity.json` and report the
sensitivity totals beside the falsifiability totals.

Minimum artifact shape:

```json
{
  "schemaVersion": "1.0.0",
  "gate": "B2 - Negative Control Sensitivity",
  "normalRun": "execution/execution-summary.json",
  "perturbedRun": "execution/gate-b-negative-control.json",
  "totals": {
    "sensitive": 119,
    "vacuous": 0,
    "inconclusive": 0,
    "inconclusiveFailsRegardless": 0,
    "notMeasured": 0
  },
  "perTest": [
    {
      "testCaseId": "TC-001",
      "normal": "passed",
      "perturbed": "failed",
      "verdict": "SENSITIVE"
    }
  ]
}
```

If B2 cannot run, write the artifact anyway with `state: "NOT_PRODUCED"`,
`reason`, and `impact`, and do not count Gate B as proven coverage. A report may
say "250 tests were falsifiable" only when Gate B ran; it may say "250 tests were
sensitive" only when this B2 artifact proves the normal/perturbed pair.


### The probe MUST reproduce the test's conditions, not merely its route

A probe is only evidence about the target if it puts the target in the **same state
the assertion was measured in**. Concretely, before re-measuring it MUST replay the
failing case's recorded `interaction` — the fills, the submit, the activation — and
not merely re-open `scope.route`.

> A probe that opens the route and skips the action **reproduces the suite's own
> omission** and then reports the result as the target's behaviour. That is not a
> weaker measurement; it is a measurement of the wrong thing, and it produced a
> `TARGET_BEHAVIOUR_DIFFERS_FROM_MEASUREMENT` verdict against an application that
> was behaving correctly (FM-11).

Therefore:

- Replay the interaction, then measure. Record `interactionReplayed: true`.
- If the interaction cannot be replayed, the probe is **not** evidence about the
  target. Emit `UNCLASSIFIED` with `interactionReplayed: false` and state that the
  cause is undetermined. Never infer drift, non-determinism, or a target change
  from a probe that did not reach the asserted state.
- A classification of drift requires the **replayed** measurement to disagree.
  Before attributing anything to the target, check whether the suite reached the
  state it claimed to — `A2`/`A3` above exist to make that answerable.

---
