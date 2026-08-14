
# Architecture Ownership Matrix

**Document:** Architecture_Ownership_Matrix.md

**Version:** 4.0

**Status:** Draft

**Purpose**

This document defines the single authoritative ownership of every
major engineering responsibility within the QA Automation Framework.

Its purpose is to eliminate duplicated responsibilities,
conflicting algorithms, multiple authorities,
and architectural ambiguity.

Every framework document SHALL comply with this ownership matrix.

If any document conflicts with this matrix,
this matrix SHALL take precedence until
the Master Architecture is updated.

---

# Design Principles

The framework follows five ownership principles.

## 1. Single Ownership

Every engineering capability SHALL have exactly one owner.

Only the owner may implement the capability.

---

## 2. Multiple Consumers

Multiple engines MAY consume information.

Consumers SHALL NEVER become owners.

---

## 3. No Duplicate Algorithms

Algorithms SHALL exist only once.

Shared algorithms SHALL be exposed through
public contracts.

---

## 4. Decision Authority

Every engineering decision SHALL originate
from exactly one engine.

---

## 5. Derived Information

Derived information SHALL NEVER become
the canonical source.

Only the owner may define truth.

---

# Ownership Matrix

| Capability                      | Owner                               | Consumers      | Notes                          |
| ------------------------------- | ----------------------------------- | -------------- | ------------------------------ |
| Framework Governance            | 01 Master Architecture              | All            | Defines architecture and rules; owns the framework identifier set (§30), including **Execution Scope Identity** (§30.1) |
| Capability Health               | 01 Capability Registry              | All            | Observed capability state; transitions at deterministic checkpoints only |
| Engineering Decisions           | 02 Decision Engine                  | All            | Sole decision authority        |
| Framework Confidence            | 02 Decision Engine                  | 05,08,12,14,15 | Canonical; engine-local scores permitted (see Confidence Ownership) |
| Risk Calculation                | 02 Decision Engine                  | 05,07,12,14,15 | Only one risk model            |
| Policy Validation               | 02 Decision Engine                  | All            | Centralized                    |
| Conflict Resolution             | 02 Decision Engine                  | All            | Centralized                    |
| Discovery                       | 03 Discovery Engine                 | 04,05,11,13    | Discovers application          |
| Runtime Discovery               | 03 Discovery Engine                 | 13             | Live discovery only            |
| Knowledge Graph                 | 04 Knowledge Graph                  | All            | Canonical application model    |
| Graph Versioning                | 04 Knowledge Graph                  | 13             | Immutable snapshots            |
| Graph Diffing                   | 04 Knowledge Graph                  | 13             | Graph-to-graph comparison      |
| Test Planning                   | 05 Test Planning Engine             | 12,15          | Creates Test Plan              |
| Test Strategy                   | 05 Test Planning Engine             | 15             | Single strategy owner          |
| Dependency Planning             | 05 Test Planning Engine             | 12             | Planning only                  |
| Resource Planning               | 05 Test Planning Engine             | 12             | Initial allocation             |
| Test Generation                 | 06 Test Generation Engine           | 07             | Generates automation           |
| Duplicate Test Detection        | 06 Test Generation Engine           | 14             | Pre-generation optimization    |
| Incremental Generation          | 06 Test Generation Engine           | 13             | Regeneration owner             |
| Execution                       | 07 Execution Engine                 | 12             | Executes tests only            |
| Browser Management              | 07 Execution Engine                 | 12             | Runtime browser lifecycle      |
| Worker Scheduling               | 07 Execution Engine                 | 12             | Runtime scheduling             |
| Timeout Management              | 07 Execution Engine                 | 12             | Runtime timeouts               |
| Self-Healing                    | 08 Self-Healing Engine              | 07             | Locator recovery               |
| Locator Evolution               | 08 Self-Healing Engine              | 10             | Canonical locator history      |
| Reporting                       | 09 Reporting & Analytics            | 14,15          | Produces reports               |
| Analytics                       | 09 Reporting & Analytics            | All            | Historical analytics           |
| Learning Repository             | 10 AI Learning Repository           | All            | Historical memory              |
| Unified Intelligence Projection | 11 Unified Test Intelligence Engine | All            | Derived projection only        |
| Framework Cache Service         | 11 Unified Test Intelligence Engine | All            | One service, region tenancy    |
| Execution Optimization          | 12 Adaptive Execution Optimizer     | 05,07          | Advisory proposal only; never runtime authority |
| Batch Planning                  | 12 Adaptive Execution Optimizer     | 07             | Execution batching             |
| Runtime Prediction              | 12 Adaptive Execution Optimizer     | 07             | Predictive optimization        |
| Incremental Discovery           | 13 Incremental Discovery Engine     | 03,04          | Detects application changes    |
| Fingerprinting                  | 13 Incremental Discovery Engine     | 04             | Live fingerprint comparison    |
| Coverage Optimization           | 14 Confidence & Coverage Optimizer  | 05,09          | Optimization only              |
| Stop Condition Optimization     | 14 Confidence & Coverage Optimizer  | 02             | Recommendation only            |
| AI Planning Review              | 15 AI Planning Review Engine        | 02             | Advisory review only           |

---

# Canonical Data Ownership

Only one document may own
each canonical dataset.

| Dataset                          | Owner |
| -------------------------------- | ----- |
| Knowledge Graph                  | 04    |
| Test Plan                        | 05    |
| Generated Tests                  | 06    |
| Execution State                  | 07    |
| **Runtime Schedule**             | **07**    |
| Locator History                  | 08    |
| Reports                          | 09    |
| Learning Database                | 10    |
| Unified Intelligence Projection  | 11    |
| **Cache Service + Region Registry** | **11** |
| **Execution Optimization Proposal** | **12** |
| Discovery Delta                  | 13    |
| Coverage Recommendations         | 14    |
| Planning Review                  | 15    |
| **Application Intelligence Contract** | **04** |

Ratified by **W7-A**, Board decision **BD-W7-1**.
The **Application Intelligence Contract (AIC)** is the versioned, immutable,
cross-skill interoperability serialization of the canonical application
intelligence owned by **04** — pages, routes, journeys, forms, parameters, APIs,
API-calls, auth surfaces, hidden/observed endpoints, relationships, and a
*derived* attack-surface routing projection, each evidence-linked and
provenance-tagged. It is a serialization/contract of `04`'s model, **not** a
second Knowledge Graph, a second discovery authority, a second intelligence
model, an offensive finding store, a vulnerability database, a second Evidence
Quality model, or a second Framework Confidence model. It carries no CVSS and no
confirmed-vulnerability status. Its schema is defined by **W7-B**; the `04`
model/node-edge extensions that populate it are defined by **W7-D**. `11` MAY
cache or project it; `11`'s projection remains derived and never canonical.

---

# Cache Region Ownership

There is exactly **one** cache implementation, owned by **11** as the Framework
Cache Service.

The Service owns the **mechanism**: storage, keying, TTL, eviction, tiering,
backend abstraction, distribution, invalidation execution, metrics, access
control.

Each **region** has exactly one semantic owner, which owns key derivation, the
validity predicate, invalidation triggers, retention class, and sensitivity
class.

The Service SHALL NEVER interpret region content.
A region owner SHALL NEVER implement storage.

| Region              | Semantic Owner | Content                    |
| ------------------- | -------------- | -------------------------- |
| `intelligence`      | 11             | Analysis projections       |
| `discovery`         | 13             | Snapshots, fingerprints    |
| `planning`          | 05             | Coverage reuse             |
| `runtime-prediction`| 12             | Historical runtime data    |
| `plugin:<name>`     | Plugin         | Plugin-defined             |

## Service Lifecycle

The Cache Service is a **platform capability**. Its lifetime is framework-scoped:
initialized during `CONFIGURING` and available for the whole execution.

It is **not** scoped to 11's pipeline phase. Document 13 executes before 11, and
a region consumer SHALL NEVER depend on its region owner's execution phase having
begun.

Consumers depend on the **region contract**, never on 11 directly. This permits
the Service to be extracted to an independently deployable component in future
without altering any consumer contract.

---

# Decision Ownership

Only one engine may calculate:

| Decision                     | Owner |
| ---------------------------- | ----- |
| **Framework Confidence**     | 02    |
| Risk                         | 02    |
| Policy                       | 02    |
| Conflict Resolution          | 02    |
| Final Engineering Decision   | 02    |

Every other engine SHALL submit
recommendations only.

An engine MAY calculate an **engine-local domain score** within its own domain
(see Confidence Ownership). Such a score is evidence submitted to 02. It is never
Framework Confidence, and it never decides.

---

# Coverage Ownership

Coverage SHALL follow
the hierarchy below.

Coverage Measurement

↓

05 Test Planning Engine

Coverage Reporting

↓

09 Reporting & Analytics

Coverage Optimization

↓

14 Confidence & Coverage Optimizer

Coverage Review

↓

15 AI Planning Review Engine

Only 05 computes canonical coverage.

---

# Risk Ownership

Risk SHALL follow
the hierarchy below.

Risk Calculation

↓

02 Decision Engine

Risk Consumption

↓

05 Planning

↓

07 Execution

↓

12 Optimizer

↓

14 Optimizer

↓

15 Review

Risk SHALL NEVER be recalculated
outside the Decision Engine.

---

# Confidence Ownership

The framework distinguishes **two** kinds of confidence. Conflating them is what
produced multiple competing confidence models.

---

## 1. Framework Confidence — Canonical

Owner

↓

**02 Decision Engine**

Consumers

↓

05 Planning · 08 Healing · 10 Learning · 12 Optimizer · 14 Optimizer · 15 Review

There SHALL be exactly **one** Framework Confidence model, owned by 02.

Framework Confidence is the value that accompanies an engineering decision, and
the only value that MAY be reported as "confidence" in any report, dashboard,
audit record, or AI prompt.

Consumers SHALL NOT implement, redefine, weight, or override Framework
Confidence. They consume it through the public contract defined by
`02_Decision_Engine.md` §21.

---

## 2. Engine-Local Domain Scores — Permitted

An engine MAY calculate an **internal, domain-specific score** where its domain
requires a measure that 02's model does not express.

Examples of legitimate engine-local scores:

| Score | Owner | Domain |
| ----- | ----- | ------ |
| Healing Candidate Score | 08 | Locator recovery viability |
| Test Value Score | 14 | Marginal engineering value of a test |
| Review Score | 15 | Plan quality |
| Risk Score | 02 | *(canonical — not engine-local)* |

### Rules For Engine-Local Scores

An engine-local score SHALL:

- Be named distinctly from "confidence" — a score, not a confidence
- Be scoped to its owning engine's domain
- Consume Framework Confidence from 02 as an input where relevant
- Be documented with its factors and evidence

An engine-local score SHALL NEVER:

- Be published, reported, or exported as Framework Confidence
- Replace, override, or contradict Framework Confidence
- Be used to justify a decision reserved to 02
- Appear in a report labelled simply "confidence"

### Precedence

Where an engine-local score and Framework Confidence disagree, **Framework
Confidence prevails** for every decision. The engine-local score remains valid
only as domain evidence submitted to 02.

---

## Summary

| | Framework Confidence | Engine-Local Score |
| --- | --- | --- |
| Owner | 02 only | The owning engine |
| Count | Exactly one | One per domain, as needed |
| May be reported as "confidence" | ✓ | ✗ |
| May justify a decision | ✓ (via 02) | ✗ (evidence only) |
| Overrides the other | ✓ | ✗ |

---

# Scheduling Ownership

Scheduling is separated into three layers with three owners and one runtime
authority.

| Layer | Owner | Canonical Artifact | Answers |
| ----- | ----- | ------------------ | ------- |
| **Intent** | 05 Test Planning Engine | `test-plan.json` | What is tested, under what constraints |
| **Advice** | 12 Adaptive Execution Optimizer | `execution-optimization-proposal.json` *(optional)* | How to run it faster |
| **Runtime** | 07 Execution Engine | `runtime-schedule.json` | What actually ran, in what order |

Only **07** schedules runtime execution.

---

## Rule — Artifact Ownership Never Implies Runtime Authority

Owning an artifact grants no authority over runtime behaviour.

12 owns the Execution Optimization Proposal. It has no scheduling authority.
07 owns the Runtime Schedule. It alone schedules.

---

## Advisory By Construction

The Proposal is advisory **structurally**, not by policy. Policy decays; structure
does not.

A Proposal MAY contain only:

- Execution ordering
- Batching and grouping
- Worker-count hint
- Browser-assignment hint
- Timeout hint

A Proposal SHALL NEVER contain:

- A test identifier absent from the Test Plan
- An added or removed test
- Altered scope, eligibility, or assertions

**Validation:** 07 SHALL reject any Proposal whose test set is not exactly the
Test Plan's test set. This is a set comparison and is mechanically verifiable.

**Null-safety:** absence of a Proposal is a valid state. 07 SHALL execute the
Test Plan's own ordering. The framework SHALL remain fully functional with the
optimizer removed — this is what makes optimization genuinely replaceable.

A Proposal is therefore a **constraint-bounded permutation** of the Test Plan. It
can affect speed. It cannot affect correctness.

---

## Reproducibility

Every execution SHALL record a run manifest:

```
(test-plan version, proposal version | null, runtime-schedule version)
```

This reproduces any execution exactly, including one in which no optimizer was
present.

---

# Knowledge Ownership

Discovery

↓

03

↓

Knowledge Graph

↓

04

↓

Unified Intelligence

↓

11

↓

Planning

↓

05

↓

Generation

↓

06

↓

Execution

↓

07

11 SHALL NEVER become
the canonical source.

---

# AI Authority

AI MAY

- Recommend
- Predict
- Review
- Optimize
- Explain

AI SHALL NEVER

- Override policies
- Override risks
- Override confidence
- Override engineering decisions
- Modify canonical datasets
- Change immutable history

Final authority remains

↓

02 Decision Engine

---

# Responsibility Rules

Every engine SHALL own

Exactly one primary responsibility.

Every engine MAY

Consume information.

Every engine SHALL NOT

Duplicate another engine.

---

# Validation Rule

Every future document SHALL be reviewed against this matrix.

If ownership conflicts exist,
the document SHALL be revised
before acceptance.

---

# Summary

This Ownership Matrix establishes
clear architectural boundaries
between every framework component.

It eliminates:

- Duplicate responsibilities
- Conflicting algorithms
- Multiple confidence model
- Multiple risk models
- Multiple coverage calculations
- Multiple execution authorities

The result is a deterministic,
maintainable,
enterprise-grade architecture
where every responsibility has
exactly one owner and many consumers.

---

# End of Architecture_Ownership_Matrix.md
