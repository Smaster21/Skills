---
name: qa-automation
description: Autonomous web-app QA — discover, plan, generate, run Playwright tests
---

# QA Automation

Autonomous, evidence-driven QA engineering for web applications. Given an
authorized URL it understands the app, generates a Playwright + TypeScript suite
from verified locators, executes it, diagnoses failures, records every decision
with measured confidence, and reports honestly what was and was not covered.

> **This `SKILL.md` is a ROUTER** — when to mount the skill and which reference
> to hand an executor. Executor playbooks live in `reference/`; the authoritative
> specification lives in `docs/`.

## Purpose

- **Problem solved:** turns an authorized web application into a running,
  self-healing automated test suite with traceable evidence — without a human
  hand-writing locators, plans, or assertions.
- **Coverage:** smoke, functional (positive, boundary, business-rule,
  CRUD-except-Delete, state-transition), UI, forms and **multi-step business
  flows** (populate → submit → assert the state moved — `workflow-flows.md`),
  authentication, navigation, API. Dashboard/table optional; visual disabled.
- **Target type:** web applications (any framework), in **Unauthenticated
  Exploration** and, when credentials are supplied, **Authenticated Exploration**.
- **Crawl behaviour:** exhaustive, and there is no other mode. **Every discovered
  route instance** is crawled within the declared `MAX_ROUTE_INSTANCES` ceiling —
  no sampling, no convergence early-stop, no mode switch. Depth grants no authority:
  it relaxes no guard and still emits QA-only output
  (`reference/scope-enforcement.md`).
- **Mount it when:** verifying / regression-testing / building an E2E suite for an
  in-scope web app, or as the defensive counterpart after recon maps a surface.
  **Defensive only** — it performs no security testing.

A run that emits scripts without discovery, decisions, measured evidence, and
disclosed scope has not done the job.

## Phases

Routing map over the 21-state machine. Each phase's authoritative specification
is its owning `docs/` document. Phase behaviour and gates:
**`reference/execution-lifecycle.md`**.

| # | Phase | Owner | Routing note |
|---|---|---|---|
| 0 | CONFIGURING | `01` §29 | Config, Execution + Correlation ID, health pre-flight (`PLAYBOOK` §18) |
| 0A | DETECTING_CHANGES *(opt)* | `13` | Skip → full discovery |
| 0B | PROFILING | `03` §18 | Measure the target's conventions (`site-agnostic-discovery.md`) |
| 1 | DISCOVERING | `03` | Depth-first, exhaustive, read-only; expands nav disclosures; no caps |
| 2 | BUILDING_GRAPH | `04` | Canonical model; an invalid graph is not persisted |
| 2A | INTELLIGENCE *(opt)* | `11` | Skip → consume `04` directly |
| 3 | Locator Verification | `03` §29 → `06` §26 | Probe every candidate; only verified locators reach generation |
| 4 | PLANNING | `05` | Coverage, workflows, risk, browser matrix |
| 4A | PLANNING_REVIEW *(opt)* | `15` | Advisory; skip → proceed flagged |
| 5 | GENERATING | `06` | Page Objects, fixtures, specs |
| 6 | VALIDATING | `01` §18 · `06` §38 | TypeScript + provenance + negative control; FAIL stops |
| 6A | OPTIMIZATION *(opt)* | `12` | Skip → plan ordering |
| 7 | EXECUTING | `07` | Sole runtime authority |
| 8 | RETRYING / SELF_HEALING *(opt)* | `08` under `02` | Classify first; healing for locator drift only |
| 9 | DIAGNOSTICS | `01` §22–23 | Failures → evidence-backed root cause |
| 10 | REPORTING | `09` | One model → every format; gaps disclosed |
| 11 | LEARNING *(opt)* | `10` | Recommends only; `02` decides |

## Output

Writes into **`./output/qa/`** only; run isolation is at the `./output/` boundary.
The output is a contract: every file or directory named in the README, summary,
or report must either exist on disk or be reported as `NOT_PRODUCED` / `EMPTY`
with a reason and impact.

| Category | Holds |
|---|---|
| `run.json` · `summary.json` · `README.md` | execution metadata, compact result, directory guide |
| `report/` | `final-report.md` + `final-report.json` |
| `discovery/` · `network/` · `knowledge/` | application contents, workflow inputs (`transitions.json`, `affordances.json`), API activity, learned model |
| `planning/` · `tests/` · `execution/` | plan, generated suite + `catalogue.json`, results |
| `evidence/` · `coverage/` · `diagnostics/` | artifacts, QA coverage, retries and healing |
| `raw/` | low-level event streams |

One artifact, one category. Stable IDs join *page → workflow → test → execution →
evidence*, reusing AIC identifiers verbatim where they exist. Contracts:
`reference/output-and-scope.md`, `final-report-md.md`, `final-report-json.md`.
Before reporting, API exercised state is reconciled from `tests/catalogue.json`
back into `network/api-inventory.json`, Gate B2 sensitivity is written to
`execution/negative-control-sensitivity.json` or explicitly marked
`NOT_PRODUCED`, and `evidence/evidence-inventory.json` explains every empty
evidence directory.

## Tools

`node` (≥20) · `npm` · `npx playwright` · `chromium` (Playwright-managed build,
installed without `install-deps` and without `sudo`) · `jq`. Install:
`npm i -D @playwright/test typescript` then `npx playwright install chromium`.

## Related Skills

`/reconnaissance` (run first — seeds discovery) · `/techstack-identification`
(fingerprint before planning) · `/api-security` (pairs with discovered APIs) ·
`/authentication` (login/session in scope) · `/regression-sweep` (downstream
re-validation).

## Rules

1. **Authorized target only.** Missing or ambiguous authorization → **STOP,
   report `BLOCKED`**. The target is always supplied, never discovered, invented,
   substituted, or carried forward from a prior run.
2. **Scope before action.** Validate every target against `SCOPE_FILE` (if
   provided) before any navigation; abort out-of-scope with a clear message and
   no output (`reference/scope-enforcement.md`).
3. **Read-only discovery.** Discovery navigates and opens navigation disclosures.
   Form submission, destructive/write-intent controls, data deletion, account
   creation, payments, authentication bypass, and unmasked secrets all stay
   outside its boundary. The crawl catalogues a
   state-changing affordance; it never exercises one. Discovering an endpoint is
   not a reason to call it.
4. **Evidence is observed, and so is every expectation.** Evidence Objects come
   from real execution and each carries a *measured* confidence. The same
   discipline binds what the suite is built from: every assertion cites the
   measurement it derives from and the scope that measurement is valid in
   (`reference/suite-self-validation.md` Gate A). An authored expectation is the
   same defect as an authored confidence value.
5. **One confidence model.** `02` §21 is the sole Framework Confidence authority;
   its weights are fixed. No second confidence or Evidence Quality model exists.
6. **Unavailable is a state, not a number.** An unavailable measurement is named
   as such — `0`, `50`, `100`, a default, a prior value, and an inference are all
   invalid substitutes.
7. **Chromium-only unless the browser matrix authorizes more.** A
   declared-but-unrun browser project misrepresents scope.
8. **Report honestly.** Reconcile every number against the runner's output;
   separate target defects from suite defects from environment artifacts; a
   partial run is reported as partial.
9. **Preserve prior artifacts.** A previous execution's output is immutable
   evidence; a new run writes a new boundary.
10. **Non-interactive.** All context via env vars; sensible defaults; clean up
    processes and temp files on exit; deterministic output for identical input.

Rules **11–25** extend these with equal force — target-agnostic locators,
coverage ledger, intent binding, workflow form coverage, auth-state isolation,
measured-response assertions, evidence-backed classification, session resilience,
per-route baselines, write gating, QA-only output, stable IDs, provenance,
healing honesty, artifact immutability: **`reference/rules-extended.md`**.

## Reference

Executor playbooks. Tier 0 loads for the whole run; Tier 1 loads per phase — the
**loading manifest is in `reference/INDEX.md`**. `SKILL.md` is never passed to an
executor.

| File | Read it when |
|---|---|
| `quickstart.md` · `discovery-profiling.md` | running end to end; profiling a target |
| `rules-extended.md` · `execution-lifecycle.md` | Rules 11–25; phase behaviour and gates |
| `site-agnostic-discovery.md` | discovery, locators, interaction — on any site |
| `suite-self-validation.md` · `coverage-ledger.md` | proving the suite sound (Gates A/A2/A3, B/B2, C); accounting every artifact |
| `known-failure-modes.md` | before building any phase — eleven recorded defects |
| `w7-api-evidence-contract.md` | anything touching API/network output |
| `workflow-flows.md` | the target has multi-step flows |
| `write-operations-and-test-data.md` | before planning a write |
| `output-and-scope.md` · `scope-enforcement.md` | output tree, env contract, scope checks |
| `final-report-md.md` · `final-report-json.md` | producing the deliverable |
| `evidence-and-confidence.md` | the evidence → confidence chain |
| `execution-entry-point-01.md` … `-07.md` | full operational contract (deep reference) |
| `INDEX.md` · `../docs/` | **loading manifest** + navigator; the 18-document spec |
