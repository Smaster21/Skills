# Final Report — `report/final-report.md`

> Executor reference. **Mandatory deliverable.** The report must be understandable
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
| 13 | QA Framework Confidence | confidence in discovery, application model, workflow model, generated tests, locators — with reasons |
| 14 | Limitations & Disclosures | **mandatory** — what could not be explored, why, and the impact on the result |
| 15 | Overall QA Conclusion | plain-language takeaway for a QA engineer |

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
