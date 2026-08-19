<!-- PRESERVED VERBATIM. This is the standalone execution entry point as it stood at
     the completion of W6 (SKILL.md sha256 ab2f8330...015065). During tool integration
     the top-level SKILL.md was restructured into a communitytools-compliant ROUTER.
     The full operational contract below is retained unchanged as deep reference:
     the coordinator reads the router; this file is the authoritative WHEN/evidence/
     governance detail an executor or reviewer consults. Apart from the RedOps Site
     Explorer scope migration — removal of security/ZAP/OWASP, accessibility/axe-core,
     performance/Lighthouse, responsive, and cross-browser (Firefox/WebKit) references;
     visual marked optional/disabled; Authenticated/Unauthenticated Exploration
     terminology — the operational contract below is retained unchanged. -->

---
name: qa-automation
description: Enterprise QA automation engineer for web apps — given any website URL, crawl and map the application, detect components, generate production-grade Playwright + TypeScript tests (smoke, functional, UI, forms, auth, navigation, API; dashboards and tables optional; visual optional/disabled initially), execute them on Chromium, retry and self-heal broken locators, capture full failure diagnostics, and produce HTML/Markdown/JSON/JUnit reports with AI root-cause analysis. Use when asked to build a Playwright test suite, automate E2E/regression testing, set up a Page Object Model framework, or generate tests for a web application.
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
| [01_Master_Architecture.md](docs/01_Master_Architecture.md) | **Highest architectural authority.** Vision, principles, components, lifecycle (§16), state machine (§17), gates (§15, §26), ownership matrix reference (§37), engine registry (§38), cache service (§39), artifact separation (§40), dependency architecture (§41), governance (§42) |
| [Architecture_Ownership_Matrix.md](docs/Architecture_Ownership_Matrix.md) | **Governance contract.** 37 capabilities → one owner each · 14 canonical datasets → one producer each · cache regions · decision, coverage, risk, confidence, scheduling ownership |

## Required Engines — Normative

Absence of any of these is a hard failure.

| # | Document | Owns |
|---|---|---|
| 02 | [02_Decision_Engine.md](docs/02_Decision_Engine.md) | **Sole decision authority.** Engineering decisions · Risk · Framework Confidence · Policy · Conflict resolution |
| 03 | [03_Discovery_Engine.md](docs/03_Discovery_Engine.md) | Discovery · Runtime discovery |
| 04 | [04_Knowledge_Graph.md](docs/04_Knowledge_Graph.md) | **Canonical application model.** Knowledge Graph · Graph versioning · Graph diffing |
| 05 | [05_Test_Planning_Engine.md](docs/05_Test_Planning_Engine.md) | Test Plan · Test strategy · **Canonical coverage** · Dependency planning · Resource planning · Retry policy |
| 06 | [06_Test_Generation_Engine.md](docs/06_Test_Generation_Engine.md) | Test generation · Duplicate detection · Incremental generation |
| 07 | [07_Execution_Engine.md](docs/07_Execution_Engine.md) | **Sole runtime authority.** Execution · Runtime Schedule · Browser management · Worker scheduling · Timeout management |
| 09 | [09_Reporting_Analytics.md](docs/09_Reporting_Analytics.md) | Reporting · Analytics |

## Optional Engines — Normative When Present

Absence changes **cost or assurance, never correctness** (`01` §16).

| # | Document | Owns | On absence |
|---|---|---|---|
| 08 | [08_Self_Healing_Engine.md](docs/08_Self_Healing_Engine.md) | Self-healing · Locator evolution · Healing Candidate Score *(engine-local)* | No healing; failures reported |
| 10 | [10_AI_Learning_Repository.md](docs/10_AI_Learning_Repository.md) | Learning Database | No historical evidence |
| 11 | [11_Unified_Test_Intelligence_Engine.md](docs/11_Unified_Test_Intelligence_Engine.md) | Unified Intelligence Projection *(derived)* · **Framework Cache Service** | Consume `04` directly |
| 12 | [12_Adaptive_Execution_Optimizer.md](docs/12_Adaptive_Execution_Optimizer.md) | Execution Optimization Proposal *(advisory)* · Batch planning · Runtime prediction | `07` executes plan order |
| 13 | [13_Incremental_Discovery_Engine.md](docs/13_Incremental_Discovery_Engine.md) | Discovery Delta · Fingerprinting | Full discovery |
| 14 | [14_Confidence_Coverage_Optimizer.md](docs/14_Confidence_Coverage_Optimizer.md) | Coverage Recommendations · Stop-condition recommendation | Full plan executes |
| 15 | [15_AI_Planning_Review_Engine.md](docs/15_AI_Planning_Review_Engine.md) | Planning Review *(advisory)* | Plan unreviewed, flagged |

## Supporting Documents — Non-Normative Operational Playbooks

Neither is an engine. Neither adds a state, a phase, or any ownership. The Engine
Registry remains **01–15**.

| Document | Layer | Answers |
|---|---|---|
| [16_QA_Execution_Playbook.md](docs/16_QA_Execution_Playbook.md) | Operational QA doctrine | **WHAT** good QA behaviour looks like — environment intelligence, workflow-first validation, evidence standards, runtime adaptation, reporting integrity |
| [IMPLEMENTATION_PLAYBOOK.md](docs/IMPLEMENTATION_PLAYBOOK.md) | Playwright implementation | **HOW** to build it — scaffold, config, discovery tactics, retry mechanics, hard-won rules, manual mode (§17), health pre-flight (§18) |

Consult both for judgement and tactics. Neither for authority.

---

# 2. Precedence

```
01_Master_Architecture              ← highest architectural authority
        ↓                             architecture, principles, lifecycle
Architecture_Ownership_Matrix       ← governance contract
        ↓                             who owns what
02 … 15                             ← engine specifications
        ↓
16_QA_Execution_Playbook            ← non-normative: operational doctrine (WHAT)
IMPLEMENTATION_PLAYBOOK             ← non-normative: implementation (HOW)
        ↓
SKILL.md                            ← orchestration only (WHEN)
```

- `01` SHALL take precedence over every other document.
- The **Matrix** governs *ownership allocation only*. It does not define architecture.
- Where an engine document conflicts with `01`, **`01` wins**.
- Where an engine document conflicts with the Matrix on *who owns what*, **the Matrix wins** until `01` is amended.
- Where the Matrix conflicts with an architectural principle in `01`, **`01` wins** and the Matrix is defective.
- Where two engine documents conflict, **the lower number wins**.
- Neither playbook overrides a numbered document. If one appears to, that playbook is defective.
- **This file overrides nothing.**
- No document authorises skipping a lifecycle stage or a mandatory quality gate.

---

# 3. Ownership Resolution

**This file never resolves ownership.** When ownership is unclear:

1. Read `Architecture_Ownership_Matrix.md` — 37 capabilities, 14 canonical datasets.
2. If the Matrix is silent or ambiguous, read `01` §37 and §42.
3. If both are silent, **stop and report**. Do not infer an owner.

## Two Authorities Only

```
02 Decision Engine   →  decides WHAT happens
07 Execution Engine  →  decides HOW it runs at runtime
```

Every other engine produces knowledge, plans, artifacts, or recommendations.
None decides. None executes.

## Routing Table — Every Operation Has One Destination

| Operation | Routes to |
|---|---|
| Any decision · risk · Framework Confidence · policy · conflict | **02** |
| Any knowledge operation · graph read, version, diff | **04** |
| Any planning operation · canonical coverage · dependencies · retry policy | **05** |
| Any generation operation · regeneration scope · duplicate detection | **06** |
| Any runtime execution · scheduling · browsers · workers · timeouts | **07** |
| Any healing operation · locator history | **08** |
| Any reporting operation · analytics | **09** |
| Any learning operation · historical evidence | **10** |
| Cache storage, eviction, TTL *(region semantics stay with the region owner)* | **11** |
| Optimization advice · batch plan · runtime prediction | **12** |
| Change detection · fingerprinting | **13** |
| Coverage optimization · stop recommendation | **14** |
| Plan review | **15** |
| Operational judgement — *how should a QA engineer behave here?* | **16** |
| Implementation tactics — *how do I write this in Playwright?* | **PLAYBOOK** |
