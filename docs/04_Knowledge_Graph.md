# QA Automation Framework

# Knowledge Graph Specification

**Document:** 04_Knowledge_Graph.md

**Version:** 4.1

**Status:** Draft

> **Revision 4.1 — W8, Site Explorer boundary.** §13 "Security model" renamed
> "Access control model" and §27 "Security Headers" renamed "Response headers"
> so neither reads as a security-observation capability. §27 remains **governance
> only** — masking, access control, audit, sanitized AI copies. The
> `Accessibility Tree` relationship-evidence value is **retained unchanged**
> (C7): it is locator and self-healing evidence, not accessibility testing.
> The AIC remains owned by this document; `attackSurface` is permanently
> `NOT_PRODUCED` by qa-automation (C1, `PLAYBOOK` §21). No node type, edge type,
> dataset, or ownership was added or removed.

**Depends On:**

- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 03_Discovery_Engine.md

---

# Purpose

This document defines the complete architecture, data model,
governance, storage model, lifecycle, reasoning model, and
engineering responsibilities of the Knowledge Graph.

The Knowledge Graph is the framework's canonical representation
of the Application Under Test (AUT).

It transforms raw discovery information into structured,
queryable engineering knowledge.

Unlike temporary discovery artifacts, the Knowledge Graph
persists relationships, enables reasoning, and supports
future planning, execution, diagnostics, and learning.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architecture Position
5. Design Principles
6. Knowledge Philosophy
7. Core Concepts
8. Graph Lifecycle
9. Graph Context
10. Inputs
11. Outputs
12. Data Contracts
13. Non-Goals

---

# 1. Purpose

The Knowledge Graph exists to answer engineering questions such as:

- What pages exist?
- Which workflows connect them?
- Which components belong to each page?
- Which APIs support each workflow?
- Which user roles access each page?
- Which tests already provide coverage?
- Which locators identify which components?

The graph provides one trusted source of truth for the framework.

---

# 2. Scope

The Knowledge Graph stores structured application knowledge.

It does NOT perform:

- Browser automation
- Test execution
- Code generation
- Crawling
- Screenshot capture
- Report generation

Those responsibilities belong to other framework modules.

The graph only models knowledge.

---

# 3. Responsibilities

The Knowledge Graph SHALL:

- Store discovered entities
- Maintain relationships
- Support graph queries
- Track application evolution
- Version every snapshot
- Provide inputs to Decision Engine
- Support planning
- Support diagnostics
- Support self-healing
- Support reporting
- Support learning

---

# 4. Architecture Position

    Discovery Engine
                           │
                           ▼
                  Discovery Artifacts
                           │
                           ▼
                  Knowledge Graph Builder
                           │
                           ▼
                    Knowledge Graph
          ┌───────────┼────────────┐
          ▼           ▼            ▼
 Decision Engine  Planning   Diagnostics
          │           │            │
          └───────────┼────────────┘
                      ▼
               Learning Repository

The Knowledge Graph acts as the shared intelligence layer
between framework components.

---

# 5. Design Principles

The Knowledge Graph follows all principles defined in
Master Architecture v3.0.

Additional principles include:

## 5.1 Canonical Source

Only one Knowledge Graph exists per execution.

All modules reference the same graph.

---

## 5.2 Immutable Snapshots

Graph snapshots are immutable.

New executions create new graph versions.

Previous versions remain available.

---

## 5.3 Explainable Relationships

Every relationship must have evidence.

Example

Login Button

↓

Submits

↓

Login Form

Evidence

- DOM hierarchy
- Event listener
- Form association

---

## 5.4 Deterministic Modeling

Identical discovery inputs produce
equivalent graph structures.

---

## 5.5 Versioned Knowledge

Every graph snapshot includes:

- Graph Version
- Schema Version
- Framework Version
- Discovery Version
- Timestamp

---

# 6. Knowledge Philosophy

The framework distinguishes between
information and knowledge.

Information

↓

"Button exists."

Knowledge

↓

"This Login Button belongs to the
Authentication workflow and is validated
by Login Smoke Test."

The graph stores engineering knowledge,
not merely HTML structure.

---

# 7. Core Concepts

The graph consists of:

Nodes

Relationships

Properties

Metadata

Evidence

Versions

Every engineering object is modeled as a node.

Relationships connect nodes.

Properties describe nodes.

Evidence justifies every relationship.

---

# 8. Graph Lifecycle

Discovery Completed

↓

Normalize Data

↓

Resolve Entities

↓

Create Nodes

↓

Create Relationships

↓

Validate Graph

↓

Version Snapshot

↓

Persist Graph

↓

Publish Event

↓

Available for Query

Each stage is mandatory.

---

# 9. Graph Context

Every graph snapshot maintains context.

Example

KnowledgeGraphContext

- Graph ID
- Execution ID
- Environment
- Application
- Framework Version
- Discovery Version
- Timestamp
- Schema Version
- Correlation ID

The context ensures complete traceability.

---

# 10. Inputs

The Knowledge Graph consumes structured outputs from
earlier framework stages.

Primary inputs include:

- Discovery Report
- Pages
- Routes
- Components
- Navigation Map
- APIs
- Authentication Model
- Risk Scores
- Configuration
- Environment Metadata
- Capability Registry

Every input must be validated before ingestion.

---

# 11. Outputs

The Knowledge Graph provides structured knowledge to:

- Decision Engine
- Test Planning Engine
- Test Generation Engine
- Diagnostics Engine
- Self-Healing Engine
- Reporting Engine
- Learning Repository

Outputs are exposed through immutable contracts.

---

# 12. Data Contracts

Every interaction with the graph uses versioned contracts.

Minimum contract fields include:

- Schema Version
- Producer
- Consumer
- Timestamp
- Correlation ID
- Payload
- Validation Result

Direct modification of graph internals is prohibited.

---

# 13. Non-Goals

The Knowledge Graph SHALL NOT:

- Execute tests
- Crawl websites
- Generate Playwright code
- Perform AI reasoning
- Retry failures
- Capture artifacts
- Replace the Learning Repository

Its sole responsibility is structured knowledge management.

---

# Summary

The Knowledge Graph transforms raw discovery results into
structured engineering knowledge.

Subsequent sections define:

- Internal architecture
- Node model
- Relationship model
- Graph schema
- Query engine
- Versioning
- Validation
- Persistence
- Change detection
- Graph analytics
- Access control model
- Governance

# ------------------------------------------------------------------

# Section 14 — Relationship Engine

# ------------------------------------------------------------------

# 14. Relationship Engine

## Purpose

The Relationship Engine is responsible for identifying, validating,
and maintaining all relationships between entities stored within the
Knowledge Graph.

Relationships are treated as first-class objects and SHALL contain
their own metadata, version history, confidence score, and evidence.

The Relationship Engine ensures that the graph represents the true
business structure of the application rather than merely storing
independent entities.

---

## Responsibilities

The Relationship Engine SHALL:

- Create relationships
- Update relationships
- Remove obsolete relationships
- Detect duplicate relationships
- Validate relationship integrity
- Calculate relationship confidence
- Preserve relationship history
- Support graph traversal

---

## Relationship Types

Supported relationship types include:

contains

references

owns

extends

implements

calls

triggers

belongs_to

requires

depends_on

navigates_to

redirects_to

opens

submits

renders

authenticates

authorizes

validates

uses

tests

covers

heals

generates

discovers

learns_from

---

## Relationship Object

```json
{
  "relationshipId": "REL-001245",
  "type": "contains",
  "source": "PAGE-001",
  "target": "COMPONENT-012",
  "confidence": 97,
  "evidence": [
    "DOM Analysis",
    "Accessibility Tree"
  ],
  "version": 3,
  "createdAt": "...",
  "updatedAt": "..."
}
```

---

## Validation Rules

Every relationship SHALL satisfy:

- Valid source entity
- Valid target entity
- Supported relationship type
- Confidence score
- Version
- Timestamp
- Supporting evidence

Invalid relationships SHALL be rejected.

---

# 15. Graph Builder

## Purpose

The Graph Builder transforms Discovery outputs into a structured
Knowledge Graph.

Discovery produces observations.

The Graph Builder produces understanding.

---

## Inputs

The Graph Builder consumes:

application.json

pages.json

components.json

routes.json

navigation.json

apis.json

authentication.json

risk-analysis.json

classification.json

business-flows.json

---

## Processing Pipeline

Discovery Output

↓

Normalize

↓

Deduplicate

↓

Create Entities

↓

Create Relationships

↓

Validate

↓

Assign IDs

↓

Generate Graph

↓

Persist Snapshot

---

## Responsibilities

The Graph Builder SHALL:

- Merge duplicate entities
- Normalize names
- Resolve references
- Assign unique IDs *(scope defined in §15.1)*
- Build relationships
- Calculate confidence
- Produce graph metadata

---

## 15.1 Scope Of Identifier Assignment

*"Assign unique IDs"* applies **only to entities this document derives from
application discovery** — facts about the Application Under Test:

- Pages
- Routes
- Components
- Forms
- APIs
- Workflows
- Roles and Permissions
- Navigation elements, modals, and the remaining discovered entity types

### Test Case Identifiers Are Not Assigned Here

> **Test Case nodes reference identifiers minted by
> `06_Test_Generation_Engine`. This document SHALL NOT mint, regenerate, or
> independently assign Test Case IDs.**

A Test Case is a **framework-generated artifact**, not an application fact. It
exists only because `06` created it, and it comes into existence during
`GENERATING` — four lifecycle states after this document is built at
`BUILDING_GRAPH` (`01` §17). An identifier cannot be assigned here for an entity
that does not yet exist.

**Consumes Test Case Identity through the public contract defined by
`06_Test_Generation_Engine.md` §22.1.**

### What This Document Retains

Nothing is removed. This document continues to:

- Hold Test Case nodes in the graph
- Hold coverage relationships such as `Test Case → covers → Checkout` and
  `Test Case → validates → Login`
- Answer coverage queries such as *"Find tests covering Checkout"* (§19)
- Enforce referential integrity over those relationships (§22)

Referencing an identifier is not owning it. This is the same relationship `11`
already holds toward this document's own entity identifiers.

| Concern | Owner |
| ------- | ----- |
| Test Case Identity | **06** §22.1 |
| Test Catalogue | **06** |
| Knowledge Graph representation of Test Cases | **04** (this document) |
| Test Case coverage relationships | **04** (this document) |

### Referential Integrity

A Test Case node whose identifier has been **retired** by `06` (§31.1 Rule 2)
remains a valid graph reference. Retirement is a catalogue condition, not a
deletion, and SHALL NOT be treated as a dangling reference under §22.

---

# 16. Graph Validation Engine

## Purpose

Before a graph is accepted as the canonical application model,
it SHALL be validated.

Invalid graphs SHALL never be persisted.

---

## Validation Rules

Validate:

- Unique entity IDs
- Unique relationship IDs
- No orphan nodes
- No invalid references
- Schema compliance
- Required entity types
- Required metadata
- Version compatibility
- Timestamp validity

---

## Validation Results

PASS

WARNING

FAIL

Warnings may continue.

Failures prevent graph persistence.

---

## Example

Missing Page Entity

↓

Relationship References Page

↓

Validation

FAIL

↓

Graph Rejected

---

# 17. Graph Versioning

## Purpose

Every execution creates a new immutable graph snapshot.

Historical graphs SHALL never be overwritten.

---

## Storage Structure

knowledge/

graph-v1.json

graph-v2.json

graph-v3.json

graph-v4.json

...

---

## Version Metadata

Each snapshot stores:

Graph Version

Execution ID

Correlation ID

Framework Version

Discovery Version

Timestamp

Environment

Graph Hash

---

## Version Rules

Snapshots SHALL be immutable.

Only new versions may be created.

Older versions remain available for comparison.

---

# 18. Graph Differencing Engine

## Purpose

Applications evolve.

The Graph Differencing Engine compares two graph versions and
identifies structural changes.

---

## Detectable Changes

Added pages

Removed pages

Modified pages

New workflows

Removed workflows

Component changes

API changes

Navigation changes

Permission changes

Locator changes

Relationship changes

---

## Example

Graph v15

Contains:

Checkout Page

↓

Graph v16

Checkout Page Removed

↓

Difference Report

Type

Breaking Change

Severity

Critical

---

## Difference Output

graph-diff.json

Includes:

Added

Removed

Modified

Unchanged

Impact Analysis

Recommended Actions

---

# 19. Graph Query Engine

## Purpose

Framework modules SHALL query the Knowledge Graph through the
Graph Query Engine.

Direct graph traversal is prohibited.

---

## Example Queries

Find all pages.

Find all forms.

Find login workflow.

Find checkout workflow.

Find reusable components.

Find APIs used by Dashboard.

Find tests covering Checkout.

Find orphan components.

Find inaccessible pages.

Find role-specific pages.

---

## Query Principles

Queries SHALL be:

Deterministic

Read-only

Version-aware

Auditable

Optimized

---

## Example

Query

Find Pages Using Search Component

↓

Query Engine

↓

Returns

Dashboard

Products

Reports

Users

---

# 20. Graph Indexing

## Purpose

Efficient graph traversal requires indexing.

Large enterprise applications may contain
thousands of entities.

---

## Required Indexes

Entity ID

Entity Type

Page Route

Workflow

API

Component Type

Role

Relationship Type

Risk Level

Execution Version

---

## Index Rules

Indexes SHALL update automatically
whenever a new graph snapshot is created.

Indexes SHALL remain immutable
for historical snapshots.

---

# 21. Graph Traversal Engine

## Purpose

The Traversal Engine navigates relationships
between entities.

Traversal SHALL support engineering reasoning
without exposing graph implementation details.

---

## Supported Traversals

Breadth-First Search

Depth-First Search

Shortest Path

Dependency Chain

Workflow Expansion

Component Reachability

Permission Traversal

Coverage Traversal

API Dependency Traversal

---

## Example

Dashboard

↓

Users

↓

User Details

↓

Permissions

↓

Audit Log

Traversal returns
the complete workflow path.

---

# 22. Graph Integrity Rules

The Knowledge Graph SHALL maintain
referential integrity.

Rules include:

Every relationship has valid endpoints.

No duplicate IDs.

No dangling references.

No cyclic workflow dependencies.

Required metadata exists.

Supported entity types only.

Version compatibility maintained.

Integrity SHALL be verified
before persistence.

---

# 23. Graph Metadata

Every graph snapshot SHALL contain
metadata describing its origin.

Required metadata:

Graph ID

Version

Execution ID

Correlation ID

Framework Version

Discovery Version

Environment

Timestamp

Creator

Checksum

Schema Version

---

## Example

```json
{
  "graphId": "GRAPH-00045",
  "version": 12,
  "executionId": "EXEC-2026-0815",
  "frameworkVersion": "3.0",
  "environment": "QA",
  "checksum": "SHA256..."
}
```

---

# 24. Design Principles

The Knowledge Graph SHALL satisfy
the following engineering principles.

Single Source of Truth

Immutable Snapshots

Deterministic Construction

Versioned Evolution

Relationship First

Schema Driven

Evidence Based

Auditable

Queryable

Extensible

Plugin Compatible

Future Proof

---

# End of Part 2

# ------------------------------------------------------------------

# Section 25 — Knowledge Graph Events

# ------------------------------------------------------------------

# 25. Knowledge Graph Events

## Purpose

The Knowledge Graph communicates with the rest of the framework
through the Event Bus.

The graph SHALL never invoke other framework components directly.

Instead, it publishes events whenever the graph changes.

---

## Core Events

KnowledgeGraphBuildingStarted

KnowledgeGraphBuildingCompleted

KnowledgeGraphValidationStarted

KnowledgeGraphValidated

KnowledgeGraphValidationFailed

KnowledgeGraphVersionCreated

KnowledgeGraphUpdated

KnowledgeGraphPersisted

KnowledgeGraphDiffGenerated

KnowledgeGraphQueryExecuted

KnowledgeGraphIntegrityFailed

KnowledgeGraphArchived

---

## Event Structure

Every event SHALL include:

Event ID

Event Type

Timestamp

Execution ID

Correlation ID

Graph Version

Producer

Payload

Metadata

---

## Example

KnowledgeGraphUpdated

↓

Decision Engine

↓

Planning Engine

↓

Generation Engine

↓

Learning Repository

---

# 26. Knowledge Graph Persistence

## Purpose

The graph SHALL be persisted as immutable snapshots.

Historical knowledge must always remain available.

Persistence SHALL support rollback, comparison,
and auditing.

---

## Storage Layout

knowledge/

graph-v1.json

graph-v2.json

graph-v3.json

metadata/

indexes/

history/

diffs/

---

## Persistence Rules

Snapshots SHALL NEVER be modified.

Only new snapshots may be created.

Persistence failures SHALL stop execution.

Successful persistence SHALL publish
KnowledgeGraphPersisted.

---

# 27. Knowledge Graph Security

> **Governance only (W8 / C5).** This section covers secret masking, credential
> handling, access control, audit logging and evidence protection for the graph
> itself. It performs **no** security testing or security assessment of the
> target.

## Purpose

The Knowledge Graph may contain
sensitive application intelligence.

Access SHALL be controlled.

---

## Protected Information

Authentication Routes

Role Permissions

Internal APIs

Admin Pages

Response headers

Tokens

Cookies

Session Information

Environment Details

Infrastructure Metadata

---

## Security Rules

Sensitive values SHALL be masked.

Access SHALL be role-based.

Audit logging SHALL be enabled.

Historical graphs SHALL remain protected.

AI modules SHALL receive sanitized copies
when required.

---

# 28. AI Integration

## Purpose

AI assists in understanding the application,
but it SHALL NOT become the source of truth.

The Knowledge Graph remains authoritative.

---

## AI Responsibilities

Suggest entity classifications.

Recommend relationships.

Identify missing workflows.

Estimate confidence.

Detect anomalies.

Recommend graph improvements.

Summarize application structure.

---

## AI Restrictions

AI SHALL NOT:

Create unsupported entities.

Delete graph data.

Modify persisted snapshots.

Invent workflows.

Fabricate relationships.

Override validation failures.

Ignore governance rules.

Every AI recommendation SHALL include:

Confidence

Evidence

Reasoning

Alternative Suggestions

---

# 29. Plugin Integration

## Purpose

Plugins may contribute additional knowledge
without modifying the graph directly.

---

## Plugin Capabilities

Add custom entities.

Add relationships.

Provide metadata.

Create indexes.

Contribute validators.

Contribute queries.

Publish graph events.

---

## Plugin Rules

Plugins SHALL use:

Public APIs

Contracts

Event Bus

Plugins SHALL NOT:

Modify internal graph state.

Access private storage.

Bypass validation.

Replace existing entities directly.

---

# 30. Quality Gates

Before the Knowledge Graph is accepted,
the following gates SHALL pass.

✓ Discovery Completed

✓ Required Entities Created

✓ Required Relationships Created

✓ Graph Validated

✓ Integrity Verified

✓ Schema Validated

✓ Version Assigned

✓ Metadata Generated

✓ Snapshot Persisted

✓ Events Published

Failure of any mandatory gate
prevents downstream execution.

---

# 31. Metrics & Telemetry

Every graph generation SHALL produce metrics.

Examples include:

Entity Count

Relationship Count

Workflow Count

API Count

Component Count

Average Relationship Confidence

Validation Duration

Graph Build Duration

Traversal Performance

Query Performance

Index Size

Snapshot Size

Graph Growth Rate

Difference Size

Integrity Errors

Metrics SHALL be stored
for historical analysis.

---

# 32. Governance Rules

The Knowledge Graph SHALL comply
with framework governance.

Every graph SHALL be:

Traceable

Versioned

Auditable

Deterministic

Explainable

Reproducible

Evidence-Based

Schema Validated

Every snapshot SHALL receive:

Graph ID

Execution ID

Correlation ID

Framework Version

Schema Version

Timestamp

Checksum

---

# 33. Future Evolution

The architecture SHALL support
future capabilities without redesign.

Examples include:

Distributed Knowledge Graphs

Cloud Synchronization

Graph Databases

Semantic Search

Natural Language Querying

AI-Assisted Refactoring

Cross-Application Knowledge

Multi-Tenant Graphs

Real-Time Graph Updates

Graph Analytics

Digital Twins

Ontology-Based Reasoning

---

# 34. Success Criteria

The Knowledge Graph is considered successful
when it enables:

✓ Complete application representation

✓ Accurate entity classification

✓ Reliable relationship modeling

✓ Workflow understanding

✓ Efficient graph querying

✓ Risk-aware planning

✓ Test coverage mapping

✓ Historical version comparison

✓ AI-assisted reasoning

✓ Plugin extensibility

✓ Enterprise governance

✓ Future scalability

without requiring architectural redesign.

---

# Summary

The Knowledge Graph is the framework's
canonical representation of the application.

It transforms raw discovery results into
structured engineering knowledge that powers
planning, test generation, execution,
diagnostics, reporting, and continuous learning.

The graph is immutable, versioned,
auditable, deterministic, and extensible,
serving as the single source of truth for
all reasoning performed by the QA Automation Framework.

---

# End of 04_Knowledge_Graph
