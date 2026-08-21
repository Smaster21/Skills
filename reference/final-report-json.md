# Report Data Contracts — `final-report.json`, `summary.json`, `run.json`

> Executor reference. **Mandatory — Phase 10.** The human report and its section list live
> in `final-report-md.md`; this file carries the machine-readable contracts,
> retry/healing reporting, failure evidence, the README, and the checklist.

## `report/final-report.json` — stable top-level contract

Structured equivalent of the report, suitable for future API/UI consumption.
Top-level keys are **fixed**; consumers may rely on their presence.

```json
{
  "report_version": "1.0.0",
  "run": {}, "exploration": {}, "scope": {},
  "discovery": {}, "network": {}, "knowledge": {},
  "workflows": [], "tests": {}, "execution": {},
  "evidence": {}, "coverage": {}, "confidence": {},
  "limitations": [], "conclusion": {}
}
```

Rules: no security fields; no duplication of sensitive data; every workflow, test
and evidence reference carries its **stable ID** so a UI can join without parsing
filenames. API entries **reuse AIC identifiers verbatim** and carry the W7-B
state and `provenance.discoverySource` — the JSON is a projection of the AIC, not
a second API model (`w7-api-evidence-contract.md`).

---

## `summary.json` — compact result

Only the high-level result a future API/UI needs:

```json
{
  "status": "completed",
  "mode": "authenticated",
  "pages_discovered": 47,
  "routes_discovered": 83,
  "workflows_discovered": 18,
  "api_endpoints_observed": 31,
  "tests_generated": 142,
  "tests_executed": 137,
  "passed": 128, "failed": 9, "blocked": 5,
  "evidence_count": 231,
  "playwright_artifacts_available": true,
  "qa_coverage": { "pages": 89.4, "routes": 91.6, "workflows": 83.3 },
  "framework_confidence": {},
  "limitations": []
}
```

`blocked` and `failed` are **distinct** and never merged: blocked means the test
could not run to a verdict; failed means it ran and the expectation was not met.

`api_endpoints_observed` counts state `OBSERVED` only. Endpoints known solely from
`js`/`sitemap`/`robots` are `DISCOVERED` and reported separately — conflating them
would claim calls that never happened. Absent values use the W7-B vocabulary
(`UNAVAILABLE`, `NOT_OBSERVED`, `NOT_PRODUCED`), never bare `0` or `null`.

---

## `run.json` — execution metadata only

```json
{
  "run_id": "…", "project_id": "…",
  "started_at": "…", "completed_at": "…", "duration_seconds": 751,
  "status": "completed",
  "exploration_mode": "authenticated",
  "entry_url": "…",
  "scope_source": "scope.json", "roe_source": "roe.json",
  "credentials_used": true,
  "framework_version": "…", "skill_version_hash": "…"
}
```

`credentials_used` is a **boolean**. Credential values, usernames and secrets are
never stored here or anywhere else.

---

## Retry and self-healing reporting — no overclaiming

Report attempts as they happened:

```
Attempt 1 → normal execution        → failed (locator_failure)
Attempt 2 → adaptive wait/recovery  → failed (timeout)
Attempt 3 → locator self-healing    → passed
Final: PASSED (healed)
```

> **Never claim healing that did not occur, and never call a plain retry a
> recovery.** If attempt 2 simply succeeded, report a retry — not healing.
> `diagnostics/locator-healing.json` records healing **only when it was actually
> attempted**.

---

## Failure evidence — predictable per test

```
evidence/tests/TC-LOGIN-001/
├── failure.json      what/where/why, retries, locator used, healing, final result, artifact paths
├── screenshot.png
├── trace.zip
├── video.webm
└── network.json
```

Create **only files that actually exist**. `failure.json` must be understandable
without opening the trace.

---

## `README.md` — plain-language directory guide

```
qa/
├── discovery/   → what the application contains
├── network/     → APIs / network activity observed
├── knowledge/   → application model learned
├── planning/    → exploration and test planning
├── tests/       → generated QA tests + catalogue
├── execution/   → test execution results
├── evidence/    → screenshots, traces, videos, raw evidence
├── coverage/    → application and QA coverage
├── diagnostics/ → retries, failures, healing
├── report/      → final human-readable report
└── raw/         → low-level execution/debug data
```

Generated at the end of every run, never hand-maintained.

---

## Confidence — one model, several views

The report presents confidence in discovery, application model, workflow model,
generated tests, locators and API mapping. These are **views over measured
evidence**, not new scores.

> The AIC is **not** a second Evidence Quality or Framework Confidence model
> (W7-A). `02` §21 remains the **sole** Framework Confidence authority (`Rule 5`). Per-area
> figures are reported as the measured Evidence Quality and coverage for that
> area, clearly labelled as such. Do **not** introduce a second confidence model,
> and never express confidence as vulnerability likelihood, exploitability or
> risk.

Where confidence is low, say **why** — small evidence set, un-targetable controls,
partial API capture, unstable environment.

---

## Acceptance checklist — a run is not complete until all hold

**Artifacts**

- [ ] output root is `./output/qa/`; nothing written to `findings/`, `security/`, `redops/`
- [ ] `run.json`, `summary.json`, `README.md` generated
- [ ] discovery, network, knowledge, workflow, planning artifacts generated
- [ ] `tests/catalogue.json` generated and traceable to generated specs
- [ ] `execution/execution-summary.json` generated
- [ ] per-test evidence directory exists for every failed test (only real files)
- [ ] retries recorded; locator healing recorded **only when attempted**
- [ ] `coverage/coverage-summary.json` generated
- [ ] `report/final-report.md` and `report/final-report.json` generated
- [ ] stable IDs assigned; `observed`/`inferred`/`generated` preserved throughout

**Correctness**

- [ ] traceability page → workflow → test → execution → evidence resolves end to end
- [ ] `failed` and `blocked` are distinct everywhere
- [ ] authenticated and unauthenticated modes each reported correctly
- [ ] partial API capture reported with `api_capture_status` and a reason
- [ ] report remains readable when sections are unavailable
- [ ] identical input produces identical artifact names and locations
- [ ] **every narrative claim about this run resolves to an artifact of this run** — each pass, defect, skipped engine, gate outcome and named directory exists on disk. No fixed narrative, no event or path carried over from a prior run (`final-report-md.md` → *Narrative sections are derived, never authored*)

**Isolation and safety**

- [ ] no security fields anywhere: no findings, CVSS, exploitability, attack
      priority, security applicability/coverage, security recommendations
- [ ] no credential, token, cookie, or authorization-header value in any artifact

> **A self-check MUST distinguish a thing from a statement about that thing.** This check wants absence of *asserted* security content, but `final-report-md.md` requires the report to declare it carries none — so a bare substring scan fails the run for obeying its own spec. Judge each match in its clause (negated = compliance), treat a soft-wrap as no boundary, and exclude the check's own report (`suite-gates-extended.md`).

- [ ] sensitive network fields masked; existing masking not weakened
- [ ] no new security decision logic; security skills and their prompts untouched
- [ ] existing evidence ingestion still resolves its expected artifacts
- [ ] normal RedOps run behaviour unchanged

A checklist item that cannot be satisfied is a **declared limitation** in
report §14, never a silent omission.
