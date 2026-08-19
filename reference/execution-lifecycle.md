# Execution Lifecycle — phase narrative

> Executor reference. The routing table lives in `SKILL.md`; this file carries
> the behaviour that governs how the phases run. Read it before entering
> DISCOVERING, and again before REPORTING.

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
