# Session Implementation Record

Complete record of one working session: what was built, what was changed, why,
and what went wrong. Written from the artifacts and git state, not from memory.

**Date:** 2026-08-18 · **Skill:** `qa-automation` (Site Explorer)

Work spans three directories:

| Directory | Role |
|---|---|
| `/home/kali/.claude/skills/qa-automation/` | The skill — specification + executor playbooks (no code) |
| `/home/kali/qa-w7c-impl-20260811/` | W7-C implementation surface — where the runtime output layer was built |
| `/home/kali/qa-orangehrm-20260818/` | Output of the live OrangeHRM engagement (6 runs) |

---

## Part 1 — The OrangeHRM engagement (runs 01–06)

### What was asked

Run the `qa-automation` skill against `https://opensource-demo.orangehrmlive.com/`,
an operator-authorized target.

### Steps taken

| Phase | Action | Result |
|---|---|---|
| 0 CONFIGURING | Resolved config, assigned `EXEC-20260818-0001`, tooling pre-flight | node 20.20.1, Chromium 1.62.1 present |
| 1 DISCOVERING | Unauthenticated crawl, then authenticated | 1 public page → 28 authenticated page loads |
| 2 BUILDING_GRAPH | Deduped by `finalUrl` | 22 unique nodes, 12 modules, 59 API endpoints |
| 3 Locator Verification | Probed every candidate live | 204 probed → 145 verified / 59 pruned |
| 4 PLANNING | Coverage, risk, browser matrix | 83 → later 121 cases |
| 5 GENERATING | Page objects, fixtures, specs | 18 → 21 files |
| 6 VALIDATING | `tsc --noEmit` + enumeration | PASS |
| 7 EXECUTING | Chromium only, 6 runs | see below |
| 9/10 | Diagnose + report | evidence-cited classification |

### Run results

| Run | Passed | Failed | Flaky | Target health | What it established |
|---|---:|---:|---:|---|---|
| 01 | 46 | 36 | 1 | DEGRADED (7534ms vs 939ms ref) | fixture-scope collision (21), over-generalised probe (13) |
| 02 | 78 | 5 | 0 | HEALTHY (1075ms) | expectations built from requests, not responses |
| 03 | 34 | 50 | 0 | HEALTHY (317ms) | proved run-02 fixes correct; exposed session collapse |
| 04 | 83 | 1 | 0 | HEALTHY (397ms) | **exposed a false pass** |
| 05 | 83 | 0 | 1 | HEALTHY (1264ms) | green |
| 06 | **121** | **0** | 0 | HEALTHY (587ms) | API coverage 20 → 57 endpoints |

**92 test failures across six runs. Zero were defects in the target application.**
Every one was a defect in the suite or the pipeline that generated it.

### Files created — `/home/kali/qa-orangehrm-20260818/qa/`

| File | Purpose |
|---|---|
| `lib/dom-probe.js` | Generic, standards-based DOM extraction (accname algorithm, ARIA, landmarks) |
| `tools/profile-site.js` | Measures a site's conventions instead of hardcoding them |
| `tools/discover.js` | Observational BFS crawl |
| `tools/discover-deep.js` | Depth-first, menu-expanding crawl |
| `tools/auth-setup.js` | Session mint from target-published demo credentials |
| `tools/build-graph.js` | Knowledge graph |
| `tools/verify-locators.js` | Phase-3 gate: probe every candidate live |
| `tools/verify-scoped.js` | Label-scoped recovery of ambiguous inputs |
| `tools/plan.js` | Test planning |
| `tools/generate.js` | Playwright suite generation |
| `tools/report.js` | Diagnostics + decision records + evidence chain |
| `tools/rw-report.js` | Markdown report |
| `tools/html-report.js` | HTML report |
| `tools/probe-followups.js` | F1–F3 root-cause probes |
| `tools/probe-api-auth.js` | F4: is the 401 the app or the suite? |
| `tools/probe-api-query.js` | F5: do dropped query strings explain the 422s? |
| `tools/probe-api-baseline.js` | Measured response baseline for all 58 GETs |
| `tools/probe-session-policy.js` | F6: single-session-per-user? (refuted) |
| `tools/probe-session-longevity.js` | F7: session longevity (inconclusive) |

### Root causes found, and why each mattered

1. **Fixture-scope collision (21 failures).** A file-scoped
   `test.use({storageState: empty})` applies to the whole file, stripping the
   session from authenticated tests in it. Proven by measurement: the same
   endpoints return `200` with a session and `401` without.

2. **Over-generalised probe (13 failures).** A shell anchor measured on
   `/dashboard/index` was promoted to a global invariant. Measured per route it
   resolves to **0, 1, or 2**.

3. **Request-not-response expectations (3 failures).** Discovery recorded which
   API calls the app makes but never what came back, so the planner asserted
   `200` everywhere. Two endpoints need query params (`422`); one is an optional
   sub-resource (`404`).

4. **Session collapse (50 failures).** Shared storage state invalidated mid-run.
   Two hypotheses tested and **refuted** (single-session-per-user; logout
   interference). **Root cause never established.**

5. **A false pass (the most valuable finding).** Run 04 finished 83/84 and looked
   like success. Both boundary tests claimed to exercise the "Employee Name"
   filter and were actually driving the **sidebar menu search**. The page
   snapshot showed `(75) Records Found` — no filter had been applied. One test
   failed and exposed it; the other **passed**, reporting coverage that did not
   exist.

6. **Arbitrary cap.** `slice(0, 20)` silently hid 37 of 58 API endpoints.

### Mistakes I made during the engagement

- **Overwrote evidence.** The authenticated discovery run wrote to the same
  filenames as the unauthenticated one and destroyed the first report.
- **Collided identifiers.** Two discovery passes each numbered from `CAND-0001`,
  breaking the TypeScript gate.
- **Over-read a probe.** Probe F1 showed only that a *newer* session survives a
  logout of an *older* one. I generalised that to "sessions are independent" and
  built run 03 on it. Later measurement refuted the strong reading.
- **Claimed session expiry from F7** when the datapoint was a transport `ERR`,
  not a `401`.
- **Corrupted three run archives.** `reports/run-02`, `run-04`, `run-05` contain
  a `--list` invocation (`expected:0, skipped:84`) rather than real results,
  because the copy happened at the wrong moment. The live numbers reported at the
  time were read correctly; the archives are wrong. This is an instance of the
  very failure mode later recorded as **FM-7**.

---

## Part 2 — Skill changes

The user's correction — *"note that these changes happen in the skill, not the
test"* — was decisive. Fixes applied to a run's output directory die with that
directory. The skill is specification + playbooks, so fixes land as **mandated
technique** and then apply to every future run on every site.

### Files created — `reference/`

| File | Lines | Why it exists |
|---|---:|---|
| `site-agnostic-discovery.md` | 195 | The pipeline had **126 hardcoded OrangeHRM selectors** — it could only ever work on one app. Mandates: profile the target, standards-based labels (accname), behavioural custom-widget detection, locator ladder, intent binding, depth-first traversal, read-only interaction boundary |
| `coverage-ledger.md` | 123 | 37 endpoints and 116 verified elements vanished with no record. Every artifact must end `TESTED` or `EXCLUDED` with a closed-vocabulary reason; `unaccounted > 0` blocks a completeness claim |
| `known-failure-modes.md` | 182 | Ten real defects with measured cost, each with the rule that prevents it |
| `suite-self-validation.md` | 153 | **The root-cause fix** — assertion provenance, negative control, classification provenance |
| `write-operations-and-test-data.md` | 127 | `CRUD-except-Delete` was declared with no method, so six runs produced zero write tests |
| `final-report-contract.md` | 253 | The 15-section report, stable JSON contracts, retry/healing honesty |
| `w7-api-evidence-contract.md` | 188 | Binding preservation of W7-A/B/C and AIC v1.0.0 |

### Files modified

| File | Change |
|---|---|
| `SKILL.md` | Phase table (added 0B PROFILING); discovery redefined observational → read-only; output table → 12 categories; **Rules 11–25 added** |
| `reference/quickstart.md` | Rewritten: scope → profile → discover → verify → plan → generate → execute → report; all paths migrated |
| `reference/output-and-scope.md` | Rewritten twice — first to a lifecycle layout, then to the Site Explorer 11-category tree |
| `reference/INDEX.md` | Registered 7 new reference files |
| `reference/evidence-and-confidence.md`, `execution-entry-point-01.md` | Path migration |
| `docs/03_Discovery_Engine.md` | §36 observational → read-only, depth-first, no arbitrary caps |
| `docs/05_Test_Planning_Engine.md` | Added binding obligations **P-1…P-5** |
| `docs/06_Test_Generation_Engine.md` | Added binding obligations **G-1…G-7** |

`docs/05` and `docs/06` are the *specification authority*. Leaving them silent
while `reference/` mandated new behaviour would have been a contradiction inside
the skill.

### The root cause (the most important finding of the session)

Twenty rules were symptom patches — they only prevent failures already known.
The user asked for the actual cause. It is a single asymmetry:

> The skill applies *"measured, never authored"* to **what it reports**
> (confidence), but never to **what it builds** (locators, assertions,
> classifications) — and no gate verifies the suite is capable of failing.

Framework Confidence *refused to exist* rather than substitute a default, while
three feet away an assertion cheerfully claimed `200` on an endpoint nobody had
called. Phase 6 `VALIDATING` checked only that the code **compiles**; 121 tests
all asserting the wrong thing compile perfectly.

Three gates now close it:

- **Gate A — Assertion Provenance.** Every assertion cites `derivedFrom`,
  `measuredValue`, `scope`. No provenance → validation FAILS.
- **Gate B — Negative Control.** Run each test under perturbation. Still passes →
  `VACUOUS`, quarantined, counted as zero coverage. *(The run-04 false pass would
  have died here in seconds instead of surviving on luck.)*
- **Gate C — Classification Provenance.** A target defect requires a probe; the
  record must name it. Otherwise `UNCLASSIFIED`.

These generalise because they need no advance knowledge of what will go wrong.

### Regressions I introduced against W7, and corrected

| # | Regression | Correction |
|---|---|---|
| 1 | Bulk-replay of endpoints (duplicate API capture) | Withdrawn — consume W7-C captured responses; `verification` probes are the narrow exception |
| 2 | `network/` implied a canonical API model | Now explicitly a derived view over the AIC |
| 3 | `PAGE-001`/`API-001` = second API identity | AIC `sha256[0:16]` ids reused verbatim; readable ids are display aliases |
| 4 | `observed/inferred/generated` = parallel provenance | Mapped onto W7-B's `discoverySource` + state ladder |
| 5 | Rule 12 "no caps" contradicted W7-C | Reconciled: no *hardcoded* ceiling; caps from `01` §29; missing cap = disclosed config gap |
| 6 | Re-specified masking | Points to the single existing authority |
| 7 | Invented `TEST_ENV_CLASS` | Withdrawn — `ALLOW_WRITE_TESTS` (W7-A BD-W7-3) already governs writes |

**A claim I withdrew:** I asserted `CRUD-except-Delete` + "never delete" was an
irreconcilable contradiction producing permanent residue. It is not —
`ALLOW_WRITE_TESTS` already requires synthetic data **and mandatory cleanup**. I
reasoned from a working tree that had the governing control deleted.

---

## Part 3 — Runtime output/report implementation

Specification alone was `NOT DEMONSTRATED`. This part made it real and executable.

### Read-only inspection first

| Surface | Finding |
|---|---|
| `/home/kali/qa-w7c-impl-20260811/` | The real W7-C implementation: `absence`, `aic-map`, `capture`, `discovery`, `identity`, `masking`, `state` — **55/55 tests passing** (baseline) |
| `/home/kali/Desktop/runner/kali_runner/result_ingest.py` | The consumer. Reads `finding.json` and **`output.rglob("*")`** — *not* a whitelist |

**The compatibility risk I had flagged was void.** `result_ingest` reads no
qa-automation path; it ingests every readable file under `output/`. Moving
`qa/reports/results.json` breaks nothing.

### Files created — `/home/kali/qa-w7c-impl-20260811/`

| File | Lines | Purpose |
|---|---:|---|
| `lib/output-package.ts` | 472 | Projects canonical W7 evidence into `./output/qa/**`. Never crawls, re-requests, re-masks, or mints an API identity |
| `lib/final-report.ts` | 324 | Renders `final-report.md` / `.json` / `README.md` **from the written artifacts only** — so every number exists in an artifact by construction |
| `tests/output-package.spec.ts` | 320 | 25 tests over the 34 required areas |
| `tests/e2e-real-capture-to-report.spec.ts` | 294 | 12 tests: real Chromium → capture → AIC → package → report |
| `tests/fixture-input.ts` | 119 | Shared synthetic fixture |
| `tests/emit-fixture-package.spec.ts` | 15 | Emits an inspectable package |

**Zero W7 modules modified.** All seven original `lib/*.ts` untouched; all five
original spec files untouched.

### Honesty rules enforced in code, not prose

- Coverage emitted only when numerator *and* denominator exist, else `Absence` +
  limitation
- Workflows `[]` + limitation (journeys are `NOT_PRODUCED`, owned by W7-F)
- States honestly empty + limitation (no producer exists)
- Healing counted **only** where a `locator_healing` attempt was recorded — a
  retry that succeeded is never called healing
- `failed` and `blocked` never merged

### Test results

| Suite | Tests |
|---|---:|
| Original W7-C (aic-map, capture-state-evidence, discovery, masking, fixture-integration) | **55** — unchanged, no regression |
| `output-package.spec.ts` | 25 |
| `e2e-real-capture-to-report.spec.ts` | 12 |
| `emit-fixture-package.spec.ts` | 1 |
| **Total** | **93 passed / 0 failed** |

### Real-capture verification

```
method GET  path /api/profile  query [verbose]
header accept        repr=plain   application/json
header authorization repr=masked  ***MASKED***
correlation {executionId: EXEC-E2E-0001, apiId: api:03236115bed4a886}
response status=200 / 201  bodyRepr=redacted
```

44 files emitted. All reconcile: `pages_discovered`, `api_endpoints_observed`,
`tests_generated`, `passed`, `report.execution.passed`.

`result_ingest.ingest_results()` run against the real package: **OK, 44 evidence
calls, 0 findings promoted** (correct — QA emits none).

### Bugs my own tests caught

1. **Method conflation.** The report printed "Methods observed: GET, POST" where
   `GET` belonged to a *discovered-only* endpoint — exactly the conflation the
   spec forbids. Fixed, with a regression test.
2. **Fixture-vs-production divergence.** My synthetic fixture invented
   `evidenceRef: 'ev:req1'`. The real W7-C shape is
   `evidence/requests/call:<hash>.json`. Only the E2E test against production
   capture code exposed this — the exact reason that phase existed.

---

## Open items

| # | Item | Status |
|---|---|---|
| 1 | **W7/W8 doc deletions** — `ALLOW_WRITE_TESTS`, `No-Re-Crawl (W8, folding W7-A §11)`, `W7-B §C11` present in `HEAD`, absent from the working tree. **Predates this session.** Not restored — reverting someone's uncommitted WIP could destroy intentional work | **Needs a decision** |
| 2 | No production orchestrator calls `buildOutputPackage` — proven in a test harness, not wired into a shipped executor, because none exists in this workspace | **Blocker for READY_FOR_UI** |
| 3 | `set-cookie` masking not exercised end-to-end — Playwright's `allHeaders()` omitted it under interception. Covered by unit tests, not E2E | Gap |
| 4 | Content is mocked per W7-C §26; real-target runtime tier is the separately-authorized **W7-H** wave | By design |
| 5 | Site-agnostic claim validated on **one** application; portability needs a second, structurally different target | `NOT DEMONSTRATED` |
| 6 | Run-03 session collapse never root-caused | Mitigated, unexplained |
| 7 | Three corrupt run archives (`run-02`, `run-04`, `run-05`) | Recorded above |

## Final status

**NOT_READY_FOR_UI** — 15 of 16 completion criteria pass. The failing criterion is
*"real/integration Site Explorer execution reaches output generation"*: the chain
is proven end-to-end through production capture code, but the caller is a test
rather than a shipped executor.

---

## The line worth keeping

> Across six executions: **92 test failures, none in the target application.**
> Nine of ten known failure modes produced a clean type check and a green-looking
> pipeline while being wrong; two produced *passing tests asserting the wrong
> thing*.
>
> A suite cannot validate itself. Only a measurement against the live target
> separates "my expectation was wrong" from "the application is broken" — and the
> default assumption must be the former.
