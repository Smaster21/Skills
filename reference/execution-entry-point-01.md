<!-- ============================================================================
     W8 SUPERSESSION ERRATA — prepended 2026-08-17.
     Everything BELOW the preservation marker is unchanged, byte for byte.
     This block is the only addition to this file.
     ============================================================================ -->

# W8 SUPERSESSION ERRATA

> **Read this before the preserved body below.** The body is the W6 execution
> entry point, retained verbatim as historical reference. Parts of it were
> superseded by the **W8 Site Explorer boundary** update. Where this errata and
> the preserved body disagree, **this errata wins** — and the current owning
> document wins over both.

## What this skill is now

The backend methodology for the **RedOps Site Explorer**:

```
URL → scope/authorization validation → browser-based exploration
    → application understanding → evidence/provenance collection
    → QA test planning → QA test generation → Playwright validation
    → Playwright execution → retry/diagnostics/self-healing
    → Site Explorer result
```

Security assessment is **not** part of this skill. A separately selected
Skillmatrix security skill reads this skill's evidence and determines its own
methodology.

## Superseded statements in the preserved body

| Location | Preserved text | Superseded by |
|---|---|---|
| Frontmatter `description` (line ~10) | Advertises "accessibility via axe-core, visual, responsive, cross-browser, Lighthouse performance, tiered security via ZAP passive proxy + OWASP Top 10 mapping" | **All of it removed in W8.** The current description is in `SKILL.md`. No axe-core, no Lighthouse, no ZAP, no OWASP mapping, no responsive or cross-browser matrices |
| §1 Document Registry — `IMPLEMENTATION_PLAYBOOK` row | "…retry mechanics, **ZAP tiers**, hard-won rules…" | `PLAYBOOK` §9 (ZAP tiers), §10 (OWASP mapping) and §11 (performance budgets) are **REMOVED — W8**. `PLAYBOOK` §5 is now the ten QA categories; §21 owns deep discovery, API capture, bounded exploration, dedup and exploration modes |
| §0 Framework Identity — "plans … and a **browser matrix**" | Implies a multi-browser matrix | **Chromium only** (`01` §2, `05` §27, `07`, `16` §32) |
| §6 Failure Handling / §8 — "security vulnerabilities", "…auth, **security**, data corruption → real defects" | Treats security defects as a QA failure class | A failure is a **target defect, suite defect, or environment artifact** (`08` §16, `01` §23). It is NEVER a security finding and never earns a severity or CVSS |
| **§8 Non-Goals** — "Security validation is a **secondary, opt-in** goal — passive by default. Active security scanning requires explicit authorization and a dedicated test environment (Playbook §9)" | Security validation is opt-in | **Superseded.** Security testing is **not a qa-automation responsibility at any tier**. It is not gated, opt-in, or configurable — the capability does not exist (`01` §2). `SECURITY_SCAN` and `ZAP_API_KEY` no longer exist |

## Added invariants (not in the preserved body)

These are now in force and belong with §7 Invariants / §9 Governance:

1. **No security interpretation.** Record factual observations only — an input
   exists, a form exists, a route exists, an API endpoint exists, a JavaScript
   or GraphQL route was discovered, an authentication surface exists, a
   parameter exists, a request/response was observed. NEVER conclude that an
   attack class applies, that a surface is vulnerable, that a security skill
   should be run, or that any of it is probable. NEVER emit a severity, CVSS,
   finding, applicability tag, routing suggestion, or filtered security test
   plan.
2. **Reserved states are never emitted.** The ladder is
   `DISCOVERED → OBSERVED → EXERCISED → VALIDATED`. `OFFENSIVELY_VALIDATED` and
   `VULNERABILITY_CONFIRMED` belong to the downstream security skill and
   validator, and are un-emittable here.
3. **Bounded exploration.** Never claim 100% or "every page". The guarantee is
   *all reachable, in-scope surfaces within the configured exploration budget*.
   Every discovered-but-not-reached surface carries exactly one reason:
   `inaccessible · blocked · capped · excluded · unavailable state ·
   unavailable credentials` (`01` §2.2, `09` §12A).
4. **Deterministic identity, no re-crawl.** A surface is revisited only for a
   recorded reason — distinct state, distinct authentication context, workflow
   transition, validation, or explicitly authorized re-discovery
   (`PLAYBOOK` §21).
5. **Exploration mode is not authorization.** **Authenticated Exploration**
   (credentials supplied) and **Unauthenticated Exploration** (none) both
   require approved scope and RoE (`01` §2.3, `01` §30).
6. **Ten QA categories only** — smoke · functional · UI · forms ·
   authentication · navigation · API, plus dashboard / table / visual
   **OPTIONAL — DISABLED BY DEFAULT**. Functional covers positive · boundary ·
   business-rule · **CRUD except Delete** · state-transition. Delete affordances
   are discovered, never exercised (`PLAYBOOK` §5, `16` §44).
7. **`attackSurface` is permanently `NOT_PRODUCED`.** Wave W7-E is not
   authorized for qa-automation (Ownership Matrix, W8 / C1).

## Still fully in force in the preserved body

The 21-state machine · the 16 phases · §3 ownership resolution · §5 phase
dispatch · **§5A Evidence → Evidence Quality → Framework Confidence** (`02` §21
sole authority, weights unchanged) · §5B artifact ownership and identity ·
§7 invariants · §7A validation tiers · §9 authorization-first, artifact
preservation, ownership preservation, "introduce nothing new", and the
`NOT DEMONSTRATED` / `BLOCKED` / `NOT PRODUCED` reporting vocabulary.

---

<!-- PRESERVED VERBATIM. This is the standalone execution entry point as it stood at
     the completion of W6 (SKILL.md sha256 ab2f8330...015065). During tool integration
     the top-level SKILL.md was restructured into a communitytools-compliant ROUTER.
     The full operational contract below is retained unchanged as deep reference:
     the coordinator reads the router; this file is the authoritative WHEN/evidence/
     governance detail an executor or reviewer consults. Apart from the RedOps Site
     Explorer scope migration — removal of security/ZAP/OWASP, accessibility/axe-core,
     performance/Lighthouse, responsive, and cross-browser (Firefox/WebKit) references;
     visual marked optional/disabled; Authenticated/Unauthenticated Exploration
     terminology — the operational contract below is retained unchanged.
     One factual repair: 18 links to docs/ were written relative to the skill root
     and did not resolve from reference/; they now read ../docs/. Target unchanged. -->

---
name: qa-automation
description: Enterprise QA automation engineer for web apps — given any website URL, crawl and map the application, detect components, generate production-grade Playwright + TypeScript tests (smoke, functional, forms, auth, navigation, tables, dashboards, API, accessibility via axe-core, visual, responsive, cross-browser, Lighthouse performance, tiered security via ZAP passive proxy + OWASP Top 10 mapping), execute them, retry and self-heal broken locators, capture full failure diagnostics, and produce HTML/Markdown/JSON/JUnit reports with AI root-cause analysis. Use when asked to build a Playwright test suite, automate E2E/regression testing, set up a Page Object Model framework, or generate tests for a web application.
---

# AI Enterprise QA Automation Engineer — Execution Entry Point

Architecture **v4.0 — FROZEN**. 15 engines, one governance contract, two playbooks.

## What this file is

This file is the **execution entry point**. It owns exactly one thing: **WHEN**.

```
WHO   →  01_Master_Architecture.md  +  Architecture_Ownership_Matrix.md
WHAT  →  16_QA_Execution_Playbook.md          (operational doctrine)
HOW   →  IMPLEMENTATION_PLAYBOOK.md           (Playwright tactics)
WHEN  →  SKILL.md                             (this file)
```

It owns no capability, produces no canonical dataset, and holds no authority.

**It SHALL NEVER** calculate risk, coverage, or confidence · decide anything ·
schedule runtime execution · plan, generate, heal, or report · resolve an
ownership question · override any document.

**It only routes.** Every substantive question is dispatched to its owner. If this
file appears to answer a specification or ownership question, this file is wrong
and the owning document wins.

Behave like an experienced Senior QA Automation Engineer, not a code generator
(`01` §1).

---

# 0. Framework Identity

This is an **AI-powered QA engineering framework** for understanding web
applications and deciding what to verify about them.

It:

- **understands** an application — crawls it, observes its runtime, classifies
  components, and builds a canonical application model;
- **decides** — evaluates evidence, rules, risk and policy before acting, and
  records every decision with its reasoning;
- **plans** — derives coverage, workflows, priorities and a browser matrix;
- **generates** — produces Playwright + TypeScript automation from verified
  locators, never from raw HTML;
- **executes** — orchestrates browsers, workers, contexts and timeouts;
- **diagnoses** — classifies failures and separates target defects from suite
  defects and from environment artifacts;
- **reports** — renders every format from one normalized model, disclosing what
  was not covered;
- **learns** — accumulates historical knowledge per application scope and
  recommends improvements for future executions.

Playwright is the **execution tool**, not the product. This framework is
therefore **not** a Playwright generator, a Selenium wrapper, or a test-script
generator. A run that emits scripts without evidence, decisions and disclosed
scope has not done the job.

The two authorities that make it a framework rather than a generator are `02`
(what happens) and `07` (how it runs). Everything else produces knowledge,
plans, artifacts or recommendations.

---

# 1. Document Registry

18 documents. All verified present. Load `01` first, then the Matrix, then
`02`→`16`.

## Governance — Normative

| Document | Role |
|---|---|
| [01_Master_Architecture.md](../docs/01_Master_Architecture.md) | **Highest architectural authority.** Vision, principles, components, lifecycle (§16), state machine (§17), gates (§15, §26), ownership matrix reference (§37), engine registry (§38), cache service (§39), artifact separation (§40), dependency architecture (§41), governance (§42) |
| [Architecture_Ownership_Matrix.md](../docs/Architecture_Ownership_Matrix.md) | **Governance contract.** 37 capabilities → one owner each · 14 canonical datasets → one producer each · cache regions · decision, coverage, risk, confidence, scheduling ownership |

## Required Engines — Normative

Absence of any of these is a hard failure.

| # | Document | Owns |
|---|---|---|
| 02 | [02_Decision_Engine.md](../docs/02_Decision_Engine.md) | **Sole decision authority.** Engineering decisions · Risk · Framework Confidence · Policy · Conflict resolution |
| 03 | [03_Discovery_Engine.md](../docs/03_Discovery_Engine.md) | Discovery · Runtime discovery |
| 04 | [04_Knowledge_Graph.md](../docs/04_Knowledge_Graph.md) | **Canonical application model.** Knowledge Graph · Graph versioning · Graph diffing |
| 05 | [05_Test_Planning_Engine.md](../docs/05_Test_Planning_Engine.md) | Test Plan · Test strategy · **Canonical coverage** · Dependency planning · Resource planning · Retry policy |
| 06 | [06_Test_Generation_Engine.md](../docs/06_Test_Generation_Engine.md) | Test generation · Duplicate detection · Incremental generation |
| 07 | [07_Execution_Engine.md](../docs/07_Execution_Engine.md) | **Sole runtime authority.** Execution · Runtime Schedule · Browser management · Worker scheduling · Timeout management |
| 09 | [09_Reporting_Analytics.md](../docs/09_Reporting_Analytics.md) | Reporting · Analytics |

---

<!-- nav -->
*part 1 of 7  ·  [part 2](execution-entry-point-02.md) →*  ·  [reference index](INDEX.md)
