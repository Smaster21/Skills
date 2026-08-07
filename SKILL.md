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

## Before Reporting

State which optional engines ran, which were skipped, and what was not verified.
Reconcile every number against the runner's own output. Separate target defects
from suite defects, and environment failures from application failures.

---

# End of Execution Entry Point
