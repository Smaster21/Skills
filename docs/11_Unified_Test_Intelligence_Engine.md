
# QA Automation Framework

# Unified Test Intelligence Engine Specification

**Document:** 11_Unified_Test_Intelligence_Engine.md

**Version:** 4.0

**Status:** Draft

**Depends On:**

- Architecture_Ownership_Matrix.md
- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 03_Discovery_Engine.md
- 04_Knowledge_Graph.md

**Owns (per Ownership Matrix):**

- Unified Intelligence Projection — *derived projection only*
- Framework Cache Service (region tenancy, unlimited consumers)
- Canonical datasets: **Unified Intelligence Projection**, **Cache Service + Region Registry**

**Consumes (never owns):**

| Capability | Owner |
| ---------- | ----- |
| Discovery, Runtime Discovery | 03 |
| Knowledge Graph, Graph Versioning, Graph Diffing | 04 |
| Risk Calculation | 02 |
| Confidence Calculation | 02 |
| Policy, Conflict Resolution, Final Decision | 02 |
| Test Planning, Test Strategy, Dependency Planning | 05 |
| Test Generation, Duplicate Detection | 06 |
| Execution | 07 |
| Reporting, Analytics | 09 |
| Learning Repository | 10 |
| Execution Optimization, Runtime Prediction | 12 |
| Incremental Discovery, Fingerprinting | 13 |
| Coverage Optimization, Stop Conditions | 14 |

> **Canonical boundary.** The Ownership Matrix states *"11 SHALL NEVER become the
> canonical source"*, and Principle 5 states *"Derived information SHALL NEVER
> become the canonical source. Only the owner may define truth."*
>
> The **Knowledge Graph (04)** is the canonical representation of the application
> under test (01 §8, 04 §5.1). The Unified Intelligence Package produced here is
> a **derived, non-authoritative projection** of 04, optimized for reuse. Where
> the projection and 04 disagree, **04 is correct** and the projection SHALL be
> rebuilt.

---

# Purpose

The Unified Test Intelligence Engine (UTIE) is responsible for performing
a single comprehensive analysis of an application and producing a unified
engineering understanding that can be reused by every downstream testing
capability.

Instead of allowing Functional Testing, Accessibility Testing,
Performance Testing, API Testing, Security Testing, Visual Testing,
and AI Planning to independently analyze the same application,
the Unified Test Intelligence Engine performs analysis once and
shares the resulting intelligence across the framework.

This eliminates duplicate analysis, reduces execution time,
improves consistency, lowers AI token consumption,
and increases engineering accuracy.

---

# Table of Contents

1. Purpose
2. Scope
3. Design Philosophy
4. Architectural Position
5. Responsibilities
6. Core Principles
7. Unified Intelligence Model
8. Inputs
9. Outputs
10. Consumers
11. Execution Lifecycle
12. Intelligence Categories
13. Non-Goals
14. Internal Architecture
15. Intelligence Coordinator
16. Context Builder
17. Intelligence Validator
18. Framework Cache Service
19. Unified Analysis Pipeline
20. Functional Analyzer
21. Component Analyzer
22. Workflow Analyzer
23. API Analyzer
24. Design Principles
25. Capability Relocation Record
26. AI Governance
27. Metrics & Telemetry
28. Event Bus Integration
29. Audit & Traceability
30. Security & Governance
31. Configuration, Quality Gates & Summary
32. Ownership Compliance

---

# 1. Purpose

Traditional automation frameworks repeatedly inspect the same page.

Example

Discovery

↓

Accessibility Scan

↓

Performance Scan

↓

Security Scan

↓

Functional Scan

↓

Visual Scan

↓

API Analysis

↓

AI Planning

Each subsystem repeats DOM inspection,
network inspection,
component discovery,
and page classification.

The Unified Test Intelligence Engine eliminates this duplication.

The application is analyzed once.

All downstream engines may consume the same projection instead of repeating the
analysis — while continuing to consult each capability's owner for authoritative
values.

---

# 2. Scope

The engine SHALL build a unified engineering model for:

- Functional behavior
- UI components
- Business workflows
- API interactions
- Accessibility
- Performance characteristics
- Security observations
- Visual structure
- Runtime behavior
- Application technologies

The engine SHALL NOT generate tests.

The engine SHALL NOT execute tests.

The engine SHALL NOT modify application state.

The engine SHALL NOT:

- **Become the canonical source** — 04 is canonical
- **Own or create relationships** *(04)*
- **Version or diff the Knowledge Graph** *(04)*
- **Calculate Risk or Confidence** *(02)*
- **Define Test Strategy or execution profiles** *(05)*
- **Compute dependencies** *(05)*
- **Compute canonical Coverage** *(05)*
- **Select or deselect tests** *(05 plans, 14 recommends, 02 decides)*
- **Detect application change** *(13)*
- **Predict runtime or estimate execution cost** *(12)*
- **Recommend or apply stop conditions** *(14, decided by 02)*
- **Produce reports or analytics** *(09)*
- **Make an engineering decision** *(02)*

Its responsibility ends after producing the Unified Intelligence projection and
operating the Framework Cache Service.

---

# 3. Design Philosophy

The engine follows one fundamental rule.

Analyze Once

↓

Understand Completely

↓

Reuse Everywhere

The framework shall never repeat expensive analysis unless
the application has changed.

---

# 4. Architectural Position

    Discovery Engine
                           │
                           ▼
                  Knowledge Graph
                           │
                           ▼
        Unified Test Intelligence Engine
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 Test Planning     Test Generation     Execution
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
            Reporting & AI Learning Repository

The Unified Test Intelligence Engine is a **projection layer** between the
Knowledge Graph and downstream engines.

It is not the primary producer of engineering knowledge. **04 is.** This engine
reprojects 04's canonical model into a reuse-optimized form so that downstream
engines need not re-derive the same analysis independently.

Downstream engines MAY consume the projection for efficiency. They SHALL consume
**04** whenever canonical truth is required — for locator identity, relationship
integrity, versioning, or any decision of record.

---

# 5. Responsibilities

The engine is responsible for:

- Projecting 04's canonical model into a reuse-optimized package
- Consolidating discovery results consumed from 03
- Correlating components with APIs, using relationships consumed from 04
- Identifying reusable business components
- Identifying reusable assertion **opportunities** for 06
- Identifying shared workflows already modelled in 04
- Classifying page complexity
- Operating the Framework Cache Service (mechanism only; region semantics belong to region owners)
- Producing reusable, versioned intelligence artifacts

The engine is **not** responsible for, and SHALL NOT perform:

| Task | Owner | Consumption |
| ---- | ----- | ----------- |
| Detecting or scoring engineering risk | 02 | Consumes Risk through the public contract defined by `02_Decision_Engine.md` §20 |
| Calculating confidence | 02 | Consumes Confidence through `02_Decision_Engine.md` §21 |
| Estimating execution cost or duration | 12 | Consumes Runtime Prediction through `12_Adaptive_Execution_Optimizer.md` §26 |
| Owning relationships between entities | 04 | Consumes through `04_Knowledge_Graph.md` §14 |
| Computing coverage | 05 | Consumes through `05_Test_Planning_Engine.md` §18 |
| Generating assertions | 06 | Supplies opportunities; 06 generates |

Downstream engines need not repeat the **projection** work performed here. They
SHALL still consult each capability's owner for authoritative values.

---

# 6. Core Principles

The engine SHALL satisfy the following principles.

Analyze Once

Every application artifact should be analyzed a single time.

Reusable Intelligence

All generated intelligence shall be reusable across multiple engines.

Deterministic

Identical inputs shall produce identical intelligence.

Evidence Based

Every intelligence record shall reference supporting evidence.

Explainable

Every inference shall include reasoning metadata.

Immutable

Generated intelligence shall never be modified in place.

Versioned

Every execution produces a new intelligence snapshot.

---

# 7. Unified Intelligence Model

The engine produces one **derived projection** of the canonical model held by 04.

It is a single, consistent view — but it is not truth of record. Every element
carries the 04 entity identifier it was projected from, so any consumer can
resolve back to canonical state.

Example

Application

↓

Pages

↓

Components

↓

Business Flows

↓

API Dependencies

↓

Security Observations

↓

Accessibility Metadata

↓

Performance Characteristics

↓

Visual Structure

↓

Engineering Recommendations

Every downstream engine consumes the same model.

---

# 8. Inputs

The engine consumes validated contracts from existing framework modules.

Primary inputs include:

| Input | Consumed from |
| ----- | ------------- |
| Discovery Results | 03 |
| Knowledge Graph (canonical) | 04 |
| Graph version and diff | 04 |
| Component Classification | 03, 04 |
| Business Flow Detection | 03, 04 |
| Risk scores | **02** |
| Confidence values | **02** |
| Discovery Delta | 13 |
| Runtime Configuration | Configuration Layer |
| Capability Registry | 01 |
| Historical Learning | 10 |
| Execution Policies | 02 |

All inputs SHALL be immutable.

> **Removed input: Planning Metadata.** This engine executes **before** the Test
> Planning Engine (§11). Consuming planning output here created an impossible
> execution order and a circular dependency. Planning consumes this projection,
> not the reverse.

---

# 9. Outputs

The engine produces a Unified Intelligence Package.

Example

unified-intelligence.json

The package includes:

- Application Summary
- Page Intelligence
- Component Intelligence
- Workflow Intelligence
- API Intelligence
- Security Intelligence
- Accessibility Intelligence
- Performance Intelligence
- Visual Intelligence
- AI Planning Metadata

This package is a **derived projection**, not a knowledge source of record.

The canonical engineering knowledge source for the remainder of the execution
lifecycle remains the **Knowledge Graph (04)**.

Every element of the package SHALL carry the 04 graph version it was projected
from. A package whose source graph version has been superseded is **stale** and
SHALL be rebuilt before use.

---

# 10. Consumers

The following framework components consume Unified Intelligence.

- Test Planning Engine
- Test Generation Engine
- Execution Engine
- Self-Healing Engine
- Reporting & Analytics
- AI Learning Repository
- Future Plugins

No consumer may directly modify Unified Intelligence.

Consumers SHALL treat the projection as an **optimization, not an authority**.
Where a consumer requires canonical truth — entity identity, relationship
integrity, graph version, risk, confidence, coverage, or a decision of record —
it SHALL consult the owning engine directly.

A consumer that treats this projection as canonical has violated Principle 5 of
the Ownership Matrix.

---

# 11. Execution Lifecycle

The engine executes after Discovery
and before Test Planning.

Execution flow

Discovery

↓

Knowledge Graph

↓

Unified Test Intelligence

↓

Planning

↓

Generation

↓

Execution

↓

Reporting

↓

Learning Repository

---

# 12. Intelligence Categories

The engine SHALL produce intelligence in the following categories.

Functional Intelligence

Business Intelligence

Component Intelligence

Workflow Intelligence

API Intelligence

Accessibility Intelligence

Security Intelligence

Performance Intelligence

Visual Intelligence

Runtime Intelligence

Each category shall be independently versioned, and each SHALL carry the source
Knowledge Graph version it was projected from.

> **Removed category: AI Intelligence.** AI does not produce a category of truth.
> Per the Ownership Matrix AI Authority clause, AI recommends, predicts, reviews,
> optimizes, and explains — it never defines a canonical dataset. AI assistance
> within each category above is governed by §26, and every AI-assisted record
> carries evidence plus confidence consumed from 02.

---

# 13. Non-Goals

The Unified Test Intelligence Engine SHALL NOT:

Generate Playwright code

Execute browser automation

Modify locators

Heal failures

Generate reports

Persist learning history

Schedule workers

Perform retries

Make engineering decisions

Those responsibilities remain with the existing framework engines.

---

# Summary

The Unified Test Intelligence Engine serves as the framework's
central intelligence producer.

Its purpose is to eliminate duplicated analysis,
provide a single reusable engineering understanding of the application,
reduce execution time,
improve consistency,
and enable future optimization layers without changing
the core architecture.

---

# End of Part 1



# ------------------------------------------------------------------

# Section 14 — Internal Architecture

# ------------------------------------------------------------------

# 14. Internal Architecture

The Unified Test Intelligence Engine is composed of multiple
specialized intelligence modules.

Each module owns one bounded responsibility.

No module shall duplicate the responsibility of another.

The architecture is designed to maximize reuse,
parallel processing,
incremental analysis,
and deterministic execution.

---

## Internal Component Diagram

```
                    Discovery Results
                           │
                           ▼
                  Intelligence Coordinator
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 Context Builder   Intelligence Validator   Cache Service
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
               Unified Analysis Pipeline
                           │
 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 ▼              ▼              ▼              ▼
Functional   Component      Workflow        API
Analyzer     Analyzer       Analyzer        Analyzer

 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 ▼              ▼              ▼              ▼
Security     Accessibility  Performance     Visual
Analyzer     Analyzer       Analyzer        Analyzer

                           │
                           ▼
                Intelligence Correlation Engine
                           │
                           ▼
               Intelligence Confidence Engine
                           │
                           ▼
                 Intelligence Validation Engine
                           │
                           ▼
                 Unified Intelligence Builder
                           │
                           ▼
                 Unified Intelligence Package
```

---

# 15. Intelligence Coordinator

## Purpose

The Intelligence Coordinator orchestrates the complete
analysis lifecycle.

It schedules every analyzer,
collects results,
tracks dependencies,
and ensures every required intelligence category
is completed before publishing the final package.

The coordinator SHALL NOT perform analysis itself.

---

## Responsibilities

Coordinate

- Functional Analysis
- Component Analysis
- Workflow Analysis
- API Analysis
- Security Analysis
- Accessibility Analysis
- Performance Analysis
- Visual Analysis

Track

- Dependencies
- Execution Order
- Completion Status
- Errors
- Metrics

---

## Coordination Rules

Every analyzer executes independently whenever possible.

Dependent analyzers execute only after
required intelligence becomes available.

Example

```
Component Analysis
        │
        ▼
Workflow Analysis
        │
        ▼
Business Intelligence
```

---

# 16. Context Builder

## Purpose

The Context Builder assembles every required input
into a single immutable Intelligence Context.

No analyzer may directly query another framework module.

---

## Context Includes

Configuration

Knowledge Graph

Discovery Results

Component Metadata

Runtime Metadata

Historical Learning

Capabilities

Execution Policies

Application Metadata

Risk Scores

Environment

Technology Stack

---

## Output

```
IntelligenceContext
{
    contextId
    executionId
    application
    configuration
    discovery
    graph
    capabilities
    runtime
    policies
    learning
    timestamp
}
```

The context SHALL remain immutable
throughout analysis.

---

# 17. Intelligence Validator

## Purpose

Before analysis begins,
every context must be validated.

Incomplete intelligence
shall never propagate downstream.

---

## Validation Rules

Validate

- Schema Version
- Required Fields
- Correlation ID
- Discovery Completion
- Knowledge Graph Integrity
- Configuration Compatibility
- Runtime Environment
- Capability Registration

---

## Validation Results

PASS

WARNING

FAIL

Warnings may continue.

Failures terminate intelligence generation.

---

# 18. Framework Cache Service

## Purpose

Repeated analysis of unchanged pages is expensive. Repeated discovery of
unchanged routes is expensive. Both need caching, and the framework SHALL have
exactly **one** cache implementation.

This engine owns that implementation as the **Framework Cache Service**.

The Service is a platform capability with **unlimited consumers**, organised as
independently owned **regions**.

---

## Two-Layer Ownership

| Layer | Owner | Owns |
| ----- | ----- | ---- |
| **Cache Service** (mechanism) | **11**, exclusively | Storage, keying, TTL, eviction, tiering, backend abstraction, distribution, invalidation execution, metrics, access control |
| **Cache Region** (semantics) | Each consumer, one owner per region | Key derivation, validity predicate, invalidation triggers, retention class, sensitivity class |

The Service SHALL NEVER interpret region content.

A region owner SHALL NEVER implement storage.

This separation is what permits one implementation to serve every consumer
without the Service acquiring knowledge of any consumer's domain.

---

## Region Registry

| Region | Semantic Owner | Content |
| ------ | -------------- | ------- |
| `intelligence` | **11** (this engine) | Page, component, workflow, API, visual analysis projections |
| `discovery` | **13** | Discovery snapshots, fingerprints |
| `planning` | **05** | Coverage reuse |
| `runtime-prediction` | **12** | Historical runtime data |
| `plugin:<name>` | Plugin | Plugin-defined |

Regions are registered, versioned, and access-controlled. Registering a region
requires no change to the Service.

---

## Service Lifecycle — Framework-Scoped

The Cache Service is initialized during **`CONFIGURING`** and remains available
for the entire execution.

Its lifetime is **not** bound to this engine's pipeline phase.

This is mandatory rather than convenient. Document 13 executes **before** this
engine. A consumer SHALL NEVER be required to wait for its region owner's
execution phase to begin, and SHALL NEVER be unable to start because a
later-phase engine has not run.

This engine's own analysis phase is simply one tenant of a service it owns.

---

## Consumption Contract

Consumers interact through the region contract only:

```text
acquire(region, key)            → entry | miss
publish(region, key, payload)   → versioned entry
validate(region, key, evidence) → valid | invalid
reportInvalidation(region, reason, evidence)
```

The Service performs invalidation. The region owner supplies the **trigger and
evidence**; it does not perform the eviction.

Consumers SHALL NEVER call another consumer, and SHALL NEVER depend on this
engine directly — only on the region contract. That indirection is deliberate:
it permits the Service to be extracted into an independently deployable
component in future **without altering a single consumer contract**.

---

## Cache Strategy

Each cached artifact carries:

Region

↓

Key (derived by the region owner)

↓

Artifact Hash

↓

Version

↓

Timestamp

↓

Source Graph Version

↓

Sensitivity Class

↓

Cache Status

Where the hash matches and the region owner's validity predicate holds, the work
SHALL be skipped.

Confidence, where relevant to a validity decision, is **consumed from 02**. The
Service SHALL NOT compute it.

---

## Distributed & Cloud Compatibility

The Service abstracts its backend. Consumers are unaffected by the choice.

| Deployment | Backend |
| ---------- | ------- |
| Single node | In-process store with disk persistence |
| Multi-worker | Shared local store |
| Distributed | Network cache (for example Redis) |
| Cloud | Object storage with a tiered hot layer |

Region keys namespace all writes, so concurrent multi-node writes cannot
collide across regions. A backend change SHALL require no consumer change.

---

## Plugin Support

A plugin registers `plugin:<name>`, declares its validity predicate, retention,
and sensitivity class, and receives caching immediately.

Plugins SHALL NOT access another plugin's region, and SHALL NOT access a core
region except where explicitly granted.

---

## AI Compatibility

Sensitivity class is declared **per region by its owner**, so the Service
enforces masking without understanding any domain.

AI modules receive **sanitized reads**. Secrets, tokens, cookies, session
identifiers, and personal data SHALL NEVER be served to an AI consumer, nor
persisted unmasked, per 01 §30 and 04 §27.

---

## Determinism

Given identical inputs and an identical cache state, resolution SHALL be
identical. Eviction SHALL be deterministic — configured policy only, never
random selection.

A cache miss SHALL NEVER change a result, only its cost. Correctness SHALL NEVER
depend on cache state.

---

# 19. Unified Analysis Pipeline

## Purpose

The pipeline standardizes
how every analyzer processes data.

All analyzers follow identical stages.

---

## Pipeline

Receive Context

↓

Validate Input

↓

Extract Features

↓

Analyze

↓

Generate Intelligence

↓

Assign Confidence

↓

Validate

↓

Return Result

Every analyzer follows
this same pipeline.

---

# 20. Functional Analyzer

## Purpose

Understands application functionality.

It identifies
what users can do.

---

## Detects

Pages

Forms

Buttons

Menus

Dialogs

Search

Tables

CRUD Operations

Navigation

Authentication

Authorization

Business Actions

State Changes

---

## Output

Functional Intelligence

Containing

Business Operations

Critical Components

Supported Actions

Navigation Paths

Reusable Assertions

---

# 21. Component Analyzer

## Purpose

Produces a deep understanding
of every UI component.

Unlike Discovery,
this analyzer focuses on behavior,
relationships,
and reuse.

---

## Ownership Boundary

Relationships between entities are **first-class objects owned by 04**
(04 §14 Relationship Engine). This analyzer SHALL NOT create, modify, or assert
a relationship.

**Consumes relationships through the public contract defined by
`04_Knowledge_Graph.md` §14.**

What this analyzer adds beyond 04 is **reuse and complexity observation** —
counting how often a component recurs, and how complex it is to automate. Those
are projections over 04's relationships, not new relationships.

---

## Projects

Component Types *(consumed from 03, 04)*

Component Roles *(consumed from 03, 04)*

Parent / Child Relationships *(consumed from 04 — not derived here)*

Reusability *(projected here)*

Shared Components *(projected here)*

Framework Components *(projected here)*

Dynamic Components *(projected here)*

Lazy Components *(projected here)*

Shadow DOM presence *(consumed from 03)*

Reusable Patterns *(projected here)*

---

## Output

Component Intelligence

Including

Component IDs *(04 identifiers, not new identity)*

Relationship references *(pointers into 04, not copies of record)*

Reuse Count

Complexity

Confidence *(consumed from 02)*

Source graph version

---

# 22. Workflow Analyzer

## Purpose

Transforms isolated pages
into complete business journeys.

---

## Detects

Authentication

Registration

Checkout

Administration

Reporting

Search

Filtering

Approval

Import

Export

Wizard Flows

Multi-Step Forms

---

## Workflow Output

Workflow ID

Pages

Transitions

Criticality

Dependencies

Risk

Estimated Duration

Coverage

Complexity

Confidence

---

# 23. API Analyzer

## Purpose

Builds an understanding
of backend communication.

---

## Detects

REST APIs

GraphQL

WebSockets

SSE

Polling

Authentication Tokens

Headers

Cookies

Response Schemas

Status Codes

Error Responses

Rate Limits

Caching

---

## Output

API Intelligence

Containing

Endpoint

Purpose

Dependencies

Authentication

Security Metadata

Performance Metrics

Business Mapping

---

# 24. Design Principles

Every analyzer SHALL satisfy

Single Responsibility

Deterministic Execution

Independent Testing

Immutable Inputs

Versioned Outputs

Observable Behavior

Explainable Results

Evidence-Based Analysis

Plugin Compatibility

Future Extensibility

No Circular Dependencies

---

# End of Part 2

# ------------------------------------------------------------------

# Part 3

# Sections 25–31

# ------------------------------------------------------------------

# 25. Capability Relocation Record

## Purpose

Sections 39–44 of an earlier revision specified an **Intelligent Test Strategy
Engine** inside this document, together with test selection, a test dependency
graph, incremental test execution, and coverage-aware stop optimization.

Every one of those capabilities is owned by another engine under the Architecture
Ownership Matrix. Retaining them here created a second strategy authority, a
second dependency model, a third incremental-execution specification, and a
second stop-condition owner.

Those sections have been **removed from this document**. The capabilities were
**not** deleted from the framework — each is fully specified by its owner. This
record exists so the relocation is traceable and the ideas are not lost.

---

## Relocation Table

| Removed section | Capability | Now owned by | Specified in |
| --------------- | ---------- | ------------ | ------------ |
| §39 Intelligent Test Strategy Engine | Test Strategy | **05** | 05 §22 Execution Strategy Engine |
| §40 Adaptive Test Profiles | Execution mode / profile selection | **05** | 05 §22, 01 §16 Execution Modes |
| §41 Dynamic Test Selection | Test value scoring and selection | **14** | 14 §15 Test Value Scoring |
| §42 Test Dependency Graph | Dependency Planning | **05** | 05 §25 Dependency Planning Engine |
| §43 Incremental Test Execution | Change detection and affected-test selection | **13** + **05** | 13 §11–§16, then 05 |
| §44 Coverage-Aware Optimization | Coverage Optimization, Stop Conditions | **14** | 14 §16, §17 |

Additionally, the numeric gap at sections 25–38 in the earlier revision is closed
by Part 3 of this document. No content is missing.

---

## Consumption Statements

This engine **consumes Test Strategy through the public contract defined by
`05_Test_Planning_Engine.md` §22.**

This engine **consumes Dependency Planning through the public contract defined by
`05_Test_Planning_Engine.md` §25.**

This engine **consumes application change detection through the public contract
defined by `13_Incremental_Discovery_Engine.md` (Discovery Delta).**

This engine **consumes Coverage Optimization and Stop Conditions through the
public contract defined by `14_Confidence_Coverage_Optimizer.md`.**

This engine **consumes Runtime Prediction through the public contract defined by
`12_Adaptive_Execution_Optimizer.md` §26.**

---

# 26. AI Governance

## Purpose

This engine performs inference across eight analyzers. AI assists that
inference. AI does not own any of it.

---

## AI Responsibilities

AI MAY:

Suggest component classifications

Suggest workflow groupings

Identify reuse patterns

Identify likely complexity hot spots

Summarize application structure

Propose reusable assertion opportunities for 06

Flag anomalies for human or 02 attention

---

## AI Restrictions

Per the Ownership Matrix AI Authority clause, AI SHALL NEVER:

Override policies

Override risk

Override confidence

Override an engineering decision

Modify a canonical dataset

Change immutable history

Additionally, within this engine AI SHALL NEVER:

Invent an entity that 04 does not contain

Invent a relationship that 04 does not contain

Fabricate evidence or confidence

Assert canonical truth — the projection is derived, never authoritative

---

## Recommendation Contract

Every AI-assisted intelligence record SHALL carry:

- Confidence — **consumed from 02**, never computed here
- Supporting evidence, referencing 04 entities by identifier
- Reasoning summary
- Source graph version
- Alternative classifications, where they exist

An intelligence record without evidence SHALL NOT be published.

---

# 27. Metrics & Telemetry

Every projection SHALL produce metrics:

Projection Duration

Analyzer Duration, per analyzer

Cache Hit Rate

Cache Miss Rate

Pages Analyzed

Pages Served From Cache

Components Projected

Workflows Projected

APIs Projected

Average Classification Confidence (consumed from 02)

Analysis Reduction Percentage

Projection Size

Stale Projection Rebuild Count

Source Graph Version

Metrics are submitted to **09** for reporting and **10** for historical
learning. This engine SHALL NOT compute analytics or trends — `Analytics` is
owned by 09 and historical memory by 10.

---

## Cost Ceiling

The eight analyzers of §19–§23 add work. The framework's savings come from cache
reuse (§18) and from 13's incremental discovery.

For small applications, full projection may cost more than it saves.

Therefore projection scope SHALL be configurable, and the engine SHALL record
projected cost against measured saving so the trade-off is visible rather than
assumed. Where projection cost exceeds measured saving across repeated
executions, the recommendation to reduce analyzer scope SHALL be submitted to
**02**.

---

# 28. Event Bus Integration

This engine communicates exclusively through the framework Event Bus (01 §27).
Direct invocation of another engine is prohibited.

## Published Events

IntelligenceProjectionRequested

IntelligenceProjectionStarted

AnalyzerCompleted

CacheHit

CacheMiss

CacheInvalidated

IntelligenceCorrelationCompleted

IntelligenceValidationPassed

IntelligenceValidationFailed

IntelligenceProjectionCompleted

IntelligenceProjectionStale

IntelligenceProjectionFailed

## Subscribed Events

DiscoveryCompleted

KnowledgeGraphPersisted

KnowledgeGraphUpdated

IncrementalDiscoveryCompleted

ConfigurationChanged

LearningRepositoryUpdated

FrameworkShutdown

`KnowledgeGraphUpdated` SHALL invalidate the projection. A projection built from
a superseded graph version is stale by definition.

Every event SHALL include Event ID, Event Type, Timestamp, Execution ID,
Correlation ID, Producer, Source Graph Version, and Payload Version.

---

# 29. Audit & Traceability

Every projection SHALL produce an immutable audit record containing:

Projection ID

Execution ID / Correlation ID

Framework Version / Configuration Version

**Source Knowledge Graph Version**

Analyzers executed, and analyzers skipped via cache

Cache hit and miss detail

Entities projected, by category

Evidence references into 04

Validation result

Projection duration and measured saving

Audit records SHALL NOT be modified after creation.

The source graph version is mandatory. Without it, a projection cannot be shown
to correspond to any canonical state, and its determinism cannot be verified.

---

# 30. Security & Governance

The projection may contain sensitive application intelligence: authentication
routes, internal APIs, administrative pages, role permissions.

Per 04 §27 and 01 §30, this engine SHALL:

Mask sensitive values before persistence

Never persist secrets, tokens, cookies, or session identifiers

Provide sanitized projections to AI modules

Apply role-based access per cache region

Log access for audit

Sensitive information SHALL NEVER appear in logs, reports, AI prompts, or cached
artifacts.

Every projection SHALL be traceable, auditable, explainable, reproducible, and
versioned.

---

# 31. Configuration, Quality Gates & Summary

## Configuration

Configuration SHALL follow the framework hierarchy (01 §29):

CLI Arguments → Environment Variables → Project Configuration → Defaults

Configurable categories:

Projection scope · Analyzer enablement · Cache TTL · Cache size ·
Staleness policy · Evidence depth · AI assistance · Metrics · Security masking

Hard-coded projection thresholds are prohibited.

---

## Pre-Projection Gates

✓ Discovery completed (03)

✓ Knowledge Graph persisted and validated (04)

✓ Source graph version resolved

✓ Capabilities registered

✓ Configuration valid

---

## Projection Gates

✓ Every enabled analyzer completed or served from cache

✓ Evidence recorded for every intelligence record

✓ Confidence consumed from 02, not computed

✓ Correlation completed

✓ Validation passed

---

## Post-Projection Gates

✓ Projection persisted and versioned

✓ Source graph version stamped

✓ Audit record created

✓ Metrics submitted to 09 and 10

✓ Events published

Failure of any mandatory gate SHALL prevent publication of the projection.
Downstream engines then consume **04 directly**, which is always valid. The
projection is an optimization; its absence SHALL NEVER block execution.

---

## Summary

This engine performs application analysis once and reprojects the Knowledge
Graph into a reuse-optimized package, eliminating duplicated analysis across
functional, component, workflow, API, accessibility, performance, security, and
visual concerns.

It owns exactly two capabilities: the **Unified Intelligence Projection** and
**Cache Management**.

It owns no strategy, no plan, no dependency model, no risk model, no confidence
model, no coverage figure, no stop condition, and no canonical truth. Each is
consumed from its matrix owner.

The Knowledge Graph remains canonical. This engine makes it cheaper to use.

---

# 32. Ownership Compliance

This engine complies with the Architecture Ownership Matrix as follows.

## Owned Here

| Capability | Canonical Dataset |
| ---------- | ----------------- |
| Unified Intelligence Projection *(derived only)* | `unified-intelligence.json` |
| Framework Cache Service | Cache Service + Region Registry |

## Consumed Through Public Contracts

| Capability | Owner | Contract |
| ---------- | ----- | -------- |
| Discovery / Runtime Discovery | 03 | Discovery artifacts |
| Knowledge Graph *(canonical)* | 04 | Graph query contract |
| Relationships | 04 | §14 Relationship Engine |
| Graph Versioning / Diffing | 04 | §17, §18 |
| Risk | 02 | §20 Risk Engine |
| Confidence | 02 | §21 Confidence Engine |
| Policy / Final Decision | 02 | Decision contract |
| Test Strategy | 05 | §22 |
| Dependency Planning | 05 | §25 |
| Canonical Coverage | 05 | §18 |
| Assertion generation | 06 | §25 |
| Duplicate Detection | 06 | §32 |
| Runtime Prediction | 12 | §26 |
| Incremental Discovery / Discovery Delta | 13 | Discovery Delta |
| Coverage Optimization / Stop Conditions | 14 | Coverage Recommendations |
| Reporting / Analytics | 09 | Report input contract |
| Learning Repository | 10 | Learning contract |

## Explicitly Not Owned Here

This engine SHALL NEVER become the canonical source, own or create
relationships, version or diff the Knowledge Graph, calculate Risk or
Confidence, define Test Strategy, compute dependencies, compute canonical
Coverage, select tests, detect application change, predict runtime, recommend
stop conditions, produce reports, or make a final engineering decision.

Its two owned capabilities are the **derived projection** and the
**Framework Cache Service**. Everything else is consumed from its matrix owner.

---

# End of 11_Unified_Test_Intelligence_Engine.md

