# QA Automation Framework

# Decision Engine Specification

**Document:** 02_Decision_Engine.md

**Version:** 4.1

**Status:** Draft

**Depends On:** 01_Master_Architecture.md

---

# Revision History

| Version | Description |
| ------- | ----------- |
| 3.0     | Enterprise decision engine: decision pipeline, evidence (§18), Framework Confidence (§21), rules, audit (§39) |
| 4.1     | **W8 — Site Explorer boundary.** Decision questions, rule categories, permitted actions and examples referencing accessibility, performance and security testing removed. `02` §18 Evidence Objects, §21 Framework Confidence and its fixed weights, §36 evidence records and §39 audit records are **unchanged**. |

---

# Purpose

This document specifies the internal architecture, responsibilities,
decision models, algorithms, governance rules, and execution behavior
of the Decision Engine.

The Decision Engine is the primary reasoning component of the QA
Automation Framework.

No subsystem is permitted to make autonomous engineering decisions
outside the authority of the Decision Engine.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architectural Position
5. Design Principles
6. Decision Philosophy
7. Core Decision Categories
8. Decision Lifecycle
9. Decision Context
10. Inputs
11. Outputs
12. Decision Contracts
13. Non-Goals

---

# 1. Purpose

The Decision Engine exists to ensure that every engineering action
performed by the framework is:

- Deterministic
- Explainable
- Auditable
- Evidence-based
- Reproducible

Rather than allowing individual modules to independently determine
their own behavior, all engineering decisions are centralized.

This guarantees consistent execution across all framework capabilities.

---

# 2. Scope

The Decision Engine governs:

- Discovery decisions
- Planning decisions
- Generation decisions
- Execution decisions
- Retry decisions
- Healing decisions
- Coverage decisions
- Reporting decisions
- Learning recommendations

The Decision Engine does NOT perform these tasks directly.

Instead, it instructs specialized engines.

Example:

Decision Engine
↓

Execution Engine

↓

Run Test

The Execution Engine executes.

The Decision Engine decides.

---

# 3. Responsibilities

The Decision Engine is responsible for answering questions such as:

What should execute next?

Is discovery complete?

Should another page be crawled?

Should additional tests be generated?

Has enough evidence been collected?

Should this locator be healed?

Is retry safe?

Has required coverage been reached?

Is execution complete?

Should execution stop?

Should execution continue?

Should the failure be escalated?

Which workflow has highest priority?

Which browser should execute first?

Should an optional category (Visual, Dashboard, Table) execute?

Should a discovered surface be revisited, and for which recorded reason?

Every decision must produce evidence.

---

# 4. Architectural Position

    User
                       │
                       ▼
               Configuration Layer
                       │
                       ▼
                 Decision Engine
        ┌──────────┼──────────┐
        ▼          ▼          ▼
 Discovery      Planning    Execution
        │          │          │
        ▼          ▼          ▼
 Generation   Self-Healing Reporting

Every engine communicates upward through the Decision Engine.

No engine communicates laterally.

Example:

Discovery Engine

❌ must NOT invoke

Execution Engine

Instead:

Discovery Engine

↓

Decision Engine

↓

Execution Engine

---

# 5. Design Principles

The Decision Engine follows the principles defined by
Master Architecture v3.0.

Additional principles include:

## 5.1 Single Authority

Engineering decisions shall originate from exactly one authority.

No duplicate decision logic shall exist elsewhere.

---

## 5.2 Evidence Before Action

Every decision requires evidence.

Evidence may include:

- Discovery results
- Risk scores
- Metrics
- Historical executions
- User configuration
- Runtime observations

Never act solely on assumptions.

---

## 5.3 Explainability

Every decision shall be explainable.

Example

Decision

Retry Login Test

Reason

Page timeout observed

Evidence

Network idle exceeded threshold

Confidence

96%

---

## 5.4 Reproducibility

Given identical inputs:

The Decision Engine shall produce equivalent outputs.

Randomness is prohibited unless explicitly configured.

---

## 5.5 Safe Failure

When evidence is insufficient:

The engine shall refuse the decision.

Example

Unknown locator

↓

Reject healing

rather than

Guess locator

---

# 6. Decision Philosophy

The framework follows a strict hierarchy.

Observation

↓

Evidence

↓

Analysis

↓

Decision

↓

Action

↓

Validation

↓

Learning

A decision SHALL NEVER skip analysis.

---

# 7. Core Decision Categories

The Decision Engine manages several independent decision domains.

## Discovery Decisions

Examples

Should crawl continue?

Should sitemap be parsed?

Should depth increase?

Should excluded route be visited?

---

## Planning Decisions

Examples

Which workflow first?

Which browser matrix?

Which fixtures?

Parallel or sequential?

---

## Generation Decisions

Examples

Should page object be generated?

Should API tests exist?

Should a visual baseline be captured? *(only when the optional Visual category
is explicitly enabled — `16` §62; never captured by default)*

---

## Execution Decisions

Examples

Should execution pause?

Should browser restart?

Should environment be marked unhealthy?

---

## Retry Decisions

Examples

Retry?

Wait?

Heal?

Fail?

Escalate?

---

## Reporting Decisions

Examples

Generate HTML?

Generate Markdown?

Generate Allure?

Archive artifacts?

---

## Learning Decisions

Examples

Store locator evolution?

Record flaky behavior?

Recommend locator promotion?

---

# 8. Decision Lifecycle

Every decision follows exactly the same lifecycle.

Request

↓

Collect Context

↓

Validate Inputs

↓

Gather Evidence

↓

Evaluate Rules

↓

Compute Confidence

↓

Generate Candidate Decisions

↓

Conflict Resolution

↓

Select Final Decision

↓

Execute

↓

Audit

↓

Persist

↓

Notify Event Bus

No stage may be skipped.

---

# 9. Decision Context

Every decision receives a Decision Context.

The Decision Context represents the current understanding of the system.

Example

DecisionContext

Application

Environment

Execution State

Configuration

Knowledge Graph

Coverage

Risk

Historical Failures

Artifacts

Metrics

Capabilities

Every decision must operate only on its context.

Global mutable state is prohibited.

---

# 10. Inputs

The Decision Engine accepts structured contracts only.

Primary inputs include:

Configuration

Discovery Report

Knowledge Graph

Execution Metrics

Coverage Report

Risk Analysis

Historical Learning

Capability Registry

Artifact Metadata

Diagnostics

Environment Status

User Configuration

Every input must contain:

Schema Version

Timestamp

Correlation ID

Producer

Payload

Validation Result

---

# 11. Outputs

Every decision produces a Decision Result.

Decision Result includes:

Decision ID

Timestamp

Decision Type

Requested By

Confidence

Reason

Evidence

Chosen Action

Alternative Actions

Expected Outcome

Audit Metadata

Event List

Example

Decision

Retry Test

Reason

Rendering delay

Evidence

DOM not stable

Confidence

93%

Action

Retry with adaptive wait

---

# 12. Decision Contracts

Every subsystem communicates using immutable contracts.

Example

Discovery

↓

Decision Request

↓

Decision Engine

↓

Decision Response

↓

Execution

Direct object mutation is prohibited.

All contracts are versioned.

---

# 13. Non-Goals

The Decision Engine shall NOT:

Generate Playwright code

Execute browser automation

Modify Page Objects

Capture screenshots

Generate reports

Interact with browsers

Manage fixtures

Instead, it coordinates the engines that perform those tasks.

---

# Summary

The Decision Engine is the central reasoning authority of the QA Automation Framework.

Every engineering decision shall originate from this engine.

The engine is deterministic, explainable, auditable, evidence-based, and reproducible.

Subsequent sections define:

- Internal architecture
- Rule engine
- Confidence engine
- Risk engine
- Decision graph
- AI reasoning model
- Retry algorithms
- Healing algorithms
- Scheduling algorithms
- Coverage algorithms
- Conflict resolution
- Decision persistence
- Event integration

# ------------------------------------------------------------------

# Section 14 — Internal Architecture

# ------------------------------------------------------------------

# 14. Internal Architecture

The Decision Engine is not a single algorithm.

It is a collection of specialized reasoning subsystems working together
to produce one deterministic engineering decision.

Each subsystem owns one bounded responsibility.

No subsystem shall duplicate the responsibility of another.

The architecture is intentionally modular to support future extensions,
independent testing, plugin integration, and AI-assisted reasoning.

---

## 14.1 Internal Component Diagram

```

```

    Decision Request
                           │
                           ▼
                  Context Builder
                           │
                           ▼
                  Input Validator
                           │
                           ▼
                  Evidence Collector
                           │
                           ▼
                    Rule Engine
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 Risk Engine      Confidence Engine     Policy Engine
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                Candidate Decision Builder
                           │
                           ▼
                Conflict Resolution Engine
                           │
                           ▼
                 Decision Selection Engine
                           │
                           ▼
                 Decision Audit Generator
                           │
                           ▼
                 Decision Response Builder
                           │
                           ▼
                      Event Bus
                           │
                           ▼
                  Calling Framework Module

```

---

# 15. Decision Processing Pipeline

Every decision SHALL execute through the following pipeline.

```

Request
↓

Context Build
↓

Validation
↓

Evidence Collection
↓

Rule Evaluation
↓

Risk Analysis
↓

Confidence Calculation
↓

Policy Evaluation
↓

Candidate Generation
↓

Conflict Resolution
↓

Decision Selection
↓

Audit Generation
↓

Persistence
↓

Publish Event
↓

Return Response

```

Each stage is mandatory.

No implementation may bypass the pipeline.

---

# 16. Context Builder

## Purpose

The Context Builder constructs the complete execution context required
for decision making.

The Decision Engine never queries arbitrary modules directly.

Instead, all required information is assembled into a single immutable
Decision Context.

---

## Responsibilities

Collect

Configuration

Execution State

Knowledge Graph

Discovery Results

Coverage

Historical Learning

Runtime Metrics

Environment Status

Capabilities

Feature Flags

User Overrides

Risk Scores

---

## Output

DecisionContext

```

DecisionContext
{
contextId

executionId

correlationId

application

configuration

environment

knowledgeGraph

coverage

risk

learning

metrics

capabilities

runtime

timestamp
}

```

Once created, the DecisionContext SHALL NOT be modified.

---

# 17. Input Validation Engine

## Purpose

Before any reasoning begins, every request must be validated.

Invalid context must never reach the Rule Engine.

---

## Validation Rules

Validate

Schema Version

Required Fields

Timestamp

Correlation ID

Configuration Consistency

Knowledge Graph Availability

Execution State

Capability Registration

Environment Compatibility

---

## Validation Outcomes

PASS

WARNING

FAIL

Warnings may continue.

Failures terminate the decision.

---

## Example

Missing Knowledge Graph

↓

FAIL

↓

Decision Rejected

Reason

Required context unavailable

---

# 18. Evidence Collector

## Purpose

Engineering decisions require evidence.

The Evidence Collector gathers all supporting facts before rules execute.

---

## Evidence Sources

Knowledge Graph

Discovery Reports

Historical Learning

Execution History

Coverage Reports

Browser Metrics

Network Metrics

DOM Analysis

Locator History

Diagnostics

Environment State

Artifacts

Configuration

---

## Evidence Categories

Static Evidence

Examples

Application Routes

Configuration

Capabilities

Browser Matrix

---

Dynamic Evidence

Examples

Current DOM

Network Latency

CPU Usage

Retry Count

Environment Health

---

Historical Evidence

Examples

Flaky Tests

Past Failures

Locator Evolution

Healing Success

Execution Trends

---

## Evidence Object

```

Evidence
{
id

type

source

confidence

timestamp

payload
}

```

Every evidence item receives its own confidence score.

---

# 19. Rule Engine

## Purpose

The Rule Engine evaluates engineering policies.

It contains deterministic business rules.

No AI reasoning occurs here.

---

## Responsibilities

Evaluate

Retry Rules

Healing Rules

Coverage Rules

Planning Rules

Execution Rules

Exploration Rules

Governance Rules

Risk Rules

Reporting Rules

---

## Rule Format

Every rule contains

Rule ID

Priority

Description

Condition

Action

Confidence Modifier

Audit Message

Version

---

Example

```

Rule

ID

RULE-001

Condition

Retry Count < Maximum

Action

Retry

Priority

200

```

---

## Rule Categories

Execution

Discovery

Planning

Generation

Retry

Healing

Coverage

Reporting

Learning

Exploration

Governance

Plugins

---

## Rule Evaluation

Rules execute

Highest Priority

↓

Lowest Priority

The first matching rule SHALL NOT necessarily terminate execution.

Multiple rules may contribute candidate decisions.

---

# 20. Risk Engine

## Purpose

The Risk Engine estimates engineering risk associated with every action.

Testing effort should be proportional to risk.

---

## Risk Inputs

Business Criticality

Historical Failures

Authentication

Financial Operations

Administrative Access

Sensitive Data

Workflow Importance

External Integrations

Configuration

Environment

---

## Risk Outputs

Risk Score

Risk Category

Reason

Evidence

Confidence

Suggested Action

---

## Risk Categories

Critical

High

Medium

Low

Informational

---

## Example

```

Checkout Workflow

Risk Score

98

Category

Critical

Reason

Financial Transaction

Recommended Test Depth

Maximum

```

---

# 21. Confidence Engine

## Purpose

The Confidence Engine estimates confidence for every possible decision.

Confidence is never guessed.

It is calculated.

---

## Confidence Factors

Evidence Quality

Evidence Quantity

Historical Accuracy

Rule Agreement

Environment Stability

Discovery Completeness

Knowledge Graph Completeness

Execution History

---

## Confidence Scale

100

Virtually Certain

95

Extremely High

90

Very High

80

High

70

Moderate

60

Low

Below 60

Insufficient

---

## Confidence Formula

Example

```

Confidence

=

Evidence Quality × 0.35

Historical Reliability × 0.25

Rule Agreement × 0.20

Environment Stability × 0.20

```

Actual implementation may evolve without changing public contracts.

---

# 22. Policy Engine

## Purpose

Policies define organizational constraints.

Unlike rules,

Rules answer

"What should happen?"

Policies answer

"What is allowed?"

---

## Example Policies

No Active Scan in Production

No Data Mutation Without Permission

No CAPTCHA Bypass

No Authentication Bypass

No Unsafe Healing

No Destructive Retry

No Infinite Retries

No Silent Failure

---

## Policy Outcomes

Allow

Reject

Require Approval

Warn

Override

Policies always execute before candidate selection.

---

# 23. Candidate Decision Builder

The framework may generate multiple valid decisions.

Example

Retry

Heal Locator

Restart Browser

Abort Execution

Continue

Each becomes a Decision Candidate.

---

## Candidate Structure

Decision ID

Reason

Confidence

Evidence

Priority

Expected Outcome

Estimated Cost

Estimated Risk

```

DecisionCandidate
{
id
action
confidence
risk
cost
reason
evidence
}

```

---

# 24. Design Principles

Every internal subsystem shall satisfy:

Single Responsibility

Deterministic Behavior

Immutable Inputs

Observable Outputs

Independent Testing

Versioned Contracts

Auditability

Future Extensibility

Plugin Compatibility

No Circular Dependencies

---

# End of Part 2

# ------------------------------------------------------------------
# Section 25 — Conflict Resolution Engine
# ------------------------------------------------------------------

# 25. Conflict Resolution Engine

## Purpose

Multiple subsystems may recommend different actions for the same
engineering situation.

The purpose of the Conflict Resolution Engine is to evaluate every
candidate decision and deterministically select the safest,
highest-value engineering action.

Conflict resolution SHALL NEVER use randomness.

The same context shall always produce the same selected action.

---

## Example

Situation

Login page times out.

Candidate decisions

Retry

Confidence: 92%

Restart Browser

Confidence: 81%

Self-Heal Locator

Confidence: 61%

Abort Execution

Confidence: 45%

The engine evaluates all candidates before selecting one.

---

## Conflict Sources

Conflicts commonly arise between

Rule Engine

Risk Engine

Policy Engine

Learning Repository

AI Recommendation

User Configuration

Plugin Decisions

Historical Execution

No source has absolute authority.

Final authority belongs to the Conflict Resolution Engine.

---

## Conflict Resolution Goals

The selected decision shall maximize

Safety

↓

Correctness

↓

Evidence

↓

Business Value

↓

Execution Efficiency

↓

Performance

Speed alone shall never determine the winner.

---

# 26. Decision Ranking

Each candidate receives a Final Decision Score.

The score is calculated from multiple weighted dimensions.

Example

Final Score

=

Confidence

+

Evidence Quality

+

Risk Adjustment

+

Policy Compliance

+

Historical Reliability

+

Business Priority

-

Execution Cost

---

## Example

| Candidate | Score |
|------------|------:|
| Retry | 94 |
| Heal Locator | 82 |
| Restart Browser | 78 |
| Abort | 41 |

Retry is selected.

---

# 27. Decision Priority Hierarchy

When multiple candidates receive similar scores,
priority SHALL follow the hierarchy below.

Priority Order

1

Safety

2

Policy Compliance

3

Application Integrity

4

Evidence Quality

5

Business Risk

6

Confidence

7

Historical Success

8

Execution Cost

9

Execution Speed

10

User Convenience

Example

A destructive action with high confidence
shall still lose to a safe action.

---

# 28. Tie Resolution

Two decisions may produce identical scores.

Tie breaking SHALL occur in the following order.

1

Higher Evidence Quality

↓

2

Higher Confidence

↓

3

Lower Risk

↓

4

Higher Historical Success

↓

5

Lower Cost

↓

6

Earlier Rule Priority

↓

7

Stable Lexicographical Decision ID

This guarantees deterministic execution.

Random tie breaking is prohibited.

---

# 29. Decision Graph

The Decision Engine models every engineering decision
as a Directed Acyclic Graph (DAG).

Nodes represent

Decision States

Edges represent

Allowed transitions.

Example

Decision Requested

↓

Evidence Gathering

↓

Rule Evaluation

↓

Candidate Generation

↓

Conflict Resolution

↓

Decision Selected

↓

Audit

↓

Publish Event

↓

Completed

No backward edges are permitted
except through explicit rollback rules.

---

# 30. Decision State Machine

Every decision SHALL maintain an internal state.

Allowed states

REQUESTED

↓

VALIDATING

↓

COLLECTING_EVIDENCE

↓

RULE_EVALUATION

↓

RISK_ANALYSIS

↓

CONFIDENCE_CALCULATION

↓

POLICY_VALIDATION

↓

GENERATING_CANDIDATES

↓

RESOLVING_CONFLICTS

↓

SELECTING_DECISION

↓

AUDITING

↓

PERSISTING

↓

PUBLISHING_EVENT

↓

COMPLETED

---

## Failure States

VALIDATION_FAILED

INSUFFICIENT_EVIDENCE

POLICY_REJECTED

RULE_FAILURE

CONFLICT_UNRESOLVED

PERSISTENCE_FAILED

EVENT_PUBLICATION_FAILED

CANCELLED

FAILED

Every transition must be logged.

---

# 31. State Transition Rules

The engine SHALL only move forward.

Example

VALIDATING

↓

COLLECTING_EVIDENCE

↓

RULE_EVALUATION

↓

SELECTING_DECISION

Reverse transitions are prohibited.

Exception

Rollback decisions approved by the
Decision Engine itself.

---

# 32. Rollback Model

Rollback exists only for internal decisions.

Rollback SHALL NOT undo
browser actions already executed.

Rollback may reverse

Pending retries

Queued execution

Candidate selection

Temporary scheduling

Rollback SHALL record

Reason

Timestamp

Operator

Evidence

Previous State

New State

---

# 33. Decision Categories

Every decision belongs to one category.

Categories include

Discovery

Planning

Generation

Execution

Retry

Healing

Coverage

Reporting

Learning

Exploration

Governance

Configuration

Plugins

AI

Infrastructure

The category determines

Applicable rules

Policies

Risk model

Evidence model

Audit template

---

# 34. Decision Severity

Every decision receives a severity.

Severity represents
the engineering impact
if the decision is incorrect.

Levels

Critical

High

Medium

Low

Informational

Example

Clean Up Own Synthetic Test Data

Critical

Retry Click

Low

Skip Optional Category

High

---

# 35. Decision Cost Model

The Decision Engine estimates
the engineering cost
of every action.

Cost includes

Execution Time

CPU

Memory

Network

Browser Restart

Human Review

Risk Exposure

Artifact Storage

The engine prefers
lower-cost decisions
only after
correctness has been satisfied.

---

# 36. Decision Evidence Model

Every selected decision SHALL reference
supporting evidence.

Evidence SHALL include

Evidence ID

Evidence Type

Producer

Timestamp

Confidence

Supporting Facts

Artifacts

Metrics

No decision may exist
without evidence.

---

# 37. Decision Traceability

Every decision receives

Decision ID

Execution ID

Correlation ID

Knowledge Graph Version

Configuration Version

Framework Version

Rule Version

Policy Version

Risk Model Version

These identifiers allow
complete reconstruction
of the engineering process.

---

# 38. Decision Persistence

Every completed decision
is written to the Decision Store.

Example

decision-history/

decision-0001.json

decision-0002.json

decision-0003.json

Each record contains

Inputs

Evidence

Candidates

Final Decision

Confidence

Audit

Execution Metadata

Persistence failures
must never silently continue.

---

# 39. Decision Audit

Every decision SHALL produce
an immutable audit record.

The audit includes

Who requested the decision

When

Why

Evidence

Rules evaluated

Policies checked

Rejected candidates

Winning candidate

Confidence

Execution outcome

Duration

No audit record
may be modified after creation.

---

# 40. Decision Example

Decision Request

↓

Retry Login Test

↓

Evidence

Timeout

↓

Rule

Retry Allowed

↓

Risk

Low

↓

Confidence

95%

↓

Policy

Allowed

↓

Candidates

Retry

Restart Browser

Abort

↓

Winner

Retry

↓

Audit

Persist

↓

Publish Event

↓

Completed

---

# End of Part 3
```
