# Accessibility Observations — a measured result, not a testing inconvenience

> Executor reference. **Phases 0B–1 (measurement) and 10 (reporting).** Extends
> [`site-agnostic-discovery.md`](site-agnostic-discovery.md), which measures the
> facts this file requires you to report. Report section: `final-report-md.md` §12b.

### Accessibility observations are a reported result, not a testing inconvenience

Profiling and discovery already measure the two facts that matter here: the **label
coverage ratio** and the set of controls for which the accessible-name algorithm
returns nothing. Both are currently consumed only to decide what can be targeted.

That reports the symptom and hides the cause. The causation runs one way:

```
the application does not name its controls
        ↓
assistive technology cannot describe them      ← a defect in the application
        ↓
no locator strategy can bind them              ← what UN_TARGETABLE reports
```

A control with no accessible name is announced by a screen reader as an unlabelled
control. **That is a functional defect of the target**, measured, and it MUST be
reported as one — not solely as our inability to test it.

Report, from data already collected:

| Reported | Source |
|---|---|
| count of visible controls with **no** accessible name, and their routes | discovery |
| label coverage ratio, per sampled route | profiling |
| controls that are pointer-interactive but **not** keyboard-reachable | affordance probing |
| the name source that won, per control | the accessible-name walk |

These are **QA findings** — functional defects in QA categories. They are *not*
security findings, carry no severity score, and stay inside `Rule 21`.

Deeper auditing (contrast, ARIA misuse, heading order, landmark structure) is an
optional extension. If added, it must report **measured violations with provenance**
like any other expectation — never a checklist opinion.

> A run that names 94 un-nameable controls has found 94 real defects and, separately,
> lost the ability to test them. Reporting only the second is a measurement withheld.

Probe on **every sampled route**, not just the entry page: a module's own tabs
usually exist only on that module's pages.

---

