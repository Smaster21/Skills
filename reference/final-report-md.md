# Final Report — `report/final-report.md`

> Executor reference. **Mandatory deliverable — Phase 10.** The report must be understandable
> by QA engineers, developers, project managers and technical leads **without**
> knowledge of the agent, the runner, or this skill's internals — and without
> reading raw logs or traces.

## What the report must answer

Scope → Discovery → Knowledge → Workflows → Test Generation → Test Execution →
Evidence → Coverage → Limitations → QA Conclusion.

Concretely: what application was explored, under what approved scope,
authenticated or not, what pages/routes/APIs/forms/components were found, what
workflows were identified, what tests were generated and executed, what passed,
failed or was blocked, what evidence exists, what coverage was achieved, what
limited the exploration, how confident the framework is, and what a QA engineer
should take away.

**It is not a security assessment.** No vulnerability findings, CVSS,
exploitability, attack priority, security coverage, or security recommendations.
Never conclude "the application is secure" or "no vulnerabilities found" — those
claims are outside what this skill measures.

## Report type and voice

The human deliverable is a **QA Automation Test Completion Report**. It follows
the same practical shape as a test summary/completion report: scope, activity,
result, coverage, variances, evidence, open gaps, and conclusion.

Write for a QA tester, developer, project manager, and technical lead who have
not read this skill. Every major section starts with one plain-language sentence
answering "what does this mean?" before tables or framework vocabulary.

Internal terms are allowed only when they are translated the first time they
appear:

| Internal term | Plain wording to include |
|---|---|
| provenance | where the expected result came from |
| falsifiable | the test can fail when the checked behaviour is wrong |
| vacuous | the test still passes when it should fail, so it proves nothing |
| UN_TARGETABLE | the tool could not safely choose one exact element |
| tripwire | warning rule |
| rung | locator strategy step |
| AIC / W7-B | stored application/API evidence contract |

If a section needs dense detail, put a short "In simple words" sentence first,
then provide the precise table. Do not make readers learn framework internals to
understand whether the run is useful.

---

## `report/final-report.md` — required sections

| # | Section | Must contain |
|---|---|---|
| 1 | Executive Summary | plain language: what was explored, when, mode, overall result, headline numbers |
| 2 | Exploration Overview | run ID, project, entry URL, mode, start/end, duration, browser/runtime, framework version |
| 3 | Scope & Rules | approved scope, RoE applied, credential availability (**never values**), restrictions, exclusions |
| 4 | Application Discovery | pages, routes, application-map summary, forms, inputs, components, states, navigation relationships |
| 5 | API & Network Discovery | `api_capture_status` (`complete`/`partial`/`unavailable`/`failed`, **with reason when not complete**), endpoint counts **by W7-B state** (`DISCOVERED`/`OBSERVED`/`EXERCISED`), methods, inventory, associated pages/workflows, limitations. **Derived from the AIC — never a re-crawl.** A `DISCOVERED` endpoint is never described as called |
| 6 | Application Knowledge | structure, navigation, workflows, states, forms, API relationships, in understandable language |
| 7 | Discovered Workflows | per workflow: name, purpose, starting page, steps, expected outcome, observed API activity, confidence |
| 8 | Test Catalogue Summary | totals, categories, source workflows/pages, execution status — **summarise, never dump**; full catalogue stays in `tests/catalogue.json` |
| 9 | Test Execution Results | total, executed, passed, failed, blocked, skipped, retries, healed locators, duration |
| 10 | Failed Tests | per failure: ID, title, page/workflow, QA failure category, short explanation, retry history, healing result, final status, evidence available. **No stack-trace dumps** — reference the artifact |
| 11 | Evidence Summary | screenshots, traces, videos, network captures, raw counts, with references |
| 12 | Application Exploration Coverage | page/route/workflow/test-execution coverage, and what was **not** covered |
| 12b | Accessibility Observations | measured un-named controls (count + routes), label coverage ratio per route, pointer-interactive-but-not-keyboard-reachable controls. **QA findings, in QA categories — no severity score, no security framing** (`site-agnostic-discovery.md`). Where the target names every control, say so |
| 12c | Plan Review | Phase 4A tripwire flags with their numbers, or `UNREVIEWED` stated plainly (`coverage-ledger.md`) |
| 13 | QA Framework Confidence | confidence in discovery, application model, workflow model, generated tests, locators — with reasons. An unavailable term states its **distance from availability** (`evidence-and-confidence.md`), never a bare refusal |
| 13b | Write Surfaces & Mutation | per surface: id, classification (`SAFE_WRITE` · `STATE_CREATING` · `IRREVERSIBLE`), the control that gated it, whether it ran, and the **residue accounting**. For every **declined** irreversible surface, the record it *would* have created. For every one that **ran**, the records left permanently, with markers. `Records cleaned` never includes an irreversible record (`write-operations-and-test-data.md`) |
| 14 | Limitations & Disclosures | **mandatory** — what could not be explored, why, and the impact on the result. Includes **`NOT_EXERCISED` capability** (handling the target gave nothing to run) and, where any pipeline defect was found and fixed mid-run, that defect **derived from the run's own artifacts** |
| 15 | Overall QA Conclusion | plain-language takeaway for a QA engineer |

### Section detail requirements

**API & Network Discovery** must distinguish three reader-facing counts:

- discovered only: known from a spec, JavaScript, sitemap, robots, or crawl text,
  but not witnessed as a request;
- observed: witnessed in real browser/network traffic;
- exercised: driven by a generated test, with the test ids shown.

If an endpoint is exercised, the table says `EXERCISED` or includes an
`Exercised by` column with `TC-...` ids. It must not say "not exercised" in one
artifact and "exercised" in another. Any mismatch is a reporting defect.

**Discovered Workflows** must include, for each workflow:

- workflow id and name;
- purpose in one short sentence;
- starting page/route;
- step list or compact step summary;
- expected outcome and actual result;
- observed API activity, or `NOT_OBSERVED`;
- data changed, restored, or left as residue;
- evidence references;
- confidence/evidence state, or why unavailable.

**Evidence Summary** must say where each evidence type actually lives. If an
evidence directory is empty, explain why. Use "network records" when network
evidence is stored as JSON records outside `evidence/network/`; reserve "network
capture files" for actual files in the named capture directory.

**Accessibility Observations** must include a summary plus references for:

- un-named controls, with counts and affected routes or artifact path;
- label coverage by route or a linked machine-readable table;
- duplicate accessible names that block safe targeting;
- pointer-interactive but not keyboard-reachable controls, or `NOT_OBSERVED` if
  the measurement was not captured.

**Execution Results** must separate phases when needed. For example, "0 session
recoveries during final test execution" and "2 session recoveries during
authenticated discovery" are both valid, but reporting only "session recoveries:
0" while diagnostics says 2 is confusing and must be clarified.

### Narrative sections are derived, never authored

Any section describing **what happened in this run** — defects found, passes
superseded, engines skipped, gates triggered — MUST be generated from the artifacts
on disk. Fixed prose describing a previous run's events is a reporting-integrity
defect of the same class as an authored assertion (`Rule 4`).

> A report once carried hardcoded narrative describing two defects as "found during
> the run" that had in fact been fixed in an earlier development cycle, cited a
> superseded-pass directory that did not exist in that run, and **omitted the defect
> that had actually occurred**. It compiled, it read fluently, and it was false.

Concretely: count superseded passes by reading the directory; name skipped engines
from their recorded status; describe a defect from the validation and diagnostic
records that captured it. If the artifact is absent, the section says so — a
narrative is never carried over from a prior run, and never written from memory.

Example executive-summary voice:

> "The application was explored using authenticated browser automation. 47 pages
> and 83 routes were discovered. 142 QA test cases were generated and 137 were
> executed. 128 passed, 9 failed and 5 were blocked."

Example conclusion voice:

> "The exploration mapped the primary application workflows and generated a
> reusable Playwright suite. Most generated tests passed. Nine tests require
> review due to application behaviour or locator issues. Coverage was limited for
> the administrative section because the supplied account did not have access."

### The report must degrade gracefully

Sections whose data is unavailable state that plainly (`API capture: partial —
network instrumentation unavailable after minute 6`). A missing section is never
silently omitted, and never filled with a plausible guess.

---

Structured contracts (`final-report.json`, `summary.json`, `run.json`), retry/
healing reporting, per-test failure evidence, the README, and the acceptance
checklist: **`final-report-json.md`**.
