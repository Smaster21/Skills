
# QA Automation Framework

# Incremental Discovery Engine Specification

**Document:** 13_Incremental_Discovery_Engine.md

**Version:** 4.1

**Status:** Draft

> **Revision 4.1 — W8, Site Explorer boundary.** Whole-application wording
> replaced with the bounded exploration guarantee (`01` §2.2); the `Visual`
> change category marked OPTIONAL — DISABLED BY DEFAULT. Snapshots,
> fingerprinting, deltas, deduplication and the no-re-crawl foundation are
> **unchanged** — they are the mechanism the guarantee relies on.

**Depends On:**

- Architecture_Ownership_Matrix.md
- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 03_Discovery_Engine.md
- 04_Knowledge_Graph.md
- 05_Test_Planning_Engine.md
- 06_Test_Generation_Engine.md
- 11_Unified_Test_Intelligence_Engine.md

**Owns (per Ownership Matrix):**

- Incremental Discovery
- Fingerprinting (live fingerprint comparison)
- Canonical dataset: **Discovery Delta**

**Consumes (never owns):**

| Capability | Owner |
| ---------- | ----- |
| Discovery / Runtime Discovery | 03 |
| Knowledge Graph, Graph Versioning, Graph Diffing | 04 |
| Risk, Confidence, Policy, Final Decision | 02 |
| Incremental Generation (regeneration scope) | 06 |
| Framework Cache Service (mechanism) | 11 |
| Locator Evolution | 08 |

> `12_Adaptive_Execution_Optimizer.md` is **not** a dependency. Document 12
> consumes the Test Plan, which is produced after discovery. Document 12 is a
> downstream **consumer** of this engine's metrics (§21) only. Declaring it as a
> dependency would create a circular dependency.

---

# Purpose

The Incremental Discovery Engine enables the framework to discover
only the portions of an application that have changed since the
previous execution.

Instead of performing a complete application crawl on every run,
the engine compares the current application against previously
stored discovery snapshots and Knowledge Graph versions.

Only affected routes, workflows, components, APIs, and business
flows are rediscovered.

This dramatically reduces execution time while preserving
accuracy and determinism.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architectural Position
5. Design Principles
6. Incremental Discovery Philosophy
7. Discovery Lifecycle
8. Inputs
9. Outputs
10. Snapshot Manager
11. Change Detection Engine
12. Fingerprint Engine
13. Route Delta Analyzer
14. Component Delta Analyzer
15. API Delta Analyzer
16. Workflow Delta Analyzer
17. Knowledge Graph Synchronization
18. Selective Discovery Scheduler
19. Cache Consumption (`discovery` region)
20. Incremental Regeneration Triggers
21. Metrics
22. Event Bus Integration
23. Configuration
24. Quality Gates
25. Summary

---

# 1. Purpose

Traditional discovery performs a full crawl of the application
during every execution.

For large enterprise applications this is expensive,
time consuming, and unnecessary.

The Incremental Discovery Engine minimizes repeated work by
identifying only the portions of the application that have changed.

The engine SHALL:

- Reuse previous discovery results.
- Compare application snapshots.
- Detect structural changes.
- Detect behavioral changes.
- Trigger selective rediscovery.
- Preserve unchanged knowledge.
- Reduce execution time.

---

# 2. Scope

The Incremental Discovery Engine governs:

- Snapshot loading
- Snapshot comparison
- Live fingerprint generation and comparison
- Change detection against the running application
- Route / component / DOM / API / workflow delta detection
- Partial rediscovery scheduling
- Discovery Delta production
- Knowledge Graph synchronization **requests**
- Incremental regeneration **requests**

The engine SHALL NOT:

- Generate Playwright tests
- Execute browsers
- Make planning decisions
- Modify generated tests
- Perform self-healing
- **Modify, version, or diff the Knowledge Graph** (owned by 04)
- **Determine regeneration scope** (owned by 06)
- **Implement a cache store, eviction, or TTL** (owned by 11; 13 owns `discovery` region semantics only)
- **Calculate Risk or Confidence** (owned by 02)
- **Promote locators** (owned by 08)

Those responsibilities belong to their respective owners as defined by the
Architecture Ownership Matrix.

---

# 3. Responsibilities

The Incremental Discovery Engine is responsible for answering:

Has the application changed?

Which pages changed?

Which components changed?

Which APIs changed?

Which workflows changed?

Can discovery be skipped?

Can discovery be partially reused?

Should cached data remain valid?

Should Knowledge Graph nodes be updated?

Should tests be regenerated?

Should planning be rerun?

Should only affected workflows execute?

Every answer must be evidence-based.

---

# 4. Architectural Position

```text
                Previous Execution
                        │
                        ▼
              Discovery Snapshots
                        │
                        ▼
            Incremental Discovery Engine
                        │
      ┌─────────────────┼─────────────────┐
      ▼                 ▼                 ▼
Change Detection   Delta Analysis   Region Validation
      │                 │                 │
      └─────────────────┼─────────────────┘
                        ▼
        Knowledge Graph Synchronization
                        │
                        ▼
          Test Planning / Regeneration
```

The Incremental Discovery Engine operates after
loading historical knowledge but before planning.

---

# 5. Design Principles

The engine follows the architecture principles defined
in the Master Architecture.

Additional principles include:

---

## 5.1 Reuse Before Rediscovery

Always reuse verified knowledge whenever possible.

Rediscover only what has changed.

---

## 5.2 Deterministic Comparison

Given identical snapshots and application state,
the comparison SHALL always produce identical results.

Random sampling is prohibited.

---

## 5.3 Evidence-Based Updates

Every detected change must include supporting evidence.

Example:

Old DOM Hash

↓

New DOM Hash

↓

Difference

↓

Affected Components

↓

Affected Workflow

---

## 5.4 Preserve Valid Knowledge

Existing validated Knowledge Graph entities
shall not be recreated unnecessarily.

Only changed entities may be replaced.

---

## 5.5 Minimal Work Principle

The engine shall always prefer
the smallest valid discovery scope.

Re-discovery of an already-identified surface requires a recorded reason
(`PLAYBOOK` §21): distinct state · distinct authentication context · workflow
transition · validation · explicitly authorized re-discovery. A revisit without a
recorded reason is a defect.

Example:

All Previously Explored Surfaces ❌

Single Workflow ✅

Single Route ✅

Single Component ✅

---

# 6. Incremental Discovery Philosophy

The framework assumes that
most enterprise applications
change only partially between executions.

Instead of repeating discovery across every
previously explored surface,
the framework performs intelligent comparison.

Discovery philosophy:

```text
Previous Snapshot

↓

Current Application

↓

Detect Changes

↓

Classify Changes

↓

Determine Impact

↓

Rediscover Only Required Areas

↓

Update Knowledge Graph

↓

Continue Execution
```

Incremental discovery SHALL always be preferred
unless explicitly disabled.

---

# 7. Discovery Lifecycle

Every incremental discovery request follows
the same lifecycle.

```text
Load Snapshot

↓

Validate Snapshot

↓

Collect Current Metadata

↓

Compare Snapshots

↓

Identify Changes

↓

Calculate Impact

↓

Schedule Incremental Discovery

↓

Update Knowledge Graph

↓

Publish Events

↓

Return Updated Discovery Results
```

Every stage is mandatory.

---

# 8. Inputs

The engine consumes structured inputs only.

Primary inputs include:

- Previous Discovery Snapshot
- Previous Knowledge Graph
- Current Discovery Metadata
- Configuration
- Environment Information
- Cached Fingerprints
- Historical Discovery Metrics
- Runtime Metadata
- Capability Registry
- Execution Context

Every input contract shall contain:

- Schema Version
- Timestamp
- Correlation ID
- Producer
- Validation Status

---

# 9. Outputs

The engine produces:

- Updated Discovery Snapshot
- Changed Routes
- Changed Components
- Changed APIs
- Changed Workflows
- Delta Report
- Updated Knowledge Graph Requests
- Regeneration Requests
- Cache Update Instructions
- Metrics
- Event Bus Notifications

Example:

```text
Discovery Result

Changed Routes:
4

Changed Components:
11

Changed APIs:
2

Changed Workflows:
1

Full Rediscovery:
No

Partial Rediscovery:
Yes

Knowledge Graph Update:
Required
```

The outputs become inputs to the
Planning Engine,
Knowledge Graph,
and Unified Test Intelligence Engine.

---

# End of Part 1



# ------------------------------------------------------------------

# Part 2

# Sections 10–18

# ------------------------------------------------------------------

# 10. Snapshot Manager

## Purpose

The Snapshot Manager is responsible for maintaining versioned
representations of every successful discovery execution.

Rather than treating discovery as disposable runtime data,
the framework stores snapshots that become the baseline for
future incremental discovery.

Snapshots SHALL be immutable once published.

---

## Responsibilities

The Snapshot Manager SHALL:

- Create discovery snapshots
- Load previous snapshots
- Validate snapshot integrity
- Version snapshots
- Archive historical snapshots
- Expire obsolete snapshots
- Provide snapshot comparison inputs

---

## Snapshot Structure

Each snapshot SHALL contain:

```text
Snapshot
{
    snapshotId
    version
    executionId
    applicationId
    environment
    timestamp

    routes
    pages
    components
    apis
    workflows
    technologies
    authentication
    navigation

    fingerprints

    metrics

    checksum
}
```

---

## Snapshot Storage

Example

```text
discovery/

snapshot-v001.json

snapshot-v002.json

snapshot-v003.json

snapshot-v004.json
```

Snapshots SHALL never be overwritten.

---

# 11. Change Detection Engine

## Purpose

The Change Detection Engine determines whether
the application has changed since the previous execution.

It compares current discovery information
against the stored snapshot.

---

## Detection Sources

Changes may originate from:

Routes

DOM Structure

Navigation

Forms

Components

JavaScript Bundles

API Endpoints

Authentication Flow

Headers

Cookies

Service Workers

Feature Flags

---

## Detection Workflow

```text
Load Previous Snapshot

↓

Collect Current Metadata

↓

Normalize

↓

Compare

↓

Detect Changes

↓

Classify Changes

↓

Publish Delta
```

---

## Change Categories

Structural

Behavioral

Visual *(OPTIONAL — DISABLED BY DEFAULT)*

Network

Configuration

Authentication

Infrastructure

Unknown

---

# 12. Fingerprint Engine

## Purpose

The Fingerprint Engine generates stable identifiers
that represent the current state of the application.

Fingerprints allow fast comparison
without performing expensive rediscovery.

---

## Fingerprint Types

Route Fingerprint

Page Fingerprint

DOM Fingerprint

Component Fingerprint

Workflow Fingerprint

API Fingerprint

Navigation Fingerprint

Technology Fingerprint

Asset Fingerprint

Configuration Fingerprint

---

## Fingerprint Example

```text
Page

/dashboard

↓

Fingerprint

F8A6C119

↓

Next Execution

F8A6C119

↓

Unchanged
```

Example

```text
/settings

↓

Old

C193AA

↓

New

F52B81

↓

Changed
```

---

## Fingerprint Rules

Fingerprints SHALL:

Be deterministic

Ignore timestamps

Ignore session identifiers

Ignore dynamic IDs

Ignore tracking parameters

Remain stable across executions

---

# 12A. Delta Analysis Ownership Boundary

Sections 13–16 define **live-application** delta analysis: comparing the
running application against the previous discovery snapshot using
fingerprints. This is the `Fingerprinting` capability granted to 13 by the
Ownership Matrix ("live fingerprint comparison").

Sections 13–16 SHALL NOT perform **graph-to-graph** comparison.

Graph-to-graph comparison is `Graph Diffing`, owned by **04**. This engine
consumes it through the public contract defined by `04_Knowledge_Graph.md`
§18 (Graph Differencing Engine), which produces `graph-diff.json`.

| Comparison | Owner | Input |
| ---------- | ----- | ----- |
| Live application ↔ previous snapshot | **13** | Fingerprints |
| Graph version N ↔ graph version N-1 | 04 | Graph snapshots |

Where the two disagree, 04 is authoritative — it holds the canonical model.
This engine reports the disagreement as evidence and defers.

---

# 13. Route Delta Analyzer

## Purpose

The Route Delta Analyzer identifies changes in application routing by
fingerprint comparison against the previous snapshot.

Structural route relationships within the graph are consumed from 04.

It determines whether routes have been:

Added

Removed

Modified

Redirected

Renamed

---

## Analysis Inputs

Current Routes

Previous Routes

Normalized URLs

Navigation Graph

Knowledge Graph

---

## Delta Output

Example

```text
Added

/settings/profile

Removed

/profile-old

Modified

/admin/users

Redirect

/login
→
/signin
```

---

## Route Impact

Every route change receives

Impact Score

Confidence

Affected Workflows

Affected Tests

Recommended Action

---

# 14. Component Delta Analyzer

## Purpose

Components change more frequently
than application routes.

The Component Delta Analyzer detects
changes inside existing pages.

---

## Supported Components

Buttons

Forms

Inputs

Tables

Menus

Dialogs

Cards

Tabs

Accordions

Navigation

Charts

Widgets

---

## Detected Changes

Added

Removed

Renamed

Moved

Hidden

Disabled

Reclassified

Behavior Changed

---

## Example

```text
Component

Save Button

↓

Role

button

↓

Old Locator

button.save

↓

New Locator

getByRole("button",{name:"Save"})

↓

Update Required
```

---

# 15. API Delta Analyzer

## Purpose

Modern applications change APIs
without changing the UI.

The API Delta Analyzer detects
backend interface changes.

---

## Analysis Scope

HTTP Methods

Endpoints

Headers

Authentication

Request Schema

Response Schema

Status Codes

Payload Size

Latency

---

## Example

```text
Old

POST

/api/orders

↓

New

POST

/api/v2/orders

↓

Breaking Change

Yes
```

---

## Output

Changed APIs

Affected Components

Affected Workflows

Affected Tests

Risk Score

Regeneration Recommendation

---

# 16. Workflow Delta Analyzer

## Purpose

Business workflows evolve over time.

The Workflow Delta Analyzer determines
whether user journeys have changed.

---

## Workflow Sources

Navigation

Routes

Buttons

Forms

API Calls

Knowledge Graph

Historical Discovery

---

## Example

Previous

Login

↓

Dashboard

↓

Reports

Current

Login

↓

MFA

↓

Dashboard

↓

Reports

Workflow Changed

Yes

Impact

Authentication Tests

---

## Output

Changed Workflows

Affected Pages

Criticality

Coverage Impact

Planning Recommendation

---

# 17. Knowledge Graph Synchronization Request

## Purpose

Discovery changes must be reflected inside the Knowledge Graph.

This engine **SHALL NOT** perform that reflection.

Per the Ownership Matrix, `Knowledge Graph`, `Graph Versioning`, and
`Graph Diffing` are owned exclusively by **04**. Graph snapshots are immutable
(04 §5.2, §26) and Discovery never modifies existing graph versions (03 §37).

This engine therefore emits a **Knowledge Graph Synchronization Request** —
the Discovery Delta contract — and 04 applies it by constructing the next
immutable snapshot.

---

## Ownership Boundary

| Action | Owner |
| ------ | ----- |
| Detect live application change | **13** (this engine) |
| Emit Discovery Delta contract | **13** (this engine) |
| Add / update / retire graph entities | 04 |
| Update relationships | 04 |
| Increment graph version | 04 |
| Publish graph update event | 04 |

This engine SHALL NEVER add, update, retire, or version a graph entity,
relationship, or snapshot.

---

## Synchronization Flow

```text
Live Application

↓

Fingerprint Comparison            ← owned here (13)

↓

Discovery Delta                   ← canonical dataset of 13

↓

Submitted to 04                   ← ownership boundary

↓

Graph Diffing                     ← owned by 04 (§18)

↓

Entity / Relationship Update      ← owned by 04

↓

New Immutable Snapshot + Version  ← owned by 04

↓

KnowledgeGraphUpdated event       ← published by 04
```

---

## Discovery Delta Contract

Consumes graph comparison through the public contract defined by
`04_Knowledge_Graph.md` §18 (Graph Differencing Engine).

Every Discovery Delta SHALL contain:

- Schema Version
- Correlation ID
- Execution ID
- Producer (13)
- Consumer (04)
- Previous Snapshot Reference
- Fingerprint Comparison Evidence
- Changed Routes / Components / APIs / Workflows
- Unchanged (reusable) references
- Validation Status

---

## Request Principles

The request SHALL:

Be immutable once emitted

Reference supporting fingerprint evidence

Never assert a graph version

Never instruct entity deletion — only report observed absence

Leave integrity enforcement to 04 §22

---

# 18. Selective Discovery Scheduler

## Purpose

Once changes are identified,
the scheduler determines
what discovery work should execute.

The scheduler minimizes unnecessary crawling.

---

## Scheduling Strategy

Priority Order

Critical Workflows

↓

Changed Routes

↓

Changed Components

↓

Changed APIs

↓

Low-Risk Areas

↓

Unchanged Areas (Skip)

---

## Scheduler Outputs

Discovery Tasks

Priority

Estimated Duration

Required Browser

Dependencies

Expected Artifacts

---

## Example

```text
Application

500 Pages

↓

Changed

6 Pages

↓

Discovery Scope

6 Pages

↓

Reduction

98.8%
```

This selective scheduling is one of the
largest contributors to reducing
overall framework execution time.

---

# End of Part 2



# ------------------------------------------------------------------

# Part 3

# Sections 19–25

# ------------------------------------------------------------------

# 19. Cache Consumption

## Purpose

A valid cache eliminates unnecessary crawling and enables incremental
discovery to execute significantly faster than a full discovery.

Per the Ownership Matrix, the **Framework Cache Service** is owned by **11**,
with **All** engines as consumers, organised as independently owned regions.

**Consumes cache storage, keying, TTL, eviction, tiering, and invalidation
execution through the public contract defined by
`11_Unified_Test_Intelligence_Engine.md` §18 (Framework Cache Service).**

This engine SHALL NOT implement a cache store, eviction policy, TTL, or backend.

---

## Region Ownership — `discovery`

This engine is the **semantic owner** of the `discovery` region.

| Layer | Owner |
| ----- | ----- |
| Storage, keying, TTL, eviction, tiering, backend, distribution | 11 |
| Invalidation execution | 11 |
| **`discovery` key derivation** | **13** (this engine) |
| **`discovery` validity predicate** | **13** (this engine) |
| **`discovery` invalidation triggers** | **13** (this engine) |
| **`discovery` retention and sensitivity class** | **13** (this engine) |

Region content: discovery snapshots (§10) and fingerprints (§12).

Owning region *semantics* is not owning a cache. This engine defines what makes a
`discovery` entry valid; 11 stores it and evicts it.

---

## Lifecycle Guarantee

This engine executes **before** 11. The Cache Service is nonetheless available,
because it is a **framework-scoped platform capability initialized during
`CONFIGURING`** — not a component of 11's pipeline phase.

This engine SHALL NEVER be blocked by 11's execution phase not having begun, and
SHALL NEVER depend on 11 directly. It depends only on the region contract.

---

## Cache Is Never Truth

The cache SHALL never become the source of truth.

The Knowledge Graph (04) remains the canonical representation of the application.

A cache miss SHALL change only cost, never a result. Where the `discovery`
region is empty, unavailable, or invalid, this engine SHALL fall back to full
discovery per §26, and the outcome SHALL be identical.

What this engine retains independently is **snapshot integrity validation**
(§10), which is intrinsic to the Discovery Delta it owns.

---

## Cache Types Consumed

Through 11's contract, this engine may read:

- Route Maps
- Navigation Trees
- Component Inventory
- DOM Fingerprints
- API Inventory
- Workflow Graphs
- Authentication Metadata
- Technology Stack
- Static Assets
- Discovery Metrics

---

## Cache Validation Criteria Supplied To 11

This engine supplies the fingerprint evidence that 11's Cache Service uses to
validate. 11 performs the validation and returns the verdict.

Criteria evaluated:

- Cache Version
- Framework Version
- Environment
- Application Identifier
- Fingerprint Consistency
- Configuration Compatibility
- Snapshot Integrity

If validation fails,
the cache SHALL NOT be used.

---

## Cache Invalidation Triggers Reported To 11

This engine **reports** the following conditions. 11 performs invalidation.

Invalidation is warranted when:

- Application fingerprint changes
- Major route changes detected
- Technology stack changes
- Framework version incompatible
- Snapshot corruption detected
- User forces full discovery
- Cache expiration reached

---

## Cache Lifecycle

The lifecycle below is executed by **11**. This engine participates only at the
marked steps.

```text
Create Cache                      ← 11

↓

Validate Cache                    ← 11, using fingerprints supplied by 13

↓

Reuse Cache                       ← 11

↓

Monitor Changes                   ← 13 (fingerprint comparison)

↓

Report Invalidation Trigger       ← 13

↓

Invalidate                        ← 11

↓

Regenerate Cache                  ← 11
```

---

# 20. Incremental Regeneration Triggers

## Purpose

Discovery changes do not always require complete regeneration of the
automation project.

Per the Ownership Matrix, `Incremental Generation` is owned by **06**
(*"Regeneration owner"*), with 13 as consumer.

This engine therefore **emits a Regeneration Request carrying change evidence
and impact analysis**. It SHALL NOT determine regeneration scope.

**Consumes regeneration scope determination through the public contract defined
by `06_Test_Generation_Engine.md` §31 (Incremental Generation).**

| Action | Owner |
| ------ | ----- |
| Detect change, produce impact evidence | **13** (this engine) |
| Emit Regeneration Request | **13** (this engine) |
| Determine regeneration scope | 06 |
| Approve regeneration | 02 |
| Execute regeneration | 06 |
| Locator history and promotion | 08 |

---

## Trigger Sources

Changes detected in:

- Routes
- Components
- APIs
- Workflows
- Authentication
- Navigation
- Feature Flags
- Page Classification
- Risk Classification

---

## Regeneration Request Contract

The request reports **what changed and what it affects**. It does not prescribe
what 06 should regenerate.

Every Regeneration Request SHALL contain:

- Schema Version
- Correlation ID / Execution ID
- Producer (13) / Consumer (06)
- Changed entities, by category
- Affected workflows and pages, with evidence
- Impact severity, as observed change magnitude
- Discovery Delta reference
- Validation Status

Regeneration **scope levels are defined by 06** and are deliberately not
restated here. Duplicating them would create a second scope authority and would
violate the No Duplicate Algorithms principle.

Locator changes are reported as observations only. `Locator Evolution` is owned
by **08**, and locator promotion requires 08's repeated-success rule plus 02's
approval.

---

## Decision Flow

```text
Discovery Delta

↓

Impact Analysis

↓

Regeneration Scope

↓

Decision Engine Approval

↓

Generation Engine

↓

Validation

↓

Execution
```

---

# 21. Metrics

## Purpose

The Incremental Discovery Engine produces
operational metrics that quantify
discovery efficiency.

These metrics support optimization
and historical analysis.

---

## Core Metrics

Discovery Duration

Comparison Duration

Fingerprint Generation Time

Snapshot Load Time

Snapshot Save Time

Changed Routes

Changed Components

Changed APIs

Changed Workflows

Cache Hit Rate

Cache Miss Rate

Rediscovery Percentage

Discovery Reduction %

Knowledge Graph Update Time

Regeneration Scope

---

## Example

```text
Discovery Summary

Previous Pages:
612

Changed Pages:
11

Pages Reused:
601

Discovery Reduction:
98.2%

Execution Time Saved:
3h 46m
```

---

## Metric Consumers

Metrics are consumed by:

- Reporting & Analytics
- Adaptive Execution Optimizer
- AI Learning Repository
- Unified Test Intelligence Engine
- Decision Engine

---

# 22. Event Bus Integration

## Purpose

The Incremental Discovery Engine communicates
through the Framework Event Bus.

No component shall invoke another
through direct coupling.

---

## Published Events

DiscoverySnapshotLoaded

SnapshotValidationPassed

SnapshotValidationFailed

FingerprintGenerated

FingerprintMismatchDetected

RouteChanged

ComponentChanged

APIChanged

WorkflowChanged

KnowledgeGraphSynchronizationStarted

KnowledgeGraphSynchronizationCompleted

IncrementalDiscoveryStarted

IncrementalDiscoveryCompleted

IncrementalDiscoveryFailed

CacheCreated

CacheInvalidated

RegenerationRequested

---

## Subscribed Events

FrameworkStarted

DiscoveryRequested

KnowledgeGraphLoaded

ConfigurationChanged

EnvironmentChanged

LearningRepositoryUpdated

ExecutionCompleted

---

# 23. Configuration

## Purpose

The Incremental Discovery Engine
shall expose configurable behavior
without changing implementation logic.

---

## Configuration Categories

Discovery

Caching

Snapshots

Fingerprints

Delta Analysis

Knowledge Graph

Events

Metrics

Regeneration

Performance

---

## Example Configuration

```yaml
incrementalDiscovery:
  enabled: true

  cache:
    enabled: true
    ttlHours: 72

  snapshots:
    keepVersions: 20

  fingerprints:
    ignoreDynamicIds: true
    ignoreTrackingParameters: true

  regeneration:
    automatic: true

  fullDiscovery:
    force: false
```

---

## Configuration Priority

Configuration SHALL follow
the framework hierarchy.

CLI

↓

Environment Variables

↓

Project Configuration

↓

Default Configuration

---

# 24. Quality Gates

Before incremental discovery proceeds,
the following conditions SHALL be satisfied.

---

## Pre-Execution Gates

✓ Configuration Valid

✓ Previous Snapshot Available
(or Full Discovery Requested)

✓ Knowledge Graph Loaded

✓ Snapshot Integrity Verified

✓ Environment Compatible

✓ Cache Validated

---

## Runtime Gates

✓ Fingerprints Generated

✓ Fingerprint Stability Verified (§26)

✓ Live Delta Analysis Completed

✓ Impact Evidence Produced

✓ Regeneration Request Emitted
(scope determined by 06, not here)

✓ Discovery Delta Emitted To 04

---

## Post-Execution Gates

✓ Snapshot Saved

✓ Discovery Delta Accepted By 04
(graph update and versioning performed by 04)

✓ Metrics Recorded

✓ Events Published

✓ `discovery` Region Invalidation Triggers Reported To 11

✓ Discovery Report Data Submitted To 09

---

If any mandatory gate fails,
the Decision Engine determines whether to:

- Retry
- Fall back to Full Discovery
- Abort execution

---

# 25. Summary

The Incremental Discovery Engine transforms
application discovery from a full crawl
into an intelligent, incremental process.

Instead of rediscovering every previously explored page,
component, API, and workflow on each execution,
the engine detects only what has changed,
emits a Discovery Delta for 04 to apply,
and requests regeneration from 06.

By leveraging snapshots, fingerprints,
delta analysis, cache validation,
and selective scheduling,
the framework significantly reduces
execution time while maintaining
accuracy, determinism, and auditability.

This engine is a key optimization layer,
working alongside the Discovery Engine,
Knowledge Graph,
Adaptive Execution Optimizer,
and Unified Test Intelligence Engine
to enable enterprise-scale QA automation.

---

# ------------------------------------------------------------------

# Part 4

# Sections 26–27

# ------------------------------------------------------------------

# 26. Fingerprint Stability & Degradation

## Purpose

Fingerprint comparison is only trustworthy when fingerprints are stable across
executions of an unchanged application.

Dynamic applications defeat naive fingerprinting. Client-side re-render churn,
randomised element ids, injected analytics nodes, rotating CSRF tokens, and
lazy-loaded content all mutate the DOM without changing application behaviour.

An unstable fingerprint reports change where none exists. The engine then
schedules unnecessary rediscovery, emits a spurious Discovery Delta, and drives
regeneration of unchanged automation — the exact cost this engine exists to
avoid, with added risk.

This is the most likely real-world failure mode of this engine.

---

## Stability Requirements

Per §12, fingerprints already exclude timestamps, session identifiers, dynamic
ids, and tracking parameters. In addition, fingerprint generation SHALL:

- Exclude nodes marked volatile by configuration
- Exclude content whose only variation is ordering, where order is not semantic
- Normalise whitespace and attribute order before hashing
- Exclude injected third-party subtrees (analytics, chat, ad slots)
- Record which exclusions were applied, as evidence

---

## Volatility Measurement

Before a fingerprint is trusted, the engine SHALL measure its volatility.

```text
Sample the unchanged page N times

↓

Compute fingerprint per sample

↓

Distinct fingerprints = 1  → STABLE

Distinct fingerprints > 1  → UNSTABLE
```

`N` and the sampling interval are configurable. Sampling SHALL be deterministic
in count and order; random sampling is prohibited (§5.2).

---

## Degradation Policy

| Condition | Action |
| --------- | ------ |
| Fingerprint STABLE | Proceed with incremental discovery |
| Fingerprint UNSTABLE for a route | Exclude that route from fingerprint comparison; rediscover it fully; record the exclusion |
| Unstable routes exceed configured threshold | Report insufficient evidence and request full discovery from 02 |
| No previous snapshot (cold start) | Full discovery; no delta is emitted |
| Snapshot integrity check fails | Full discovery; report corruption |

Consistent with §24, the fallback decision — Retry, Fall Back To Full Discovery,
or Abort — belongs to **02**. This engine reports evidence and a
recommendation only.

---

## Reporting Duty

Unstable fingerprints and excluded routes SHALL be reported to **09** so the
execution report states which portions of the application were **not** compared
incrementally, and why.

Silent exclusion is prohibited. A route excluded from comparison SHALL NEVER be
presented as unchanged.

---

# 27. Ownership Compliance

This engine complies with the Architecture Ownership Matrix as follows.

## Owned Here

| Capability | Canonical Dataset |
| ---------- | ----------------- |
| Incremental Discovery | Discovery Delta |
| Fingerprinting (live comparison) | Fingerprint Set |

## Consumed Through Public Contracts

| Capability | Owner | Contract |
| ---------- | ----- | -------- |
| Discovery / Runtime Discovery | 03 | Discovery artifacts |
| Knowledge Graph | 04 | Graph query contract |
| Graph Versioning | 04 | Snapshot contract |
| Graph Diffing | 04 | §18 `graph-diff.json` |
| Risk | 02 | Risk contract |
| Confidence | 02 | Confidence contract |
| Policy / Final Decision | 02 | Decision contract |
| Incremental Generation scope | 06 | §31 |
| Framework Cache Service | 11 | §18 (`discovery` region) |
| Locator Evolution | 08 | Locator history contract |
| Reporting | 09 | Report input contract |

## Explicitly Not Owned Here

This engine SHALL NEVER calculate Risk or Confidence, mutate or version the
Knowledge Graph, perform graph-to-graph diffing, determine regeneration scope,
implement cache storage, promote locators, schedule execution, or make a final
engineering decision.

Every such capability is consumed from its matrix owner.

---

# End of 13_Incremental_Discovery_Engine.md
