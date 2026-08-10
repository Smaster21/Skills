# QA Automation Framework

# Master Architecture Specification

**Document:** 01_Master_Architecture.md

**Architecture Version:** 4.0

**Status:** Draft

**Author:** AI Architecture Specification

**Governs:** Documents 01–15

**Governance Contract:** `Architecture_Ownership_Matrix.md`

**Purpose**

This document defines the complete architecture, engineering principles, execution model, governance, and extensibility model of the QA Automation Framework.

Every future implementation must comply with this specification.

If any implementation conflicts with this specification, this document SHALL take precedence.

---

# Authority Model

This document is the **highest architectural authority** in the framework.

`Architecture_Ownership_Matrix.md` is the **ownership allocation contract**. It
does not compete with this document; it implements this document's Modularity
principle (§3.3) by assigning exactly one owner to every capability.

```
01_Master_Architecture.md          ← highest architectural authority
        │                            defines architecture, principles, lifecycle
        ▼
Architecture_Ownership_Matrix.md   ← ownership allocation contract
        │                            assigns owners, datasets, runtime authority
        ▼
02 … 15                            ← engine specifications
        │                            inherit from 01, comply with the Matrix
        ▼
IMPLEMENTATION_PLAYBOOK.md         ← non-normative tactics
```

Resolution order:

- Where an engine document conflicts with **this** document, this document wins.
- Where an engine document conflicts with the **Ownership Matrix** on *who owns
  what*, the Matrix wins until this document is amended.
- Where the Matrix appears to conflict with an architectural principle stated
  here, this document wins and the Matrix SHALL be corrected.

See §37 for the full governance contract.

---

# Revision History

| Version | Description                                                          |
| ------- | -------------------------------------------------------------------- |
| 1.0     | Initial QA Automation Skill                                          |
| 2.0     | Modular execution and self-healing improvements                      |
| 3.0     | Enterprise AI Automation Architecture                                |
| 4.0     | Expanded architecture: 15 engines, Ownership Matrix governance, Framework Cache Service, execution artifact separation, intelligence / review / optimization states |

---

# Table of Contents

1. Vision
2. Goals
3. Design Principles
4. Architecture Overview
5. System Components
6. Decision Engine
7. Execution Lifecycle
8. State Machine
9. Governance
10. Quality Standards
11. Extension Model
12. Future Evolution

**Architecture v4.0 additions (§37–§42)**

37. Architecture Ownership Matrix
38. Engine Registry
39. Framework Cache Service
40. Execution Artifact Separation
41. Dependency Architecture
42. Ownership Governance

---

# 1. Vision

The framework is not a Playwright generator.

The framework is not a Selenium wrapper.

The framework is not a test recorder.

The framework is an AI-powered QA Engineering Platform capable of understanding an unknown web application, planning test strategy, generating maintainable automation, executing intelligently, learning from failures, and continuously improving.

The framework should behave like an experienced Senior QA Automation Engineer rather than a code generator.

---

# 2. Goals

Primary goals:

- Understand unknown web applications.
- Build an application model.
- Discover business workflows.
- Prioritize testing based on risk.
- Generate enterprise-quality Playwright automation.
- Produce maintainable Page Objects.
- Capture complete diagnostics.
- Recover safely from failures.
- Learn from execution history.
- Support enterprise CI/CD.

Secondary goals:

- Accessibility validation.
- Performance validation.
- API validation.
- Security validation.
- Visual validation.
- Cross-browser validation.

Non-goals:

- Penetration testing.
- Production exploitation.
- Destructive automation.
- Unsupported browser automation.
- CAPTCHA bypass.
- Authentication bypass.

---

# 3. Engineering Principles

Every module must satisfy these principles.

## 3.1 Deterministic

Given identical inputs and configuration, the framework should produce equivalent outputs.

Avoid random behavior unless explicitly requested.

---

## 3.2 Observable

Every important decision must be explainable.

Examples:

- Why a locator was selected.
- Why a retry occurred.
- Why a test was skipped.
- Why a page was classified as critical.
- Why self-healing was accepted.

Never perform silent decisions.

---

## 3.3 Modular

Each subsystem owns one responsibility.

Subsystems communicate through documented interfaces.

No module may directly manipulate another module's internal state.

---

## 3.4 Enterprise First

Design for:

- maintainability
- scalability
- reliability
- auditability
- reproducibility
- CI/CD

Never optimize solely for small demo projects.

---

## 3.5 Safe By Default

Potentially destructive operations require explicit opt-in.

Examples include:

- Delete actions
- Bulk updates
- Database reset
- Active security scanning
- Load testing
- Data mutation

Default behavior is observational.

---

## 3.6 Explainable AI

Every AI-generated output should include reasoning metadata internally (not necessarily user-facing), including confidence, assumptions, and evidence used.

---

# 4. High-Level Architecture

The framework consists of **15 primary engines**, governed by
`Architecture_Ownership_Matrix.md`, supported by platform services that belong to
no single phase.

```
                              User
                                │
                                ▼
                       Configuration Layer
                                │
                                ▼
                  ╔═════════════════════════════╗
                  ║   02  DECISION ENGINE       ║  ← sole decision authority
                  ║   risk · confidence ·       ║
                  ║   policy · conflict         ║
                  ╚═════════════════════════════╝
                                │
     ┌──────────────────────────┼──────────────────────────┐
     ▼                          ▼                          ▼
13 Incremental          03 Discovery Engine        Capability Registry
   Discovery      ────────────► (full or selective)
   (change detect)              │
     │                          ▼
     └────────────────► 04 KNOWLEDGE GRAPH  ← canonical application model
                                │
                                ▼
                    11 Unified Test Intelligence
                       (derived projection)
                                │
                                ▼
                       05 Test Planning Engine
                          test-plan.json
                                │
                                ▼
                    15 AI Planning Review  ──► 02 decides
                       (advisory)
                                │
                                ▼
                     06 Test Generation Engine
                                │
                                ▼
                        Validation Engine (§18)
                                │
                                ▼
                  12 Adaptive Execution Optimizer
                     execution-optimization-proposal.json
                     (OPTIONAL · advisory)
                                │
                                ▼
                  ╔═════════════════════════════╗
                  ║   07  EXECUTION ENGINE      ║  ← sole runtime authority
                  ║   runtime-schedule.json     ║
                  ╚═════════════════════════════╝
                                │
     ┌──────────────────────────┼──────────────────────────┐
     ▼                          ▼                          ▼
08 Self-Healing         Diagnostics (§23)         14 Confidence &
   (02 approves)                                     Coverage Optimizer
     │                          │                     (stop recommendation
     └──────────────────────────┼──────────────────────┘   → 02 decides)
                                ▼
                      09 Reporting & Analytics
                                │
                                ▼
                     10 AI Learning Repository
                                │
                                ▼
                        Future Executions (via 02)


  ── PLATFORM SERVICES (span the whole execution, own no phase) ──

  Framework Cache Service (owned by 11 · §39)   Event Bus (§27)
  regions: discovery · intelligence ·           Capability Registry (§7)
           planning · runtime-prediction ·      Configuration Layer (§29)
           plugin:*
```

## Reading The Diagram

- **Double-boxed engines hold authority.** 02 decides; 07 executes. No other
  engine holds either.
- **11, 13, 14, 15 are advisory or derived.** They enrich, detect, recommend, and
  review. None decides.
- **12 is optional.** Removing it changes speed, never correctness (§40).
- **Platform services are not lifecycle phases.** They are available from
  `CONFIGURING` onward and are consumed by many engines (§39).

---

# 5. Core Components

The framework consists of **15 primary engines**, each specified by its own
document, plus cross-cutting components that belong to no single phase.

## 5.1 Primary Engines (Documents 01–15)

| # | Engine | Primary Responsibility |
| - | ------ | ---------------------- |
| 01 | Master Architecture | Framework governance |
| 02 | Decision Engine | All engineering decisions; risk, confidence, policy, conflict resolution |
| 03 | Discovery Engine | Understanding the application |
| 04 | Knowledge Graph | Canonical application model |
| 05 | Test Planning Engine | Test plan, strategy, dependencies, resources, canonical coverage |
| 06 | Test Generation Engine | Automation generation, incremental regeneration |
| 07 | Execution Engine | Runtime execution and runtime scheduling |
| 08 | Self-Healing Engine | Locator recovery, locator evolution |
| 09 | Reporting & Analytics | Reports and historical analytics |
| 10 | AI Learning Repository | Long-term historical memory |
| 11 | Unified Test Intelligence Engine | Derived intelligence projection; Framework Cache Service |
| 12 | Adaptive Execution Optimizer | Execution optimization, batch planning, runtime prediction |
| 13 | Incremental Discovery Engine | Application change detection, fingerprinting |
| 14 | Confidence & Coverage Optimizer | Coverage optimization, stop-condition recommendation |
| 15 | AI Planning Review Engine | Advisory review of the Test Plan |

Full ownership detail, canonical datasets, and consumer lists are defined by
`Architecture_Ownership_Matrix.md`. See §38 for the engine registry and §37 for
the governance contract.

## 5.2 Cross-Cutting Components

These are **not** engines and own no lifecycle phase:

1. Configuration Layer (§29)
2. Capability Registry (§7)
3. Event Bus (§27)
4. Validation Engine (§18)
5. Artifact Manager (§22)
6. Diagnostics Engine (§23)
7. Security Engine
8. Framework Cache Service (§39 — platform service owned by 11)

## 5.3 Backward Compatibility

The v3.0 component list is preserved in full. Every component named in v3.0
remains present, unrenamed, and unrenumbered. Version 4.0 makes the engine
boundaries explicit and adds engines 11–15; it removes nothing.

Each component owns exactly one bounded responsibility. Duplicate ownership is
prohibited (§42).

---

# 6. Decision Engine

## Purpose

The Decision Engine is the brain of the framework.

No subsystem should independently decide:

- what to test
- when to retry
- when to stop
- when to self-heal
- when to escalate

All such decisions must be delegated to the Decision Engine.

---

## Responsibilities

The Decision Engine answers questions such as:

- What page should be tested next?
- Has sufficient discovery been completed?
- Is retry appropriate?
- Is the failure flaky?
- Is locator healing safe?
- Has required coverage been achieved?
- Should execution stop?
- Should additional tests be generated?

---

## Inputs

- Configuration
- Discovery Results
- Knowledge Graph
- Execution History
- Diagnostics
- Coverage Metrics
- Risk Scores

---

## Outputs

- Execution Plan
- Retry Decisions
- Healing Decisions
- Priorities
- Coverage Decisions
- Stop Conditions

---

## Decision Rules

Every decision must be:

- deterministic where possible
- evidence-based
- logged
- reproducible
- auditable

The Decision Engine must never rely solely on intuition or randomness.

---

# 7. Capability Registry

Every feature supported by the framework must be registered.

Examples:

- Playwright
- API Testing
- Accessibility
- Performance
- Security
- Visual Testing
- Mobile Emulation
- Multi-Browser

Each capability includes:

- Name
- Version
- Dependencies
- Configuration
- Status (Enabled/Disabled)
- **Health (Healthy/Degraded/Unavailable/Unknown)**
- Owner Component

The Decision Engine queries the Capability Registry before invoking any feature.

---

## 7.1 Status And Health Are Orthogonal

`Status` and `Health` describe different things and SHALL NEVER be conflated.

| Field | Expresses | Set by | Changes |
| ----- | --------- | ------ | ------- |
| **Status** | **Configuration intent** — is this capability permitted to be used? | Configuration Layer (§29) | Only by configuration |
| **Health** | **Observed runtime condition** — is this capability actually working? | Capability Registry, on a Decision Engine decision | Only at deterministic checkpoints (§7.3) |

Both fields are always present. Neither implies the other.

A capability may be `Enabled` and `Unavailable` — permitted but currently broken.
A capability may be `Disabled` and `Healthy` — working, but not authorised for use.

Configuration SHALL NEVER be inferred from health, and health SHALL NEVER be
inferred from configuration.

---

## 7.2 Health Values

| Value | Meaning |
| ----- | ------- |
| **Healthy** | Observed working within expected parameters |
| **Degraded** | Observed working, but outside expected parameters |
| **Unavailable** | Observed not working |
| **Unknown** | Not yet observed this execution — the default |

### Unknown Is The Default

Every capability begins each execution at `Health: Unknown` and remains there
until observed. `Unknown` is not a failure.

**`Unknown` SHALL NEVER independently gate execution.** An unobserved capability
is not a broken capability, and treating it as one would block every first
execution and every capability the run does not exercise.

Where a gate requires a health condition, that gate SHALL specify which values
satisfy it. `Unknown` alone SHALL NOT cause a stop.

---

## 7.3 Health Transitions

Health is a **field of a registry record**, not a lifecycle state. It introduces
no state in §17 and no phase in §16.

Transitions SHALL occur **only at deterministic checkpoints** — the batch and
workflow-group boundaries already used for checkpoint evaluation elsewhere in the
framework. Health SHALL NEVER change mid-batch, on test-completion order, or on
wall-clock elapse, because none of those is deterministic under parallel
execution (§3.1).

Two executions observing identical conditions at identical checkpoints SHALL
produce an identical transition sequence.

### Authority

Health is **observed** by the engines that encounter the capability, **decided**
by the Decision Engine, and **recorded** by the Capability Registry.

```
Observation  →  Decision Engine (§6)  →  Capability Registry records
```

The Registry SHALL NOT decide. No engine SHALL write health directly. This
introduces no authority: §6 already holds sole decision authority, and the
Registry already holds capability state.

Every transition SHALL be logged with its timestamp, the evidence that prompted
it, and the decision that authorised it (§30).

---

## 7.4 Backward Compatibility

`Status` is unchanged in name, values, and meaning.

A registry record carrying no `Health` value is read as `Health: Unknown`.
Capabilities registered before this amendment therefore remain valid without
modification, and behaviour is unchanged until a health observation occurs.
--------------------------------------------------------------------------------

# 8. Knowledge Graph

## Purpose

The Knowledge Graph is the framework's long-term understanding of the application.

Unlike temporary discovery results, the Knowledge Graph stores structured relationships between application entities and enables reasoning across multiple execution phases.

The Knowledge Graph SHALL be treated as the canonical representation of the application under test.

---

## Objectives

The Knowledge Graph must answer questions such as:

- What pages exist?
- Which components belong to each page?
- Which workflows connect multiple pages?
- Which APIs support a workflow?
- Which components are reused?
- Which pages require authentication?
- Which business flows are critical?
- Which tests already provide coverage?

---

## Core Entities

The Knowledge Graph shall model at least the following entities:

- Application
- Environment
- Page
- Route
- Component
- Form
- Modal
- Navigation Element
- API Endpoint
- Workflow
- User Role
- Permission
- Test Case
- Locator
- Assertion
- Artifact

Each entity shall have a unique identifier.

---

## Relationships

Relationships are first-class objects.

Examples:

Page
→ contains
→ Component

Component
→ triggers
→ API

Workflow
→ includes
→ Page

Role
→ accesses
→ Page

Test
→ validates
→ Workflow

Locator
→ identifies
→ Component

Assertion
→ verifies
→ Component

---

## Versioning

The Knowledge Graph shall be versioned.

Each execution creates a new snapshot.

Example:

knowledge/
    graph-v1.json
    graph-v2.json
    graph-v3.json

The framework shall compare graph versions to detect application changes.

---

# 9. Discovery Engine

## Purpose

The Discovery Engine is responsible for understanding the application before any test generation occurs.

Discovery SHALL always precede planning.

No test generation may begin without successful discovery unless explicitly disabled.

---

## Discovery Objectives

The engine shall discover:

- Routes
- Pages
- Navigation
- Components
- Forms
- Tables
- Modals
- Drawers
- Search Components
- Pagination
- Infinite Scroll
- File Uploads
- API Calls
- Authentication
- Cookies
- Local Storage
- Session Storage
- Service Workers
- Feature Flags (where detectable)

---

## Crawl Strategy

Default strategy:

- Breadth-first search
- Same-origin only
- Configurable maximum depth
- Deduplicate URLs
- Normalize query parameters
- Ignore logout routes by default
- Respect configured exclusions

Optional:

- Sitemap discovery
- robots.txt parsing
- Seed URLs
- Manual include paths

---

## Discovery Outputs

Discovery produces:

application.json

pages.json

routes.json

components.json

apis.json

navigation.json

authentication.json

discovery-report.json

These artifacts become inputs to later stages.

---

# 10. Component Classification Engine

## Purpose

Raw HTML elements are not sufficient.

The framework must classify elements into meaningful business components.

Example:

Instead of

input

The framework should recognize

Search Box

or

Email Field

or

Password Field

---

## Classification Levels

Level 1

HTML

button

input

select

textarea

---

Level 2

Semantic

Login Button

Checkout Button

Delete Button

Save Button

Search Field

Filter Dropdown

Date Picker

---

Level 3

Business Context

Authentication

Checkout

Payment

Administration

Reporting

Settings

Analytics

---

## Classification Confidence

Every classification shall include:

Confidence Score

Evidence

Alternative Classifications

Example

Search Box

Confidence

97%

Evidence

Placeholder contains "Search"

Role

searchbox

Associated icon

magnifier

---

# 11. Business Flow Detection Engine

## Purpose

The framework should discover user journeys rather than isolated pages.

---

## Examples

Authentication

Login

↓

Dashboard

↓

Logout

---

E-Commerce

Home

↓

Products

↓

Product Detail

↓

Cart

↓

Checkout

↓

Payment

↓

Confirmation

---

Administration

Login

↓

Dashboard

↓

Users

↓

Edit User

↓

Save

↓

Audit Log

---

## Detection Sources

Business flows should be inferred using:

Navigation links

Forms

Button actions

Network requests

URL transitions

Breadcrumbs

Page titles

Component semantics

Historical execution

---

## Workflow Graph

Each detected workflow shall contain:

Workflow ID

Pages

Transitions

Criticality

Dependencies

Estimated Duration

Covered Tests

Coverage Status

---

# 12. Risk Analysis Engine

## Purpose

Testing effort should be proportional to business risk.

The framework shall compute a risk score for every page, workflow, and component.

---

## Risk Factors

Business Criticality

Authentication Required

Financial Operations

Data Modification

Administrative Privileges

External Integrations

Sensitive Data

Historical Failures

User Impact

Complexity

---

## Risk Levels

Critical

High

Medium

Low

Informational

---

## Risk Output

Every discovered page shall include:

Risk Score

Risk Category

Reason

Suggested Test Depth

Example

Checkout

Risk

Critical

Reason

Financial transaction

Recommended Tests

150

---

# 13. Test Planning Engine

## Purpose

Planning determines what will be tested before any code generation.

Generation without planning is prohibited.

---

## Planning Responsibilities

Determine:

Execution order

Test priorities

Coverage goals

Dependencies

Required fixtures

Required environments

Browser matrix

Parallelization

Retry strategy

Artifact collection

---

## Planning Output

The engine generates:

test-plan.json

The plan shall include:

Execution phases

Estimated duration

Resource requirements

Dependencies

Coverage targets

Exit criteria

---

## Planning Rules

Critical workflows execute before low-risk pages.

Shared fixtures are reused.

Independent tests execute in parallel.

State-dependent tests execute sequentially.

Flaky tests are isolated.

Security tests are opt-in unless configured.

---

# 14. Data Contracts

Every module exchanges structured contracts.

No module should depend on another module's internal implementation.

Minimum contracts:

Discovery Contract

Knowledge Contract

Planning Contract

Execution Contract

Healing Contract

Reporting Contract

Each contract shall include:

Schema Version

Timestamp

Correlation ID

**Execution Scope Identity**

Producer

Consumer

Payload

Validation Status

Execution Scope Identity is defined by §30.1 and owned by this document. It is
carried on every contract so that any consumer can determine which application a
payload belongs to without inferring it. Where no scope is supplied, the field
carries the default scope (§30.1).

---

## Canonical Datasets

Every canonical dataset has **exactly one owner**, defined by
`Architecture_Ownership_Matrix.md` under *Canonical Data Ownership*.

A dataset's owner is the only component permitted to produce it. Any number of
components may consume it.

Additional contracts introduced in v4.0:

Intelligence Contract *(11)*

Optimization Proposal Contract *(12)*

Discovery Delta Contract *(13)*

Coverage Recommendation Contract *(14)*

Planning Review Contract *(15)*

Cache Region Contract *(11, §39)*

Runtime Schedule Contract *(07, §40)*

A component SHALL NEVER produce a dataset it does not own, and SHALL NEVER
duplicate an owner's algorithm. Where a capability is required but not owned, it
SHALL be consumed through the owner's public contract (§42).

---

# 15. Quality Gate — Intelligence Layer

Before entering test generation, the following conditions must be satisfied:

✓ Discovery completed

✓ Knowledge Graph built

✓ Pages classified

✓ Components classified

✓ Business workflows detected

✓ Risk analysis completed

✓ Test plan generated

✓ Required configuration validated

If any mandatory gate fails, the framework shall stop and report the issue rather than generating incomplete tests.

---

## Additional Gates — Architecture v4.0

These gates apply **only when their engine is present**. An absent engine SHALL
NOT block the pipeline; its gate is recorded as not applicable.

| Gate | Owner | Mandatory when present |
| ---- | ----- | :--------------------: |
| Change detection completed, or full discovery elected | 13 | ✓ |
| Fingerprint stability verified | 13 | ✓ |
| Discovery Delta accepted by 04 | 13 → 04 | ✓ |
| Intelligence projection validated and stamped with its source graph version | 11 | ✓ |
| Planning review completed and submitted to 02 | 15 | ✓ |
| Optimization proposal validated against the Test Plan test set | 12 → 07 | ✓ |

## Gate Failure Semantics

| Outcome | Effect |
| ------- | ------ |
| PASS | Advance |
| WARNING | Advance, record the finding |
| FAIL on a mandatory gate | Stop, produce a diagnostic report |
| FAIL on an optional engine's gate | **Degrade, do not stop** |

Degradation is explicit and always safe:

- 13 fails → full discovery
- 11 fails → consume 04 directly
- 12 fails or its proposal is rejected → execute the Test Plan's own ordering
- 14 unavailable → execute the full approved plan
- 15 fails → submit the plan to 02 unreviewed, flagged as unreviewed

In every case the executed result is correct. Only cost or assurance changes, and
the degradation SHALL be reported by 09.

---

# 16. Execution Orchestrator

## Purpose

The Execution Orchestrator is responsible for coordinating the execution of all generated tests.

It SHALL NOT make business decisions.

Business decisions belong exclusively to the Decision Engine.

The Execution Orchestrator executes the approved Test Plan.

---

## Responsibilities

The Execution Orchestrator shall:

- Load the Test Plan
- Schedule execution
- Allocate browsers
- Manage worker pools
- Coordinate fixtures
- Collect execution metrics
- Capture artifacts
- Report execution state
- Notify the Decision Engine of failures

---

## Execution Pipeline

Every execution shall follow the deterministic pipeline below. Engine ownership
is shown for each phase.

```
Configuration                                    Configuration Layer
        ↓
Change Detection            (optional)           13  Incremental Discovery
        ↓
Discovery                   (full or selective)  03  Discovery Engine
        ↓
Knowledge Graph                                  04  Knowledge Graph
        ↓
Unified Intelligence                             11  Intelligence Projection
        ↓
Planning                                         05  Test Planning Engine
        ↓
AI Planning Review          (advisory)           15  Planning Review → 02
        ↓
Generation                                       06  Test Generation Engine
        ↓
Validation                                       Validation Engine (§18)
        ↓
Execution Optimization      (OPTIONAL)           12  Execution Optimizer
        ↓
Execution                                        07  Execution Engine
        ↓
Coverage Optimization       (advisory, at        14  Coverage Optimizer → 02
                             checkpoints)
        ↓
Artifact Collection                              Artifact Manager (§22)
        ↓
Diagnostics                                      Diagnostics Engine (§23)
        ↓
Reporting                                        09  Reporting & Analytics
        ↓
Learning Repository Update                       10  AI Learning Repository
```

## Pipeline Rules

**Determinism.** Given identical inputs and configuration, the pipeline SHALL
produce equivalent outputs (§3.1). Phases marked *optional* or *advisory* SHALL
NEVER alter that guarantee.

**Optional phases.** Change Detection (13) and Execution Optimization (12) MAY be
absent. Their absence changes cost, never correctness:

| Phase absent | Consequence |
| ------------ | ----------- |
| 13 Change Detection | Full discovery runs. Result identical. |
| 11 Unified Intelligence | Downstream engines consume 04 directly. Result identical. |
| 12 Execution Optimization | 07 executes the Test Plan's own ordering. Result identical. |
| 14 Coverage Optimization | Full approved plan executes. Result identical, cost higher. |
| 15 Planning Review | Plan proceeds to 02 unreviewed. Result identical, quality unassessed. |

**Advisory phases.** 14 and 15 produce recommendations only. Every recommendation
is submitted to **02**, which decides. No advisory phase may halt, skip, or
approve on its own authority.

**No phase may be skipped without explanation.** An omitted phase SHALL be
recorded with its reason and reported by 09.

**Backward compatibility.** The v3.0 pipeline is a valid subset of this pipeline.
An execution with 11, 12, 13, 14, and 15 all absent is exactly the v3.0
lifecycle and remains fully supported.

---

## Execution Modes

Supported modes:

- Smoke
- Regression
- Sanity
- Full
- Security
- Accessibility
- Performance
- Visual
- API
- Custom

Each mode may define:

- Included test groups
- Retry policy
- Browser matrix
- Reporting level

---

# 17. State Machine

The framework SHALL maintain an explicit execution state.

Allowed states:

INITIALIZING

CONFIGURING

DETECTING_CHANGES

DISCOVERING

BUILDING_GRAPH

**INTELLIGENCE**

PLANNING

**PLANNING_REVIEW**

GENERATING

VALIDATING

**OPTIMIZATION**

EXECUTING

RETRYING

SELF_HEALING

COLLECTING_ARTIFACTS

DIAGNOSTICS

REPORTING

LEARNING

COMPLETED

FAILED

CANCELLED

Every state transition shall be logged.

No state may be skipped without explanation.

---

## States Added In v4.0

| State | Owner | Purpose | Skippable |
| ----- | ----- | ------- | :-------: |
| `DETECTING_CHANGES` | 13 | Fingerprint comparison against the previous snapshot | Yes |
| **`INTELLIGENCE`** | 11 | Project 04 into the reuse-optimized package | Yes |
| **`PLANNING_REVIEW`** | 15 | Advisory review of the Test Plan, submitted to 02 | Yes |
| **`OPTIMIZATION`** | 12 | Produce the advisory optimization proposal | Yes |

A skippable state SHALL transition directly to its successor when its engine is
absent or disabled. The skip SHALL be logged with its reason.

`COVERAGE_OPTIMIZATION` is deliberately **not** a top-level state. Document 14
evaluates at deterministic checkpoints *within* `EXECUTING` and submits
recommendations to 02; it does not own a phase of its own.

---

## State Transition Rules

Canonical forward path:

```
INITIALIZING
     ↓
CONFIGURING
     ↓
DETECTING_CHANGES ····┐  (skippable → DISCOVERING)
     ↓                │
DISCOVERING ◄─────────┘
     ↓
BUILDING_GRAPH
     ↓
INTELLIGENCE ·········┐  (skippable → PLANNING)
     ↓                │
PLANNING ◄────────────┘
     ↓
PLANNING_REVIEW ······┐  (skippable → GENERATING)
     ↓                │
GENERATING ◄──────────┘
     ↓
VALIDATING
     ↓
OPTIMIZATION ·········┐  (skippable → EXECUTING)
     ↓                │
EXECUTING ◄───────────┘
     ↓  ├─► RETRYING ─────┐   (02 approves)
     ↓  └─► SELF_HEALING ─┤   (02 approves)
     ↓ ◄──────────────────┘
COLLECTING_ARTIFACTS
     ↓
DIAGNOSTICS
     ↓
REPORTING
     ↓
LEARNING
     ↓
COMPLETED
```

Terminal failure states: `FAILED`, `CANCELLED`.

A module may never transition backwards except through the Decision Engine.

`PLANNING_REVIEW` MAY return to `PLANNING` **only** on a 02-approved revision
decision, and only within the iteration limit defined by 15. That bounded loop is
the sole permitted backward transition added in v4.0.

## Compatibility

The v3.0 state sequence remains valid. An execution that skips
`DETECTING_CHANGES`, `INTELLIGENCE`, `PLANNING_REVIEW`, and `OPTIMIZATION`
traverses exactly the v3.0 states in exactly the v3.0 order.

No v3.0 state was renamed, removed, or reordered.

---

# 18. Validation Engine

## Purpose

Generated automation shall be validated before execution.

Poor-quality automation should never execute.

---

## Validation Rules

Validate:

- TypeScript compilation
- Playwright syntax
- Duplicate tests
- Duplicate locators
- Broken imports
- Circular dependencies
- Missing assertions
- Empty test bodies
- Invalid selectors
- Hard-coded waits
- Unsafe XPath
- Unsupported APIs

---

## Validation Outcomes

PASS

WARNING

FAIL

Warnings may proceed.

Failures stop execution.

---

# 19. Self-Healing Engine

## Purpose

The Self-Healing Engine attempts safe recovery from recoverable failures.

It shall never hide genuine application defects.

---

## Supported Healing

Locator changes

Timing issues

Detached elements

Minor DOM restructuring

Delayed rendering

Lazy loading

Shadow DOM discovery

---

## Unsupported Healing

Incorrect assertions

Backend defects

Business logic defects

Security vulnerabilities

Authorization failures

Application crashes

Data corruption

---

## Healing Workflow

Failure

↓

Classify

↓

Is healing allowed?

↓

NO

↓

Report

↓

END

YES

↓

Generate candidate solution

↓

Verify

↓

Retry

↓

Pass?

↓

YES

↓

Record healing event

↓

Continue

↓

NO

↓

Report failure

---

## Healing Principles

Healing must:

- Be deterministic
- Preserve test intent
- Avoid masking defects
- Produce audit logs
- Record confidence

---

# 20. Locator Intelligence

## Locator Priority

The framework shall always prefer stable semantic locators.

Priority order:

1. getByTestId()
2. getByRole()
3. getByLabel()
4. getByPlaceholder()
5. getByText()
6. aria-label
7. name
8. id
9. stable CSS
10. relative XPath

Absolute XPath is prohibited.

---

## Locator Confidence

Each locator receives:

Confidence Score

Reason

Evidence

Example

Locator

getByRole("button", { name: "Login" })

Confidence

98%

Reason

Accessible name + unique role

---

## Locator History

Every successful replacement shall be recorded.

Example

Old Locator

button.submit

↓

New Locator

getByRole(...)

↓

Version

2

↓

Confidence

96%

The framework SHALL NOT overwrite existing locators immediately.

Candidate locators require repeated successful execution before promotion.

---

# 21. Retry Engine

## Retry Philosophy

Retries exist only for transient failures.

Retries shall never hide application defects.

---

## Retry Policy

Attempt 1

Normal retry

Attempt 2

Adaptive wait strategy

Attempt 3

Self-healing

Final attempt

Decision Engine decides:

Stop

Escalate

Continue

---

## Retry Classification

Retry only for:

Timing

Network instability

Rendering delay

Detached element

Animation overlap

Do not retry:

Assertion failures

Logic failures

Permission failures

Application exceptions

Security defects

---

# 22. Artifact Manager

Every execution shall collect standardized artifacts.

Artifacts include:

Screenshot

Full-page screenshot

Trace

Video

Console logs

Network logs

HAR

DOM snapshot

Page HTML

Cookies

Storage

Playwright trace

Execution metadata

Environment metadata

Artifacts shall be linked to:

Execution ID

Test ID

Workflow ID

Correlation ID

---

# 23. Diagnostics Engine

Purpose:

Convert raw failures into engineering knowledge.

---

## Diagnostics Sources

Console

Network

Trace

Screenshot

DOM

Knowledge Graph

Execution History

Locator History

API Responses

Browser Events

---

## Failure Categories

Configuration

Locator

Timing

Rendering

API

Backend

Authentication

Authorization

Assertion

Browser

Accessibility

Performance

Unknown

---

## Root Cause Output

Every failure report shall include:

Category

Severity

Confidence

Evidence

Suggested Fix

Related Components

Affected Workflows

Historical Occurrence

---

# 24. Learning Repository

## Purpose

Unlike the Knowledge Graph, the Learning Repository stores historical execution knowledge.

It improves future executions.

---

## Repository Stores

Locator evolution

Retry history

Flaky tests

Execution duration

Performance trends

Healing success

Failure frequency

Browser instability

Coverage history

Regression history

---

## Learning Rules

The repository SHALL NEVER modify generated tests directly.

It provides recommendations.

The Decision Engine decides whether to apply them.

---

# 25. Metrics & Telemetry

Every execution shall produce measurable metrics.

Examples:

Discovery Duration

Planning Duration

Generation Duration

Execution Duration

Healing Success Rate

Retry Rate

Flaky Rate

Coverage

Pass Rate

Failure Rate

Average Locator Confidence

Average Test Duration

Artifact Size

Risk Coverage

These metrics support continuous improvement.

---

# 26. Execution Quality Gates

Before execution:

✓ Test Plan Approved

✓ Validation Passed

✓ Configuration Valid

✓ Browser Ready

✓ Fixtures Loaded

✓ Environment Verified

During execution:

✓ Artifacts Captured

✓ Metrics Recorded

✓ State Logged

✓ Healing Audited

✓ Capability Health Recorded

Capability Health Recorded is satisfied when every health transition observed at
a checkpoint has been recorded in the Capability Registry with its evidence and
authorising decision (§7.3).

A capability at `Health: Unknown` satisfies this gate. `Unknown` means unobserved,
not unrecorded, and SHALL NOT cause the gate to fail (§7.2).

After execution:

✓ Reports Generated

✓ Learning Repository Updated

✓ Coverage Calculated

✓ Summary Produced

If any mandatory quality gate fails, execution shall stop safely and produce a diagnostic report.

---

# 27. Event Bus Architecture

## Purpose

The Event Bus provides a decoupled communication mechanism between framework components.

Instead of directly invoking each other, components SHALL publish events.

Interested components subscribe to those events.

This architecture improves scalability, extensibility, testing, and plugin support.

---

## Design Goals

The Event Bus SHALL:

- Remove tight coupling
- Support plugins
- Enable asynchronous workflows
- Improve observability
- Simplify debugging
- Support future distributed execution

---

## Core Events

Framework lifecycle events

ConfigurationLoaded

DiscoveryStarted

DiscoveryCompleted

KnowledgeGraphCreated

PlanningStarted

PlanningCompleted

GenerationStarted

GenerationCompleted

ExecutionStarted

ExecutionCompleted

ExecutionCancelled

ValidationFailed

FrameworkCompleted

FrameworkFailed

---

Test Events

TestStarted

TestPassed

TestFailed

TestSkipped

TestRetried

TestHealed

TestCompleted

---

Locator Events

LocatorSelected

LocatorRejected

LocatorUpdated

LocatorHealed

LocatorPromoted

---

Reporting Events

ReportStarted

ReportGenerated

CoverageCalculated

SummaryGenerated

---

Security Events

PassiveScanStarted

PassiveScanCompleted

ActiveScanStarted

SecurityFindingCreated

---

Learning Events

LearningUpdated

FlakyTestDetected

RiskScoreUpdated

KnowledgeGraphUpdated

---

# 28. Plugin Architecture

## Purpose

The framework SHALL support independent plugins.

Core functionality should remain lightweight.

Additional capabilities should be installable.

---

## Plugin Types

Discovery Plugin

Planning Plugin

Generation Plugin

Execution Plugin

Reporting Plugin

Security Plugin

Performance Plugin

Accessibility Plugin

Notification Plugin

AI Plugin

Cloud Execution Plugin

---

## Plugin Lifecycle

Load

↓

Initialize

↓

Register Capabilities

↓

Subscribe Events

↓

Execute

↓

Shutdown

---

## Plugin Metadata

Every plugin must declare

Name

Version

Author

Description

Dependencies

Capabilities

Supported Framework Version

Configuration Schema

---

## Plugin Isolation

Plugins SHALL NOT

Modify internal framework state directly.

Plugins SHALL communicate through

Events

Contracts

Public APIs

---

# 29. Configuration Framework

Configuration SHALL be hierarchical.

Priority

CLI Arguments

↓

Environment Variables

↓

Project Configuration

↓

Default Configuration

---

## Configuration Categories

Execution

Discovery

Retry

Reporting

Security

Accessibility

Performance

Parallelism

Browsers

AI

Plugins

Learning

---

Every configuration option shall include

Name

Type

Default

Description

Validation Rules

---

# 30. Security & Governance

## Security Principles

Sensitive information must never appear in

Logs

Reports

Screenshots

AI prompts

Error messages

---

Examples

Passwords

Tokens

Session IDs

API Keys

Personal Information

Financial Information

must be masked before persistence.

---

## Governance Rules

Every automated action must be

Traceable

Auditable

Reproducible

Explainable

---

Every execution must receive

Execution ID

Correlation ID

**Execution Scope Identity**

Framework Version

Configuration Version

Knowledge Graph Version

---

## 30.1 Execution Scope Identity

### Purpose

Execution Scope Identity answers one question no existing identifier answers:

> **Which application is this execution about?**

Every existing identifier distinguishes *one run from another*. None distinguishes
*one application from another*. Without it, knowledge accumulated for one
application cannot be kept separate from knowledge accumulated for a different
one.

### Ownership

Execution Scope Identity is a **framework-wide identifier owned by this document**,
alongside the identifiers listed above.

It SHALL NOT be owned by any engine. Documents `04`, `05`, `09`, and `10` all
require scope isolation; if any single engine owned the identifier, the other
three would depend on a peer, violating the dependency layering of §41.

Ownership of the identifier is separate from ownership of any behaviour keyed by
it. An engine MAY use scope identity to partition data it already owns; doing so
grants it no ownership of the identifier.

### Distinction From Existing Identifiers

| Identifier | Distinguishes | Cardinality |
| ---------- | ------------- | ----------- |
| **Execution Scope Identity** | **One application from another** | **Many executions share one scope** |
| Execution ID | One run from another | One per execution |
| Correlation ID | One causal chain from another | One per execution, propagated |
| Plan ID | One test plan from another | One or more per execution |
| Framework Version | One framework release from another | One per execution |
| Configuration Version | One configuration from another | One per execution |
| Knowledge Graph Version | One graph snapshot from another | One or more per execution |

**Execution Scope Identity is the only identifier that is deliberately stable
across executions.** Every other identifier above changes, or may change, from
run to run. Scope identity persists precisely so that successive runs of the same
application are recognisable as such.

### Not Playwright's Project ID

`projectId` as emitted by the test runner identifies a **browser project** —
`chromium`, `firefox`, `webkit`, a device profile. It is unrelated to Execution
Scope Identity and SHALL NEVER be used as one. The two SHALL NOT be conflated in
any artifact, report, or contract.

### Derivation

The default scope identity is derived from the **normalized origin of the
configured base URL**.

Normalization is **consumed** through the public contract defined by
`03_Discovery_Engine.md` §18 (URL Normalization Engine). This document defines
**no normalizer**, and no component SHALL implement a second one.

An explicit scope identity MAY be supplied through the configuration hierarchy
(§29), in which case it takes precedence over the derived default.

Derivation SHALL be deterministic: the same base URL SHALL always yield the same
scope identity.

### Backward Compatibility

Where no scope identity is supplied and none can be derived, execution SHALL
resolve to a **single implicit default scope**.

Existing single-application behaviour is therefore unchanged: one implicit scope
behaves exactly as no scope at all. Artifacts produced before this amendment
remain valid and are read as belonging to the default scope.

Absence of scope identity SHALL NEVER halt execution.

---

# 31. AI Governance

AI SHALL assist engineering.

AI SHALL NOT invent facts.

AI SHALL NOT fabricate test results.

AI SHALL NOT mark failures as passes.

AI SHALL NOT suppress defects.

Every AI recommendation shall include

Confidence

Evidence

Reasoning

Suggested Action

---

# 32. Continuous Learning

Learning improves future executions.

Learning must never automatically change framework behavior.

Instead

Learn

↓

Recommend

↓

Decision Engine Reviews

↓

Human Approval (optional)

↓

Apply

---

Learning Sources

Execution History

Locator History

Coverage

Flaky Tests

Failures

Performance

Security Findings

---

# 33. Quality Standards

Every generated project must satisfy

No duplicated tests

No duplicated locators

No hardcoded waits

No invalid imports

No TypeScript errors

No lint errors

Deterministic execution

Independent tests

Parallel-safe execution

Stable locators

Accessible selectors

Reusable Page Objects

Meaningful assertions

Readable code

---

# 34. Enterprise Requirements

The framework SHALL support

Multi-browser

Multi-environment

CI/CD

Parallel execution

Retry policies

Artifact retention

Report archiving

Incremental execution

Distributed execution (future)

Plugin ecosystem

Versioned architecture

---

# 35. Future Architecture

Future capabilities should integrate without redesign.

Examples

Mobile Automation

Native Apps

Desktop Apps

API Contract Testing

AI Visual Validation

Chaos Testing

Mutation Testing

Contract Testing

Cloud Device Farms

LLM-powered Test Review

Distributed Execution

Voice UI Testing

AR/VR Testing

---

# 36. Architecture Success Criteria

The architecture is considered successful if it enables:

✓ Unknown application discovery

✓ Knowledge Graph generation

✓ Risk-based planning

✓ Business workflow detection

✓ Enterprise Playwright generation

✓ Self-healing automation

✓ Explainable AI decisions

✓ Safe retries

✓ Rich diagnostics

✓ Comprehensive reporting

✓ Historical learning

✓ Plugin support

✓ CI/CD integration

✓ Future extensibility

without requiring architectural redesign.

---

# ------------------------------------------------------------------

# Architecture v4.0 — Expanded Architecture Integration

# Sections 37–42

# ------------------------------------------------------------------

# 37. Architecture Ownership Matrix

## Purpose

`Architecture_Ownership_Matrix.md` is the framework's **ownership allocation
contract**. It exists to implement the Modularity principle (§3.3) — one
responsibility per subsystem — as an enforceable, auditable document.

It defines:

- **Single ownership** — every capability has exactly one owner
- **Canonical datasets** — every dataset has exactly one producer
- **Artifact ownership** — and that it never implies runtime authority
- **Runtime authority** — 07 alone schedules runtime execution
- **Decision authority** — 02 alone calculates risk, confidence, and policy
- **Conflict resolution** — which document prevails when two disagree

---

## Relationship To This Document

**This document remains the highest architectural authority.**

The Ownership Matrix governs *ownership allocation only*. It does not define
architecture, principles, or lifecycle — those are defined here.

| Question | Authority |
| -------- | --------- |
| What is the architecture? | **01** (this document) |
| What are the engineering principles? | **01** |
| What is the lifecycle and state machine? | **01** |
| Who owns capability X? | **Ownership Matrix** |
| Who produces dataset Y? | **Ownership Matrix** |
| Which engine decides? | **Ownership Matrix** → 02 |
| How is capability X implemented? | The owning engine's document |
| How is it built in Playwright? | `IMPLEMENTATION_PLAYBOOK.md` (non-normative) |

Where the Matrix conflicts with an architectural principle stated here, this
document wins and the Matrix SHALL be corrected. Where an engine document
conflicts with the Matrix on ownership, the Matrix wins until this document is
amended.

---

## The Five Ownership Principles

Restated here because they are architectural, not merely administrative:

1. **Single Ownership** — every capability has exactly one owner; only the owner
   may implement it.
2. **Multiple Consumers** — any number of engines may consume; a consumer never
   becomes an owner.
3. **No Duplicate Algorithms** — an algorithm exists once and is exposed through
   a public contract.
4. **Decision Authority** — every engineering decision originates from exactly
   one engine.
5. **Derived Information** — derived information never becomes canonical; only
   the owner defines truth.

## Compliance Requirement

Every engine document SHALL declare, in its header:

- The capabilities it **owns**, per the Matrix
- The capabilities it **consumes**, with each owner named
- An **Ownership Compliance** section listing what it explicitly does not own

A document that owns a capability the Matrix assigns elsewhere is
**non-compliant** and SHALL be revised before acceptance.

---

# 38. Engine Registry

## Purpose

The authoritative list of engines governed by this architecture, with the phase
each occupies and whether it is required.

---

## Registry

| # | Engine | Lifecycle Phase | Required | Authority |
| - | ------ | --------------- | :------: | --------- |
| 01 | Master Architecture | — (governance) | ✓ | Architectural authority |
| 02 | Decision Engine | Throughout | ✓ | **Sole decision authority** |
| 03 | Discovery Engine | `DISCOVERING` | ✓ | — |
| 04 | Knowledge Graph | `BUILDING_GRAPH` | ✓ | **Canonical application model** |
| 05 | Test Planning Engine | `PLANNING` | ✓ | Canonical Test Plan and coverage |
| 06 | Test Generation Engine | `GENERATING` | ✓ | Canonical generated tests |
| 07 | Execution Engine | `EXECUTING` | ✓ | **Sole runtime authority** |
| 08 | Self-Healing Engine | `SELF_HEALING` | Optional | Canonical locator history |
| 09 | Reporting & Analytics | `REPORTING` | ✓ | Canonical reports |
| 10 | AI Learning Repository | `LEARNING` | Optional | Canonical historical memory |
| 11 | Unified Test Intelligence | `INTELLIGENCE` | Optional | Derived projection · Cache Service |
| 12 | Adaptive Execution Optimizer | `OPTIMIZATION` | Optional | Advisory proposal only |
| 13 | Incremental Discovery | `DETECTING_CHANGES` | Optional | Canonical Discovery Delta |
| 14 | Confidence & Coverage Optimizer | Checkpoints in `EXECUTING` | Optional | Recommendation only |
| 15 | AI Planning Review | `PLANNING_REVIEW` | Optional | Advisory review only |

---

## Required Versus Optional

**Required engines** form the minimum viable architecture. This is exactly the
v3.0 pipeline, and it remains fully supported.

**Optional engines** are optimization and assurance layers. Each may be absent,
disabled, or replaced. Their absence changes cost or assurance, never
correctness (§16).

This distinction is what makes the architecture scalable across deployment sizes:
a small target may run the required set only; an enterprise target enables the
full fifteen.

---

## Authority Summary

Exactly **two** engines hold authority:

```
02 Decision Engine  →  decides WHAT happens
07 Execution Engine →  decides HOW it runs at runtime
```

Every other engine produces knowledge, plans, artifacts, or recommendations.
None decides. None executes.

---

# 39. Framework Cache Service

## Purpose

Repeated analysis and repeated discovery are the framework's two largest
avoidable costs. Both require caching, and the framework SHALL have exactly
**one** cache implementation.

The Framework Cache Service is a **platform service**, not an engine. It occupies
no lifecycle phase, owns no engineering decision, and appears in no state
transition.

It is **owned by Document 11**. No new engine and no new document is created for
it.

---

## Two-Layer Ownership

| Layer | Owner | Owns |
| ----- | ----- | ---- |
| **Cache Service** (mechanism) | **11**, exclusively | Storage, keying, TTL, eviction, tiering, backend abstraction, distribution, invalidation execution, metrics, access control |
| **Cache Region** (semantics) | Each consumer, one owner per region | Key derivation, validity predicate, invalidation triggers, retention class, sensitivity class |

The Service SHALL NEVER interpret region content.

A region owner SHALL NEVER implement storage.

This separation permits one implementation to serve every consumer without the
Service acquiring knowledge of any consumer's domain.

---

## Regions

| Region | Semantic Owner | Content |
| ------ | -------------- | ------- |
| `intelligence` | 11 | Analysis projections |
| `discovery` | 13 | Snapshots, fingerprints |
| `planning` | 05 | Coverage reuse |
| `runtime-prediction` | 12 | Historical runtime data |
| `plugin:<name>` | Plugin | Plugin-defined |

Consumers: **unlimited**, by region grant.

---

## Service Lifecycle

The Cache Service is initialized during **`CONFIGURING`** and remains available
for the entire execution.

Its lifetime is **not** bound to Document 11's `INTELLIGENCE` phase.

This is architecturally mandatory rather than convenient. Document 13 executes
during `DETECTING_CHANGES`, **before** `INTELLIGENCE`. A consumer SHALL NEVER be
required to wait for its region owner's phase to begin, and SHALL NEVER be unable
to start because a later-phase engine has not run.

Document 11's own analysis phase is one tenant of a service it owns.

---

## Architectural Guarantees

**Correctness independence.** A cache miss SHALL change only cost, never a
result. No correctness property may depend on cache state.

**Determinism.** Identical inputs and identical cache state SHALL resolve
identically. Eviction SHALL follow configured policy; random eviction is
prohibited.

**Loose coupling.** Consumers depend on the **region contract**, never on
Document 11. This permits the Service to be extracted into an independently
deployable component in future without altering a single consumer contract.

**Distribution.** The backend is abstracted — in-process, shared local, network
cache, or object storage — and a backend change SHALL require no consumer change.

**Plugin extensibility.** A plugin registers `plugin:<name>` and receives caching
with no core change.

**AI safety.** Sensitivity class is declared per region by its owner, so the
Service enforces masking without understanding any domain. AI modules receive
sanitized reads only (§30, §31).

---

# 40. Execution Artifact Separation

## Purpose

Planning, optimization, and execution are three distinct concerns. Conflating
their artifacts is how a framework loses the ability to replace its optimizer or
to reproduce a run.

This section defines the separation permanently.

---

## Three Artifacts, Three Owners

| Layer | Owner | Artifact | Answers |
| ----- | ----- | -------- | ------- |
| **Intent** | **05** | `test-plan.json` *(canonical)* | What is tested, under what constraints |
| **Advice** | **12** | `execution-optimization-proposal.json` *(optional)* | How to run it faster |
| **Runtime** | **07** | `runtime-schedule.json` *(canonical)* | What actually ran, in what order |

---

## The Governing Rule

> **Ownership of an artifact SHALL NEVER imply authority over runtime
> execution.**

Document 12 owns a proposal. It has no scheduling authority.

**Document 07 is the only runtime execution authority.**

---

## Advisory By Construction

The proposal is advisory **structurally**, not by policy. Policy erodes under
maintenance pressure; structure does not.

A proposal MAY contain only ordering, batching, worker-count hint,
browser-assignment hint, and timeout hint.

A proposal SHALL NEVER contain a test absent from the plan, an added or removed
test, or any change to scope, eligibility, or assertions.

**Verification:** 07 SHALL reject any proposal whose test set is not exactly the
Test Plan's test set. This is a set comparison — mechanically verifiable, requiring
no interpretation.

A proposal is therefore a **constraint-bounded permutation** of the plan. It can
affect speed. It cannot affect correctness.

---

## Null-Safety

**Absence of a proposal is a valid state.**

Where the proposal is absent, rejected, or stale, 07 executes the Test Plan's own
ordering, which 05 guarantees is complete and sufficient.

The framework SHALL remain fully functional with Document 12 removed entirely.

A framework that cannot execute without its optimizer does not have a replaceable
optimizer. This property is what makes optimization genuinely swappable, and what
allows a plugin to supply a competing proposal with no core change.

---

## Reproducibility

Every execution SHALL record a run manifest:

```
(test-plan version, proposal version | null, runtime-schedule version)
```

This reproduces any execution exactly, including one in which no optimizer was
present. Determinism (§3.1) holds whether or not optimization occurred.

---

## Deviation Feedback

07 MAY deviate from a valid proposal for runtime safety without requesting
approval, and SHALL record every deviation with its reason.

Deviations flow 07 → 09 → 10 → 12, making proposal quality measurable across
executions. An optimizer whose proposals are never compared against actual
runtime cannot be shown to help.

---

# 41. Dependency Architecture

## Purpose

Define the permitted dependency direction and demonstrate that the expanded
architecture contains **no circular dependencies**.

---

## Dependency Layers

Dependencies flow **downward only**. No engine depends on a later layer.

```
LAYER 0   Governance
          01 Master Architecture · Architecture_Ownership_Matrix
                        │
LAYER 1   Authority
          02 Decision Engine            ← depends on 01 only
                        │
LAYER 2   Platform Services
          Framework Cache Service (11) · Event Bus · Capability Registry
          available from CONFIGURING · own no phase
                        │
LAYER 3   Knowledge Acquisition
          13 Incremental Discovery  →  03 Discovery  →  04 Knowledge Graph
                        │
LAYER 4   Knowledge Projection
          11 Unified Test Intelligence      ← derived from 04
                        │
LAYER 5   Planning
          05 Test Planning  →  15 AI Planning Review  →  02 decides
                        │
LAYER 6   Generation
          06 Test Generation
                        │
LAYER 7   Optimization        (optional)
          12 Adaptive Execution Optimizer
                        │
LAYER 8   Execution
          07 Execution Engine  ·  08 Self-Healing (02 approves)
          14 Coverage Optimizer at checkpoints → 02 decides
                        │
LAYER 9   Knowledge Return
          09 Reporting  →  10 Learning  →  02 (future executions)
```

---

## Cycle Analysis

Three apparent cycles are resolved by construction:

**11 ↔ 13 — Cache.** 13 (Layer 3) consumes a cache service owned by 11
(Layer 4). This is **not** a cycle: 13 depends on the *region contract* in
Layer 2, not on Document 11's engine phase. The Service is a platform service
initialized at `CONFIGURING` (§39).

**05 ↔ 12 — Optimization.** 12 consumes the Test Plan; 05 never consumes the
proposal. The dependency is unidirectional, and 12's output is advisory input to
07, not to 05.

**10 → 02 — Learning.** 10 supplies historical evidence to 02 for *future*
executions. Within one execution there is no back edge. Across executions, the
edge is mediated by 02 and requires its approval (§32).

---

## Same-Execution Versus Cross-Execution Dependencies

Layering constrains **runtime** dependencies. It does not prohibit consuming
historical knowledge produced by a previous execution.

The two must be distinguished, because conflating them would either forbid
learning entirely or permit genuine runtime cycles.

| | Same-Execution Dependency | Cross-Execution Dependency |
| --- | --- | --- |
| Consumes | Output produced **in this run** | Artifact **persisted by a previous run** |
| Layer rule | **Downward only** — strictly enforced | Exempt from layer ordering |
| Blocks on | The producing engine completing **now** | Nothing; the artifact already exists |
| Cycle risk | **Yes** if upward | **No** — the producer already terminated |
| Example | 05 consuming 04's graph | 12 consuming historical runtime from 10 |

### Same-Execution Rule — Unchanged

An engine SHALL NOT depend on an engine in a **later layer** for output produced
**within the same execution**.

This is absolute. It is what guarantees a terminating, deterministic pipeline, and
this section does not weaken it.

Document 13 therefore does not declare a same-execution dependency on Document 12,
even though 12 consumes 13's metrics — that is a *consumer* relationship, and
consumption is not dependency.

### Cross-Execution Rule — Permitted

An engine MAY consume artifacts persisted by a previous execution **regardless of
layer**, provided:

- The artifact is **immutable and already persisted** before this execution began
- No engine **waits** on a later-layer engine during this execution
- Consumption is **read-only** — the consumer never writes another engine's dataset
- The dependency is **optional** — absence degrades quality, never correctness
- The source and its version are **recorded** for reproducibility

Artifacts eligible for cross-execution consumption:

| Artifact | Owner | Typical cross-execution consumers |
| -------- | ----- | --------------------------------- |
| Learning Database | 10 | 02, 12, 13, 14, 15 |
| Reports and Analytics | 09 | 14, 15 |
| Discovery Snapshots | 13 | 13 (its own prior snapshot) |
| Locator History | 08 | 06, 08, 10 |
| Execution History | 07 | 12, 14 |
| Previous Test Plans | 05 | 15 |
| Previous Graph Versions | 04 | 13 |
| Cache regions | 11 | All region consumers |

### Why This Is Not A Cycle

A cross-execution edge points **backwards in time**, not backwards in the
pipeline. Its producer terminated in a previous run, so nothing in this run can
wait on it.

```
Execution N-1 :  … 09 → 10  ──┐  (persisted, immutable, terminated)
                               │
                               ▼   read-only, optional
Execution N   :  13 → 03 → 04 → 11 → 05 → 15 → 06 → 12 → 07 → 09 → 10
                 └───────── strictly downward within the run ─────────┘
```

No engine in execution N blocks on an engine in execution N.

### Interface References — Also Permitted

A third category exists, distinct from both of the above.

An engine MAY declare a dependency on a **later-layer engine's contract** in
order to know what interface its own output must satisfy. This creates **no
wait** and therefore no cycle: the reference is to a published interface
definition, not to a runtime value.

| Reference | Nature | Cycle? |
| --------- | ------ | :----: |
| 12 → 07 | 12 must emit a proposal 07 can validate; consumes 07's browser, worker, and timeout contracts | No |
| 13 → 06 | 13 must emit a Regeneration Request 06 can act on | No |
| 13 → 11 | Cache region contract — platform service at Layer 2 | No |
| 15 → 14 | 15 consumes 14's coverage gap analysis from the **previous** execution | No |

An interface reference SHALL be read-only, SHALL NOT block, and SHALL NOT require
the later-layer engine to have executed.

### Declared-Dependency Convention

A document's `Depends On` header MAY list an engine it consumes only across
executions, or whose interface it must satisfy. Where the direction is upward,
the document SHALL make the nature explicit — cross-execution, interface
reference, or platform service — so a reader cannot mistake it for a runtime
dependency.

Documents 12, 13, and 15 list upward references of exactly these kinds. None is a
runtime dependency, and none introduces a cycle.

### The Single Test

For any declared dependency, ask one question:

> **Within this execution, must the declaring engine wait for the declared engine
> to finish?**

If **yes**, and the declared engine is in a later layer, it is a **cycle** and is
prohibited.

If **no**, it is a cross-execution read, an interface reference, or a platform
service, and it is permitted.

Runtime layering is not weakened by this section. Waiting upward remains
prohibited without exception.

### Prohibited In All Cases

- Waiting, within one execution, on a later-layer engine
- Writing a dataset owned by another engine, in any execution
- Treating a historical artifact as canonical current truth
- Requiring a cross-execution artifact for correctness — a first execution has no
  history and SHALL still be correct

---

## Verified Properties

✓ Every dependency flows downward
✓ No engine depends on a later layer
✓ Platform services precede every consumer
✓ Advisory engines depend on their subject, never the reverse
✓ Optional engines have no dependents
✓ Learning feeds forward through 02, never backward within a run

---

# 42. Ownership Governance

## Purpose

State the governance rules that make the expanded architecture maintainable for a
ten-year horizon.

---

## Rule 1 — One Primary Responsibility

Every engine SHALL own exactly **one** primary responsibility, as assigned by
`Architecture_Ownership_Matrix.md`.

An engine that acquires a second primary responsibility SHALL be split, or the
second responsibility SHALL be relocated to its owner.

## Rule 2 — Contracts Only

All inter-engine communication SHALL occur through **public contracts** and the
Event Bus (§27).

Prohibited:

- Direct invocation of another engine
- Reading another engine's internal state
- Writing another engine's canonical dataset
- Lateral communication that bypasses 02 for a decision

## Rule 3 — Duplicate Ownership Is Prohibited

Two engines SHALL NEVER implement the same capability.

Where an engine requires a capability it does not own, the specification SHALL
state:

> *"Consumes &lt;capability&gt; through the public contract defined by
> &lt;owner document&gt;."*

Formulas, pipelines, and state machines SHALL NOT be duplicated across documents.

## Rule 4 — Consumers Never Become Owners

Consuming a value grants no right to compute, redefine, or override it.

Specifically: risk, confidence, policy, and conflict resolution are computed by
**02** alone; canonical coverage by **05** alone; canonical application model by
**04** alone; runtime scheduling by **07** alone.

## Rule 5 — Derived Is Never Canonical

A derived projection SHALL NEVER become the source of truth. Document 11 produces
a derived projection of 04; 04 remains canonical.

## Rule 6 — Advisory Means Advisory

An engine designated advisory SHALL NOT approve, reject, block, halt, or skip.
It submits a recommendation to **02**.

Where possible, advisory status SHALL be enforced **structurally** rather than by
policy — as with the proposal set-equality check in §40.

## Rule 7 — Optional Means Removable

An engine designated optional SHALL be removable without affecting correctness.
Its absence SHALL degrade cost or assurance only, and SHALL be reported.

## Rule 8 — Amendment Order

Extending the architecture SHALL follow:

```
1. Amend this document (01)
2. Amend Architecture_Ownership_Matrix.md
3. Amend or add the engine document
4. Verify no ownership conflict, no duplicate authority, no cycle
5. Update SKILL.md registry
```

An engine document SHALL NOT be accepted before steps 1 and 2 are complete. This
sequence exists because engines 11–15 were originally authored against a Master
Architecture that had no knowledge of them, which produced six competing scoring
models, three execution authorities, and four coverage owners.

---

## Governance Verification

Before any architecture release:

✓ All 15 engines represented in §38
✓ Every capability has exactly one owner
✓ Every canonical dataset has exactly one producer
✓ No duplicated formula, pipeline, or state machine
✓ No circular dependency (§41)
✓ Diagrams consistent with the Ownership Matrix
✓ Lifecycle internally consistent (§16)
✓ State machine internally consistent (§17)
✓ Determinism preserved (§3.1)
✓ Optional engines removable without correctness loss

---

# End of Master Architecture
