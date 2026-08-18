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
     governance detail an executor or reviewer consults. Nothing here was edited. -->

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
| [IMPLEMENTATION_PLAYBOOK.md](docs/IMPLEMENTATION_PLAYBOOK.md) | Playwright implementation | **HOW** to build it — scaffold, config, discovery tactics, retry mechanics, ZAP tiers, hard-won rules, manual mode (§17), health pre-flight (§18) |

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

## Framework Confidence vs Engine-Local Scores

**02 owns the single Framework Confidence model.** An engine MAY compute an
internal domain score — Healing Candidate Score (`08`), Test Value Score (`14`),
Review Score (`15`). Such a score is **evidence submitted to 02**. It SHALL NEVER
be reported as confidence, override Framework Confidence, or justify a decision.

---

# 3A. QA Reasoning Layer

## What This Is

An **internal reasoning capability of this orchestrator**. It is the deliberate
thinking a Senior QA Engineer does *before* raising a request — so that what
reaches an owner is a considered proposal with evidence, not a reflex.

It runs **silently**, before major requests are routed.

## What This Is Not

| Not | Because |
|---|---|
| An engine | The Engine Registry remains **01–15** (`01` §38). No number is assigned. |
| A document | It lives here. Nothing is added to the registry. |
| A lifecycle phase | Phase dispatch (§5) is unchanged — still 16 phases. |
| A state | The state machine (§4) is unchanged — still 21 states. |
| An owner | It owns no capability and produces no canonical dataset. |
| A canonical artifact | Its output is transient input, absorbed into the receiving owner's evidence and audit record (§below). |

It changes **no** ownership, **no** authority, and **no** routing. Every request
still travels to the same owner it did before, through the §3 routing table.

---

## When It Runs

Before each of these requests is routed — never instead of routing it:

| Before | Package goes to | Which then routes to |
|---|---|---|
| Discovery decisions | **02** | 03 / 13 |
| Planning decisions | **02** | 05 |
| Test prioritization | **02** | 05, with 14's input |
| Test generation requests | **02** | 05 → 15 → 06 |
| Runtime re-planning requests | **02** | 05 → 15 → 06 → 07 |
| Healing requests | **02** | 08 |
| Reporting conclusions | **09** | — |
| Learning recommendations | **02** | 10 |

Note that no arrow terminates at the reasoning layer. It is always upstream of a
decision, never a substitute for one.

---

## What It Reasons About

**Objective**

• Business objective · • User objective · • Functional objective
• Risk reduction *(risk values consumed from `02`)*

**Evidence**

• Evidence required · • Existing evidence · • Missing evidence
• Assumptions, and whether each is verifiable

**Hypotheses**

• Failure hypotheses — what would break, and how it would present
• Alternative validation paths — cheaper or stronger ways to obtain the same proof

**Economics**

• Cost versus value · • Expected confidence improvement
• Expected coverage improvement

---

## What It SHALL NEVER Do

It SHALL NEVER own decisions, planning, execution, healing, reporting, or
learning.

It SHALL NEVER calculate **Framework Confidence** or **Framework Risk**.

It SHALL NEVER bypass the Decision Engine.

Every value it reasons with is **consumed from its owner**:

| Value | Consumed from | Contract |
|---|---|---|
| Framework Risk | **02** | §20 Risk Engine |
| Framework Confidence | **02** | §21 Confidence Engine |
| Canonical coverage | **05** | §18 Coverage Analysis |
| Coverage gaps, test value | **14** | Coverage Recommendations |
| Execution cost, runtime prediction | **12** | §26 Runtime Predictor |
| Application model | **04** | Graph query contract |
| Historical evidence | **10** / **09** | Learning and analytics contracts |
| Duplicate detection | **06** | §32 |

Where a required value is unavailable, the layer SHALL state the gap. It SHALL
NOT substitute an estimate of its own for an owner's value.

---

## Output — The Reasoning Package

Every recommendation SHALL contain exactly these five fields:

| Field | Content |
|---|---|
| **Reasoning** | Why this request, in terms of the objective it serves |
| **Supporting evidence** | Concrete observations, cited by source and owner |
| **Uncertainty** | What remains unknown, and what would resolve it |
| **Expected confidence improvement** | Estimated, as evidence for `02` — never a Framework Confidence value |
| **Expected execution cost** | Consumed from `12` where available; otherwise stated as unavailable |

A package with an empty evidence field SHALL NOT be submitted.

### Not A Canonical Artifact

The package is **transient**. It is submitted to the receiving owner and absorbed
into that owner's existing evidence and audit record — `02` §36 Evidence Model
and §39 Decision Audit for decisions, `09` for reporting conclusions.

It is **not** persisted as a dataset of its own, appears in no Canonical Data
Ownership row, and creates no new artifact. The Matrix is unchanged.

### Estimates Are Evidence, Not Values

The two estimated fields follow the engine-local score precedent already
established for `08`, `14`, and `15`:

- They are **evidence submitted to an owner**
- They SHALL NEVER be reported as Framework Confidence or Framework Risk
- They SHALL NEVER override an owner's value
- Where an estimate and an owner's value disagree, **the owner's value prevails**

---

## Visibility

The layer is **invisible by default**. It produces no user-facing output.

Its reasoning becomes visible only when verbose or debug output is enabled,
resolved through the configuration hierarchy owned by the Configuration Layer
(`01` §29). This orchestrator defines no configuration option of its own.

When visible, reasoning SHALL be shown as the package's five fields — never as
narration, and never as a substitute for the owner's recorded decision.

---

## Determinism

Identical inputs SHALL produce an identical package (`01` §3.1).

The layer SHALL NOT introduce randomness, wall-clock dependence, or
completion-order dependence. Where it reasons during execution, it does so at the
same deterministic checkpoints `14` uses (§5, Phase 7) — never per test
completion.

## Failure Mode

Where evidence is insufficient to reason, the layer SHALL **refuse to
recommend** — consistent with `02` §5.5 Safe Failure.

It SHALL then route the underlying request with the evidence gap stated, so the
owner decides on a known-incomplete picture rather than a fabricated one.

Refusing to recommend SHALL NEVER block a request from reaching its owner.

---

# 4. Execution State Machine

21 states, per `01` §17. Maintain and log explicit state. Every transition,
including every skip, SHALL be logged with its reason.

```
INITIALIZING
     ↓
CONFIGURING
     ↓
DETECTING_CHANGES °········┐
     ↓                     │
DISCOVERING ◄──────────────┘
     ↓
BUILDING_GRAPH
     ↓
INTELLIGENCE °·············┐
     ↓                     │
PLANNING ◄─────────────────┘
     ↓
PLANNING_REVIEW °··········┐
     ↓  ╰─ revise ─► PLANNING   (02-approved only, capped by 15 §28A)
GENERATING ◄───────────────┘
     ↓
VALIDATING
     ↓
OPTIMIZATION °·············┐
     ↓                     │
EXECUTING ◄────────────────┘
     ↓  ├─► RETRYING ──────┐   (02 approves)
     ↓  └─► SELF_HEALING ──┤   (02 approves)
     ↓ ◄───────────────────┘
COLLECTING_ARTIFACTS
     ↓
DIAGNOSTICS
     ↓
REPORTING
     ↓
LEARNING
     ↓
COMPLETED

°  skippable — transitions directly to its successor when its engine is
   absent or disabled.
```

Terminal failure states: **`FAILED`**, **`CANCELLED`**.

## Skip Rules

| State | Owner | Skips to |
|---|---|---|
| `DETECTING_CHANGES` | 13 | `DISCOVERING` |
| `INTELLIGENCE` | 11 | `PLANNING` |
| `PLANNING_REVIEW` | 15 | `GENERATING` |
| `OPTIMIZATION` | 12 | `EXECUTING` |

A skipped state SHALL be logged with its reason and reported by `09`.
No other state is skippable.

`14` is **not** a state — it evaluates at deterministic checkpoints *within*
`EXECUTING`.

## Rollback Rules

- Forward-only. A module may **never** transition backwards.
- **One exception:** `PLANNING_REVIEW → PLANNING`, permitted only on a
  **02-approved** revision decision, bounded by `15`'s iteration cap. On cap
  exhaustion, `15` escalates to `02`; the plan proceeds with findings disclosed
  by `09`, never silently.
- Rollback affects framework state only. It SHALL NEVER undo browser actions
  already executed (`02` §32).

## Backward Compatibility

Skipping all four `°` states traverses exactly the v3.0 sequence, in v3.0 order.

---

# 4A. Execution Flow — End To End

The same 21 states and 16 phases, read as one causal chain. **This section adds no
state and no phase** — it is a reading aid for the model above and §5 below.

```
Request + authorized target
        ↓                       ← authorization verified first (§9); missing → STOP
Scope + governance validation    01 §30.1 scope identity · 01 §29 configuration
        ↓
Discovery                        03 · Phase 1
        ↓
Application / knowledge model    04 · Phase 2
        ↓
Locator verification             03 §29 → 06 §26 · Phase 3
        ↓
Risk + test planning             05 · Phase 4   (risk consumed from 02)
        ↓
Test generation                  06 · Phase 5
        ↓
Suite validation                 01 §18 · 06 §38 · Phase 6
        ↓
Execution orchestration          07 — sole runtime authority · Phase 7
        ↓
Browser / tool execution          07 §26, §27
        ↓
Evidence collection              02 §18 · Phase 9
        ↓
Per-item evidence confidence     PLAYBOOK §19
        ↓
Evidence Quality                 PLAYBOOK §19 → 02 §21's 0.35 term
        ↓
Decision evaluation              02 §15 pipeline
        ↓
Framework Confidence             02 §21 — sole authority
        ↓
Diagnostics + reporting          01 §23 · 09 · Phase 10
        ↓
Learning                         10 · Phase 11
        ↓
Next execution's decisions       10 §7.5 pinned snapshot
```

Evidence flows **forward** into decisions and **across executions** into learning.
An execution never reads history it is concurrently writing (`10` §7.5), and
same-execution dependencies flow downward only (`01` §41).

---

# 5. Phase Dispatch

16 phases, per `01` §16. Read the owning document **before** entering its phase.

### Phase 0 — CONFIGURING *(required)*
**Owner:** Configuration Layer · `01` §29, §7
Resolve configuration by priority — CLI → env → project → default. Register
capabilities. Assign Execution ID and Correlation ID. Initialize the **Framework
Cache Service** (`01` §39 — available from here onward, not from `11`'s phase).
**Playbook:** §1 scaffold · §2 config · §18 health pre-flight
**Gate:** configuration valid; capabilities registered.

### Phase 0A — DETECTING_CHANGES *(optional — 13)*
Load previous snapshot → fingerprint → verify stability (`13` §26) → compare
live → emit **Discovery Delta** to `04`, **Regeneration Request** to `06`.
**Emit:** snapshot, Discovery Delta · **Gate:** `13` §24
**Degrade:** no snapshot, unstable fingerprints, any failure → full discovery.
**Never:** mutate/version/diff the graph (`04`) · determine regeneration scope
(`06`) · implement cache storage (`11`) · calculate risk (`02`).

### Phase 1 — DISCOVERING *(required)*
**Owner:** `03`
Crawl, observe runtime, classify components, detect flows, validate, persist a
versioned snapshot. **Playbook:** §3 · **Gate:** `03` §35
**Constraint:** observational only (`03` §36). Never delete data, create
accounts, submit payments, bypass authentication, or store unmasked secrets.

### Phase 2 — BUILDING_GRAPH *(required)*
**Owner:** `04`
Normalize, deduplicate, resolve entities, create nodes and relationships with
evidence, verify integrity, version, diff. **Gate:** `04` §30
**Constraint:** an invalid graph SHALL NEVER be persisted.

### Phase 2A — INTELLIGENCE *(optional — 11)*
Project `04` into the reuse-optimized package; serve unchanged work from cache.
**Emit:** `unified-intelligence.json` stamped with **source graph version**
**Gate:** `11` §31 · **Degrade:** any failure → consume `04` directly.
**Constraint:** derived, **never canonical**. Where it disagrees with `04`, `04`
is correct.

### Phase 3 — Locator Verification *(required)*
**Owner:** `03` §29 → `06` §26
Probe every locator candidate; keep only those that resolve **and** return the
expected count. **Playbook:** §4 and the first two Hard-Won Rules in §15
**Gate:** no unverified locator reaches generation.

### Phase 4 — PLANNING *(required)*
**Owner:** `05`
Coverage analysis → workflow selection → risk prioritization (risk consumed from
`02`) → execution strategy → dependencies → resources → schedule → artifacts →
validate → approve. **Emit:** `test-plan.json` *(canonical)* + supporting plans
**Gate:** `01` §15 Intelligence Layer gate. Plan validation FAIL prevents
generation (`05` §32).
**Constraint:** the plan SHALL be executable with **no optimizer present**
(`05` §23A).

### Phase 4A — PLANNING_REVIEW *(optional — 15)*
Review coverage sufficiency (from `05`), effort-vs-risk proportionality (risk
from `02`), planned-strategy quality, cost, cross-engine consistency. Score and
submit **Planning Review** to `02`.
**Gate:** `15` §23 · iteration cap `15` §28A · **Degrade:** proceed unreviewed, flagged.
**Constraint:** **advisory only.** Never approves, rejects, or blocks. Never
calculates risk or confidence, computes coverage, performs plan validation, or
assesses generated artifacts — none exist yet.

### Phase 5 — GENERATING *(required)*
**Owner:** `06`
Select templates, expand workflows, generate Page Objects, components, fixtures,
utilities, tests, assertions; resolve locators and dependencies; assemble; stamp
traceability (`06` §40). **Playbook:** §5, §6, §7
**Constraint:** prohibited before plan approval. Direct HTML-to-test generation
prohibited.

### Phase 6 — VALIDATING *(required)*
**Owner:** `01` §18 · `06` §38
TypeScript compilation, Playwright syntax, imports, duplicates, circular
dependencies, missing assertions, invalid selectors, hardcoded waits, unsafe XPath.
**Gate:** WARNING may proceed. **FAIL stops** — poor automation never executes.

### Phase 6A — OPTIMIZATION *(optional — 12)*
Consume plan, `05`'s dependency plan and parallel candidacy, historical runtime.
Produce batch plan, runtime prediction, **Optimization Proposal**.
**Gate:** `12` §32 · **Degrade:** absent/failed/rejected → plan's own ordering.
**Constraint:** a **constraint-bounded permutation** — MAY reorder, batch, hint;
SHALL NEVER add, remove, or re-scope a test. Never schedules, allocates browsers
or workers, or sets timeouts.

### Phase 7 — EXECUTING *(required)*
**Owner:** `07` — **sole runtime authority**
**Validate the proposal first** (`07` §30A): reject unless its test set is
**exactly** the plan's test set. Rejected or absent → plan ordering. Log; never halt.
Then: build context, prepare environment, allocate resources, launch browsers,
isolate contexts, initialize fixtures, schedule workers, execute, capture.
**Emit:** `runtime-schedule.json` *(canonical)*, execution history, artifacts,
deviation record · **Gate:** `07` §40 · **Playbook:** §11, §12, §15, §17
**Constraint:** executes only — does not decide, retry autonomously, or heal.
No artifact owned elsewhere overrides it.

#### Deterministic Checkpoints within Phase 7 *(optional — 14)*
At **batch or workflow-group boundaries only** — never per test completion, which
is non-deterministic under parallel execution. Consume coverage from `05` and
confidence from `02`; rank gaps; score value; prioritize `06`'s redundancy
findings; submit stop-or-continue **recommendation** to `02`.
**Degrade:** absent, or no history on a first run → full plan executes (`14` §30).
**Constraint:** never stops execution, skips a test, calculates confidence, or
computes coverage. **Every skipped test and early stop SHALL be disclosed to `09`**
(`14` §29) — a skipped test is never passed and never coverage; an early-stopped
run is never complete.

### Phase 8 — RETRYING / SELF_HEALING *(optional — 08, under 02)*
Classify the failure first (`08` §16). Then analyse DOM, generate ranked
candidates, score, apply policy, request **02 approval**, apply, verify, audit.
**Playbook:** §8 mechanics only · **Limits:** `08` §32 — 3 healing attempts,
2 locator replacements, 1 wait adaptation, 1 browser restart.
**Retry ownership:** `05` plans · `02` approves · `07` enforces.
**Constraint:** score below 60 → reject. Never heal failed assertions, business
logic, backend defects, API failures, auth failures, security vulnerabilities, or
data corruption (`08` §13). A healed locator is a candidate; promotion needs
repeated success plus approval.

### Phase 9 — COLLECTING_ARTIFACTS / DIAGNOSTICS *(required)*
**Owner:** `01` §22, §23
Collect the planned artifact set, link each to Execution/Test/Workflow/Correlation
ID, then convert failures into root-cause findings with category, severity,
confidence *(from `02`)*, evidence, suggested fix, historical occurrence.
**Playbook:** §12 · **Constraint:** mask secrets before persistence.

### Phase 10 — REPORTING *(required)*
**Owner:** `09`
Collect, validate, normalize, aggregate, compute metrics, generate analytics,
render every format from one normalized model, archive, publish.
**Playbook:** §10, §13, §16 · **Gate:** `09` §33
**Constraint:** never fabricate, suppress, or alter a result. Disclose skipped
scope, degraded engines, and unexecuted work.

### Phase 11 — LEARNING *(optional — 10)*
Collect, validate, normalize, analyse, detect patterns, generate recommendations
with confidence *(from `02`)*, persist as immutable versions, publish.
**Constraint:** recommends only. Never modifies tests, Page Objects, or
configuration. `02` decides whether a recommendation applies.

### Throughout — Decision Audit
**Owner:** `02` §38 · **Emit:** `decision-history/decision-{NNNN}.json` —
inputs, evidence, candidates, rejected alternatives, winner, confidence, audit.
**Constraint:** persistence failure SHALL NEVER silently continue.

---

# 5A. Evidence, Evidence Quality, And Framework Confidence

This chain is **operationally mandatory**. A run that produces tests and reports
but not this chain has not satisfied the framework.

```
Real execution observation
        ↓
Evidence Object                  02 §18   (id · type · source · confidence · timestamp · payload)
        ↓
Per-item evidence confidence     PLAYBOOK §19
        ↓
Evidence Quality                 PLAYBOOK §19 → 02 §21's Evidence Quality term
        ↓
02 §21 Confidence Engine         + Historical Reliability · Rule Agreement · Environment Stability
        ↓
Framework Confidence             02 §21 — the only value reportable as "confidence"
        ↓
Decision · execution · report artifacts
```

## Evidence Objects

- Evidence Objects are produced from **real execution observations** — a live DOM
  read, a probe result, a measured latency, a persisted artifact.
- Evidence SHALL NEVER be fabricated, manually manufactured, or back-filled to
  satisfy a criterion (`01` §31).
- Specification text is **not** runtime evidence and SHALL NEVER be converted
  into it (§7A).
- Every applicable Evidence Object carries a **measured** `confidence`. A
  hand-authored confidence value is a defect, not evidence.
- Evidence with an empty required field is still evidence — it scores lower. It
  is never silently completed.

## Evidence Quality

- Evidence Quality is **derived from the measured Evidence Objects** of the
  decision it belongs to.
- It is **not a second confidence model**. It is one **input term** of `02` §21's
  existing model.
- The measurement procedure — completeness, directness, corroboration, rounding,
  clamping, aggregation — belongs to **`PLAYBOOK` §19**. Do not reimplement it
  here and do not compute it a second way.
- Empty evidence SHALL fail safely per `PLAYBOOK` §19 and `02` §5.5:
  `evidenceQuality` is **undefined** and the decision is refused
  `INSUFFICIENT_EVIDENCE`. It is never zero and never defaulted.

## Framework Confidence — `02` §21 is the sole authority

`02` §21 owns the framework's **single** confidence model. Its four terms and
their existing weights are:

| `02` §21 term | Weight | Measurement procedure |
|---|---:|---|
| Evidence Quality | 0.35 | `PLAYBOOK` §19 |
| Historical Reliability | 0.25 | `PLAYBOOK` §20.1 |
| Rule Agreement | 0.20 | `PLAYBOOK` §20.2 |
| Environment Stability | 0.20 | `PLAYBOOK` §20.3 |

These are **`02` §21's existing terms and weights — not a new model.** They sum
to 1.00 and SHALL NOT be reweighted, replaced, duplicated, or relocated.

- Every other component **supplies observations** to `02` through its existing
  contract. None of them computes confidence.
- No component may publish a competing Framework Confidence value. Engine-local
  scores (`08`, `14`, `15`) are evidence submitted to `02` (§3).
- Framework Confidence is computed **only when all four terms are available**.
- Where any term is unavailable, `02` **refuses** the value and **names every**
  unavailable term. A partial confidence score SHALL NEVER be produced.

## The three `PLAYBOOK` §20 term measurements

Surface only. The procedures are owned by `PLAYBOOK` §20 through `02` §21's
contract; read them there before relying on a value.

| Term | What it measures | What it is NOT |
|---|---|---|
| **Historical Reliability** (`§20.1`) | Framework **lifecycle** reliability across eligible prior executions of the current **Execution Scope Identity** (`01` §30.1), read from the pinned snapshot (`10` §7.5) | **Not test pass rate.** A target defect SHALL NEVER reduce framework reliability — finding real defects is success, not unreliability |
| **Rule Agreement** (`§20.2`) | How consistently the **applicable evaluated rules** already recorded in `02` §39 agree with the selected candidate | **Not a new rule system.** `02` §19 remains the only Rule Engine. `02` §19's `Confidence Modifier` SHALL NEVER be read — it would be a second path into confidence |
| **Environment Stability** (`§20.3`) | How nominally the target behaved during the **pre-execution** verification window (`07` §17, §31 · `PLAYBOOK` §18) | **Not during-execution health.** Feeding `07` §40's continuous health back in would let the outcome being scored influence the term that scores it |

## Unavailable measurements

An unavailable term is a **named state**, never a number:

| State | Term | Source |
|---|---|---|
| `NO_HISTORY` | Historical Reliability | `PLAYBOOK` §20.1 |
| `INSUFFICIENT_HISTORY` | Historical Reliability | `PLAYBOOK` §20.1 |
| `NO_APPLICABLE_RULES` | Rule Agreement | `PLAYBOOK` §20.2 |
| `RULE_OUTCOMES_NOT_RECORDED` | Rule Agreement | `PLAYBOOK` §20.2 |
| `NO_ENVIRONMENT_OBSERVATION` | Environment Stability | `PLAYBOOK` §20.3 |
| `NO_LATENCY_REFERENCE` | Environment Stability | `PLAYBOOK` §20.3 |
| `INSUFFICIENT_EVIDENCE` | Evidence Quality, and the decision itself | `PLAYBOOK` §19 · `02` §5.5 |
| `MISSING_TERM` | Framework Confidence, naming each absent term | `02` §21 · `02` §5.5 |

`PLAYBOOK` §20.1's D-3 additionally requires a refusal when `W` or
`minimumExecutions` is not resolvable through `01` §29; the implementation
records this as `NO_HISTORY_CONFIGURATION`.

**An unavailable measurement SHALL NEVER be replaced with `0`, `50`, `100`, a
configured default, a previous execution's value, or an inferred value.** A
cold-start scope with no history is a legitimate unavailable state, not an error,
and SHALL NEVER halt execution (`10` §7.3).

## Persistence — no new artifact

All three destinations already exist:

| Value | Recorded in | Owner |
|---|---|---|
| Per-item evidence confidence | the `confidence` field of the Evidence Object | `02` §18 |
| Evidence Quality | the decision's evidence record | `02` §36 |
| Framework Confidence, and each term's value or unavailable reason | the decision audit record | `02` §39 |

No dataset, file, or schema is created for this chain.

---

# 5B. Artifact Ownership And Identity

Artifacts persist under their **existing** owners. Never invent a destination,
and never claim ownership of another engine's artifact.

| Artifact | Owner | Canonical document |
|---|---|---|
| Discovery snapshot, component inventory, verified inventory | **03** | `03` §11, §29, §31 |
| Knowledge Graph, graph version, graph diff | **04** | `04` §30 |
| Test Plan, canonical coverage | **05** | `05` §32 |
| Generated tests, Page Objects, fixtures | **06** | `06` §30, §40 |
| Runtime Schedule, execution history, deviation record | **07** | `07` §39 |
| Locator History | **08** | `08` |
| Reports, analytics | **09** | `09` §33 |
| Learning Database | **10** | `10` §13, §7.1 |
| Decision history, evidence record, decision audit | **02** | `02` §36, §38, §39 |

## Identity — consumed, never redefined

| Identity | Owner | Used for |
|---|---|---|
| **Test Case Identity** | **06** §22.1 | Stable identity of a test across regeneration |
| **Execution ID / Correlation ID** | **01** §30 | One run, one causal chain |
| **Execution Scope Identity** | **01** §30.1 | *Which application* this run is about; the historical partition key for `10` |

This file, and every engine, **consumes** these identities. None generates,
redefines, or substitutes one, and no second identity system exists. `projectId`
from the test runner identifies a browser project and SHALL NEVER be used as a
scope identity (`01` §30.1).

## Event model

Components communicate through the **Event Bus** (`01` §27), which `07` §38
integrates with published and subscribed lifecycle events. Publish and subscribe
through that contract rather than invoking engines directly. This file defines no
event type; the Core Event set is `01` §27's.

## Learning

Learning is **downstream of real observations** (`10`). It accumulates per
Execution Scope Identity (`10` §7.2), is pinned immutable within an execution
(`10` §7.5), and treats a cold start as normal (`10` §7.3).

`10` **recommends only.** It SHALL NEVER fabricate an observation, become a
second decision authority, compute confidence, modify tests, Page Objects or
configuration, or introduce a lifecycle state. `02` decides whether a
recommendation applies.

---

# 6. Failure Handling

| Situation | Action |
|---|---|
| **Required engine fails** | **STOP.** Produce a diagnostic report. Never emit partial results as complete. |
| **Optional engine fails** | **DEGRADE, do not stop** (`01` §15). Apply the phase's degrade rule. Report the degradation. |
| **Mandatory gate FAIL** | Stop; diagnostic report. |
| **Gate WARNING** | Advance; record the finding. |
| **Mandatory input missing** (`BASE_URL`, environment, capability) | Stop at the gate and report. Never substitute a guess for evidence. |
| **Degraded mode** | Only `02` may approve. Log the reason; `09` reports it. |
| **Abort** | `FAILED` or `CANCELLED`. Preserve all collected evidence. |
| **Rollback** | Framework state only, forward-only except the 02-approved `PLANNING_REVIEW → PLANNING` loop. Never undo executed browser actions. |
| **Re-plan** | Observation → Evidence → **Re-plan Request** → `02` → `05` → `15` → `06` → Validation → Execution. Runtime SHALL NEVER generate tests directly. |
| **Recovery** | Classify first (`08` §16), then `02` approves. Preserve evidence across recovery. |
| **Environment interstitial** | Classified as an **environment issue**, ordered ahead of every other rule. Never scored as a regression (Playbook §15). |
| **Evidence set empty** | `evidenceQuality` undefined → refuse the decision `INSUFFICIENT_EVIDENCE` (`02` §5.5). Never zero, never defaulted (§5A). |
| **A `02` §21 term unavailable** | `02` refuses Framework Confidence with `MISSING_TERM`, naming every absent term. The decision still proceeds on rules, policy and risk; only the confidence value is refused (§5A). |
| **No history for this scope** | Cold start is normal (`10` §7.3). Historical Reliability is `NO_HISTORY`; execution SHALL NOT halt and the absence SHALL NOT be reported as a failure. |
| **Required measurement input missing** | Refuse with the named state. Never substitute `0`, `50`, `100`, a default, a previous value, or an inferred value. |

## Failure Classification

Classify **before** considering any recovery (`08` §16). The classification
determines what the failure means and whether healing is even permitted:

| Classification | Meaning | Healing |
|---|---|---|
| `LOCATOR` | The element exists; its address drifted | Permitted, under `02` approval and `08` §32 limits |
| `ASSERTION` | An expectation was not met — usually a **target defect** | **Prohibited** (`08` §13) |
| `ENVIRONMENT` | The target or infrastructure misbehaved | Prohibited; not a regression |
| Business logic · backend · auth · security · data corruption | Real defects | **Prohibited** (`08` §13) |

A target defect and a suite defect are different findings and SHALL be reported
separately (§9). Evidence SHALL be preserved across every recovery attempt and
every terminal state, including `FAILED` and `CANCELLED`.

## Degradation Ledger

Every degraded run SHALL state, in the report: which optional engines ran, which
were skipped, why, and what the skip cost in assurance. An optimized,
early-stopped, incrementally-discovered, or credential-limited run SHALL NEVER be
presented as complete.

---

# 7. Invariants

| Invariant | Source |
|---|---|
| **Deterministic** — identical inputs produce equivalent outputs | `01` §3.1 |
| **Observable** — every decision explainable; no silent decisions | `01` §3.2 |
| **Modular** — one responsibility per subsystem; no cross-module mutation | `01` §3.3 |
| **Safe by default** — destructive operations require explicit opt-in | `01` §3.5 |
| **Evidence before action** — insufficient evidence means refuse, not guess | `02` §5.2, §5.5 |
| **Immutable contracts** — versioned, with schema, timestamp, correlation ID, producer, consumer, validation status | `01` §14 |
| **Immutable snapshots** — never overwrite a discovery, graph, plan, report, or learning version | `03` §31, `04` §17, `05` §33, `09` §29, `10` §28 |
| **Traceable** — Execution ID, Correlation ID, framework/config/graph version on everything | `01` §30 |
| **Masked** — secrets never reach logs, reports, screenshots, traces, or AI prompts | `01` §30, `07` §41 |
| **Honest AI** — never invent facts, fabricate results, mark failures as passes, or suppress defects | `01` §31 |
| **Gates are hard** — required-engine FAIL stops; optional-engine FAIL degrades | `01` §15, §26 |
| **One owner per capability** — consumers never become owners; algorithms exist once | Matrix, `01` §42 |
| **Optional means removable** — absence degrades cost or assurance, never correctness | `01` §16, §42 |
| **Advisory means advisory** — 11, 12, 14, 15, 16 recommend; `02` decides; `07` executes | Matrix, `01` §42 |
| **Derived is never canonical** — `11`'s projection never displaces `04` | Matrix, `01` §37 |
| **One Framework Confidence** — owned by `02`; engine-local scores are evidence | Matrix, `01` §37 |
| **Runtime layering** — same-execution dependencies flow downward only; cross-execution historical reads are exempt and never block | `01` §41 |
| **Artifact ownership ≠ runtime authority** — `12` owns a proposal; `07` alone schedules | Matrix, `01` §40 |
| **Reasoning precedes routing** — the QA Reasoning Layer (§3A) reasons before a request is raised; it owns nothing, decides nothing, and persists nothing | §3A |
| **Evidence is observed, never authored** — Evidence Objects come from real execution observations; each applicable one carries a measured confidence | `02` §18, `PLAYBOOK` §19 |
| **One Evidence Quality measurement** — derived from measured Evidence Objects; an input term of `02` §21, not a second model | `PLAYBOOK` §19 |
| **Unavailable is a state, not a number** — no term is ever substituted with `0`, `50`, `100`, a default, a previous value, or an inference | `02` §5.5, `01` §31 |
| **Confidence is all-or-nothing** — computed only when all four `02` §21 terms are available; otherwise refused with every absent term named | `02` §21 |
| **Specification is not runtime** — a documented requirement is never evidence that it happened | §7A |

---

# 7A. Validation Tiers — Specification Is Not Runtime

Four distinct claims. They are **not** interchangeable, and conflating them is a
reporting defect (`01` §31, `PLAYBOOK` §16).

| Tier | Question | Sufficient evidence |
|---|---|---|
| **Specification validation** | Does the architecture *say* this must happen? | A document and section reference |
| **Implementation validation** | Does the implementation *contain* the behaviour? | Source plus a passing unit/integration test |
| **Runtime validation** | Did the framework *do* it against a real authorized target? | Artifacts produced by that execution |
| **Evidence validation** | Can a persisted value be *traced and recomputed*? | The artifact path, its inputs, and a reproduction |

> **Specification PASS is not Runtime PASS.**
>
> **Implementation test PASS is not Runtime PASS.**

Report the tier you actually have:

- `PASS` — observed at the tier claimed, with the artifact path.
- `NOT DEMONSTRATED` — the framework ran, but this property was not observable.
- `BLOCKED` — execution could not legitimately proceed.
- `FAIL` — execution proceeded and a required invariant was violated.

`NOT PRODUCED` is the honest answer for an artifact that does not exist. Never
infer it from another artifact, and never create it to satisfy a check.

---

# 8. Non-Goals

Out of scope by architecture (`01` §2):

penetration testing · production exploitation · destructive automation ·
unsupported browser automation · CAPTCHA bypass · authentication bypass

Security validation is a **secondary, opt-in** goal — passive by default. Active
security scanning requires explicit authorization and a dedicated test
environment (Playbook §9). Native mobile automation is a future capability
(`01` §35), not a current one.

---

# 9. On Invocation

## Before anything else — authorization

0. **Establish the authorized target.** Use only a target the user has explicitly
   authorized. If the target or its authorization is **missing or ambiguous**,
   **STOP and report `BLOCKED`**. SHALL NEVER discover, select, invent, or
   substitute a target, and SHALL NEVER carry authorization forward from a
   previous run to a new one.

Then:

1. Read **`01`** in full. Highest architectural authority.
2. Read the **Ownership Matrix**. Never implement a capability another engine owns.
3. Read **`02`**. Nothing decides for itself.
4. Read the document owning the phase you are entering, **before** entering it.
5. Before raising any request listed in §3A, run the **QA Reasoning Layer** and
   attach its package. Silent unless verbose or debug output is enabled.
6. Consult **`16`** for operational judgement and the **Implementation Playbook**
   for tactics, once you know what the phase requires.
7. Walk the phases in order, including the optional ones enabled. Log every state
   transition and every skip with its reason.
8. At each gate: **PASS** → advance · **WARNING** → advance, record ·
   **FAIL (required)** → stop, diagnose · **FAIL (optional)** → degrade, report.
9. If a mandatory input is missing, stop at the gate and report it.

## Minimum Viable Run

Required engines only — `02`–`07` and `09`. This is exactly the v3.0 pipeline and
is always valid. Enable `08` and `10`–`15` as the target's scale justifies.

## Governance The Executing Agent Enforces

These hold in every run, and are the reason a run may legitimately stop:

- **Authorized targets only.** Missing or ambiguous authorization → `BLOCKED`.
- **Never substitute a target**, and never scan an unrelated system.
- **Preserve historical artifacts.** A previous execution's output is immutable
  evidence. Never modify, overwrite, or append to it.
- **Keep runtime artifacts inside the current execution's own boundary.**
- **Preserve ownership.** Never implement a capability another engine owns, and
  never become the owner of an artifact you consume (§3, Matrix).
- **Preserve confidence ownership** — `02` §21 only — and **Evidence Quality
  ownership** — `PLAYBOOK` §19 as `02`'s term (§5A).
- **Preserve identity ownership** — `06` §22.1, `01` §30, `01` §30.1 (§5B).
- **Introduce nothing new**: no engine, dataset, lifecycle state, phase,
  identifier authority, ownership authority, confidence model, or Evidence
  Quality model.
- **Never modify a frozen document** unless the current task explicitly
  authorizes it.
- **Never fabricate evidence** to satisfy a criterion, and never convert
  specification evidence into runtime evidence (§7A).
- Report `NOT DEMONSTRATED` when runtime evidence does not exist, `BLOCKED` when
  execution cannot legitimately proceed, and `NOT PRODUCED` for an artifact that
  was never created.

If satisfying a requirement would need any of the prohibited changes above,
**STOP and report the exact conflict.** Do not invent a resolution.

## Before Reporting

State which optional engines ran, which were skipped, and what was not verified.
Reconcile every number against the runner's own output. Separate target defects
from suite defects, and environment failures from application failures.

State the **validation tier** of every claim (§7A), and give the artifact path
for each reported value. For the evidence chain, report the Evidence Object
count, the per-item confidences, `evidenceQuality`, and either Framework
Confidence or the named unavailable terms (§5A). A capped, optimized,
early-stopped, incrementally-discovered, single-browser or credential-limited run
SHALL NEVER be presented as complete.

---

# End of Execution Entry Point
