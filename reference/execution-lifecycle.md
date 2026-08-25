# Execution Lifecycle — phase narrative

> Executor reference. The routing table lives in `SKILL.md`; this file carries
> the behaviour that governs how the phases run. Read it before entering
> DISCOVERING, and again before REPORTING.

## The step contract — what each phase must produce, and when it stops

The skill defines **outputs and stopping conditions**, not source code: an executor
builds the programs. Without a fixed contract every run re-invents the pipeline and
the same class of defect recurs — the base rate in `known-failure-modes.md` is *92
failures, none of them a defect in the target*. Every one was in a pipeline rebuilt
from scratch.

So the *slots* are fixed even though the code is not. Each step is a separate
executable unit; **a non-zero exit halts the sequence** and later steps do not run.

| # | Step | Reads | MUST write | Halts the run when |
|---|---|---|---|---|
| 0 | configure | env | `run.json` (at completion) | `BASE_URL` absent · target out of scope |
| 0B | profile | target | `discovery/profile-*.json`, `discovery/discovery-summary.json` | entry route unreachable |
| 1 | discover | profile | `discovery/pages.json` · `routes.json` · `forms.json` · `inputs.json` · `components.json` · `states.json` · `application-map.json` · `transitions.json` · `affordances.json` · `network/*` | — (skips are ledgered, never fatal) |
| 2 | graph | discovery | `knowledge/application-model.json` · `page-model.json` · `workflow-model.json` · `interaction-model.json` | graph invalid — an invalid graph is not persisted |
| 3 | verify | discovery | `discovery/verified-inventory.json` · `assertion-target-probes.json` · `network/api-verification-probes.json` | — (pruned candidates are ledgered) |
| 4 | plan | graph + verified | `planning/test-plan.json` · `coverage/coverage-summary.json` · `diagnostics/coverage-ledger.json` | any population `unaccounted > 0` |
| 4A | plan review | ledger | `planning/plan-review.json` | never — advisory only |
| 5 | generate | plan | `tests/**` · `tests/catalogue.json` | a planned interaction reaches no emitter |
| 6 | validate | plan + generated source | `execution/validation.json` | **verdict `FAIL`** |
| 6 | negative control | generated suite | `execution/negative-control.json` | never — `VACUOUS` tests are quarantined |
| 7 | execute | suite | `execution/results.json` · `results.xml` · artifacts | runner cannot start |
| 7+ | sensitivity | both runs | `execution/negative-control-sensitivity.json` | never |
| 9 | diagnose | results | `diagnostics/failures.json` · `retry-history.json` · `locator-healing.json` | never |
| 10 | report | everything above | `report/final-report.{md,json}` · `summary.json` · `run.json` · `README.md` | a cited artifact is absent |
| 11 | learn | terminal state | learning-store execution record | never |

Three properties this contract fixes, independent of how the code is written:

1. **Halting is mechanical.** A step that fails exits non-zero and the sequence stops.
   A gate whose result nothing consumes is not a gate — `Rule 8` forbids presenting a
   run as complete when part of it did not execute.
2. **Every step is resumable.** Its inputs are files written by earlier steps, so a
   step re-runs without repeating the ones before it. This is what makes a
   partially-completed run recoverable rather than discardable.
3. **Artifact names are the contract.** An executor may organise its programs freely;
   it may **not** rename, relocate, or omit an artifact above. Downstream steps and the
   report join on these paths (`output-and-scope.md`).

Where a step genuinely cannot produce an artifact, it writes the absence in W7-B
vocabulary — `UNAVAILABLE` · `NOT_PRODUCED` · `NOT_EXERCISED` — never an empty file
that reads as a measurement.

---

**Discovery is exhaustive and read-only.** Phase 1 profiles the target, then
crawls it **depth-first** — a page's whole subtree before its next sibling —
mapping routes, components, forms, inputs, parameters and network/API. It **may
open navigation disclosures** (menus, tabs, expanders), because many applications
render their navigation only on activation and a link-only crawl silently misses
most of the application. It **never** submits a form and never activates a
destructive or write-intent control, so discovery still creates, edits and
deletes nothing. Full technique and safety boundary:
`reference/site-agnostic-discovery.md`.

**Nothing is capped and nothing disappears.** No URL cap, no per-category cap, no
`slice(n)`. Every discovered artifact — route, control, action, form, table, API
endpoint — terminates as `TESTED` or `EXCLUDED` with a reason, accounted in
`qa/coverage/coverage-summary.json`. A run with any unaccounted item is not
complete (`reference/coverage-ledger.md`).

**Target-agnostic by construction.** No application-specific selector, class,
URL prefix or text literal is ever hand-written. Locators derive from web
standards (ARIA roles, the accessible-name algorithm, native form semantics) and
from measurements recorded in `qa/discovery/discovery-summary.json`. A hard-coded
framework class is a defect even when it works.

**The suite is validated, not just compiled.** The framework's evidence
discipline governs what it *reports*; it MUST equally govern what it *builds*.
Phase 6 therefore gates three things beyond compilation: every assertion cites
its measurement (**provenance**), every test is proven capable of failing
(**negative control**), and every failure attributed to the target cites the
probe supporting it (**classification provenance**). A test that passes under
perturbation is `VACUOUS`, is quarantined, and counts as no coverage. Full
contract: `reference/suite-self-validation.md`.

**Evidence chain (operationally mandatory).** Execution → Evidence Objects
(`02` §18) → per-item confidence (`PLAYBOOK` §19) → Evidence Quality → `02` §21
Framework Confidence → decision/report artifacts. Framework Confidence has one
authority — `02` §21 — with fixed terms Evidence Quality 0.35 / Historical
Reliability 0.25 / Rule Agreement 0.20 / Environment Stability 0.20
(`PLAYBOOK` §19, §20.1–§20.3). An unavailable term is a **named state**, never a
substituted number; Framework Confidence computes only when all four are
available, else refuses and names the gaps. Full contract:
`reference/evidence-and-confidence.md`.

**Scope & false-positive handling.** Authorized target only; validate against
`SCOPE_FILE` before any navigation (see Rules and `reference/output-and-scope.md`).
Target defects, suite defects, and environment artifacts are classified and
reported separately (`08` §16, `PLAYBOOK` §18).

---
