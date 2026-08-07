# QA Automation Framework

# Self-Healing Engine Specification

**Document:** 08_Self_Healing_Engine.md

**Version:** 3.0

**Status:** Draft

**Depends On:**

- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 03_Discovery_Engine.md
- 04_Knowledge_Graph.md
- 05_Test_Planning_Engine.md
- 06_Test_Generation_Engine.md
- 07_Execution_Engine.md

---

# Purpose

This document defines the complete architecture, responsibilities,
healing strategies, governance rules, candidate scoring model,
validation process, and enterprise standards for the
Self-Healing Engine.

The Self-Healing Engine enables the framework to recover from
recoverable automation failures while preserving the original
business intent of every test.

The engine SHALL improve automation stability without masking
real application defects.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architectural Position
5. Design Principles
6. Healing Philosophy
7. Healing Lifecycle
8. Healing Context
9. Inputs
10. Outputs
11. Healing Contracts
12. Supported Healing
13. Unsupported Healing

---

# 1. Purpose

The Self-Healing Engine exists to recover safely from transient,
environmental, or UI-level automation failures.

Its objective is to increase automation resilience while ensuring
that failures caused by genuine application defects remain visible.

Healing SHALL NEVER change the intended business behavior of a test.

---

# 2. Scope

The Self-Healing Engine governs:

- Locator recovery
- Timing recovery
- Rendering recovery
- Detached element recovery
- Dynamic DOM adaptation
- Shadow DOM recovery
- Lazy loading recovery
- Retry-assisted healing
- Candidate locator evaluation
- Healing candidate scoring (engine-local)

The engine SHALL NOT:

- Modify application code
- Ignore failed assertions
- Bypass authentication
- Alter business rules
- Hide backend defects
- Suppress security issues

---

# 3. Responsibilities

The Self-Healing Engine is responsible for answering questions such as:

Can this failure be healed?

Is healing allowed by policy?

Is the failure recoverable?

What caused the locator failure?

Has this locator evolved previously?

Should another locator be attempted?

Is browser restart required?

Should execution continue?

Should healing be rejected?

Every healing action shall produce evidence,
confidence, and a complete audit trail.

---

# 4. Architectural Position

```
                    Decision Engine
                           │
                           ▼
                  Self-Healing Engine
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
 Locator Recovery   Timing Recovery   DOM Analysis
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  Validation Engine
                           │
                           ▼
                  Execution Engine
```

The Self-Healing Engine SHALL operate only after approval from
the Decision Engine.

It SHALL never initiate healing independently.

---

# 5. Design Principles

The Self-Healing Engine follows the architectural principles
defined in the Master Architecture.

Additional principles include:

## 5.1 Preserve Test Intent

Healing SHALL preserve the original business purpose
of the generated automation.

Example

Original Intent

Click Login Button

Acceptable Healing

Updated semantic locator

Unacceptable Healing

Skip login step

---

## 5.2 Safety First

Healing must always prefer safe recovery
over aggressive automation changes.

If the Healing Candidate Score is below the safety floor (§32), or Framework
Confidence from 02 is insufficient, the engine shall reject healing.

---

## 5.3 Explainability

Every healing action shall explain:

- Why healing occurred
- What changed
- Confidence
- Evidence
- Validation results
- Historical references

---

## 5.4 Deterministic Behavior

Given identical inputs,
the healing engine shall produce
identical healing recommendations.

Random healing is prohibited.

---

## 5.5 Auditability

Every healing attempt,
whether successful or rejected,
shall be permanently recorded.

---

# 6. Healing Philosophy

Healing follows the framework's engineering model.

```
Failure

↓

Evidence

↓

Classification

↓

Candidate Healing

↓

Validation

↓

Decision

↓

Retry

↓

Audit

↓

Learning
```

Healing SHALL never occur
before failure classification.

---

# 7. Healing Lifecycle

Every healing request SHALL follow
the same execution lifecycle.

```
Healing Request

↓

Build Healing Context

↓

Validate Inputs

↓

Classify Failure

↓

Generate Candidates

↓

Evaluate Candidates

↓

Compute Healing Candidate Score

↓

Policy Validation

↓

Decision Engine Approval

↓

Apply Healing

↓

Verification

↓

Audit

↓

Learning Update
```

No stage may be skipped.

---

# 8. Healing Context

Each healing request receives
an immutable Healing Context.

The Healing Context contains:

- Execution State
- Failure Details
- Test Metadata
- Locator History
- Knowledge Graph
- DOM Snapshot
- Browser State
- Network State
- Historical Healing
- Runtime Metrics
- Configuration
- Policies

Once created,
the Healing Context SHALL NOT be modified.

---

# 9. Inputs

The Self-Healing Engine accepts
versioned structured contracts only.

Primary inputs include:

- Failure Report
- Diagnostics Report
- DOM Snapshot
- Locator Metadata
- Knowledge Graph
- Execution Metadata
- Browser Metrics
- Artifact Metadata
- Configuration
- Decision Approval

Every input SHALL include:

- Schema Version
- Timestamp
- Correlation ID
- Producer
- Payload
- Validation Status

---

# 10. Outputs

Every healing attempt produces
a Healing Result.

The result includes:

- Healing ID
- Healing Type
- Status
- Framework Confidence *(consumed from 02)*
- Healing Candidate Score *(engine-local, §20)*
- Selected Candidate
- Validation Outcome
- Evidence
- Audit Metadata
- Learning Recommendation

Example

Healing

Locator Recovery

Status

Successful

Framework Confidence (from 02)

96%

Healing Candidate Score (engine-local)

94

Action

Promote Candidate Locator

---

# 11. Healing Contracts

Every subsystem communicates
using immutable contracts.

Example

```
Execution Engine

↓

Healing Request

↓

Self-Healing Engine

↓

Healing Response

↓

Decision Engine

↓

Execution Engine
```

Direct modification of runtime objects
is prohibited.

All contracts SHALL be versioned.

---

# 12. Supported Healing

The framework supports healing for:

- Broken locators
- Delayed rendering
- Timing instability
- Detached elements
- Lazy loading
- Minor DOM restructuring
- Shadow DOM traversal
- Dynamic attributes
- Stable semantic locator replacement
- Recoverable browser timing issues

Every supported healing type
must pass validation before execution.

---

# 13. Unsupported Healing

The framework SHALL reject healing for:

- Failed business assertions
- Incorrect calculations
- Backend defects
- API failures
- Authentication failures
- Authorization failures
- Security vulnerabilities
- Data corruption
- Business logic defects
- Missing application functionality

These failures shall be reported,
not healed.

---

# End of Part 1

# ------------------------------------------------------------------

# Section 14 — Internal Architecture

# ------------------------------------------------------------------

# 14. Internal Architecture

The Self-Healing Engine is composed of multiple specialized
subsystems.

Each subsystem owns exactly one engineering responsibility.

No subsystem shall duplicate the responsibility of another.

All healing decisions SHALL remain deterministic,
observable, and auditable.

---

## 14.1 Internal Component Diagram

```

                     Healing Request
                            │
                            ▼
                  Healing Context Builder
                            │
                            ▼
                    Input Validator
                            │
                            ▼
                Failure Classification Engine
                            │
                            ▼
                    DOM Analysis Engine
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
 Locator Recovery    Timing Recovery    Shadow DOM Recovery
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ▼
                Candidate Healing Generator
                            │
                            ▼
              Healing Candidate Scoring
                            │
                            ▼
                     Policy Engine
                            │
                            ▼
                  Healing Decision Builder
                            │
                            ▼
                    Validation Engine
                            │
                            ▼
                   Decision Engine Approval
                            │
                            ▼
                    Execution Engine
                            │
                            ▼
                   Learning Repository

```

---

# 15. Healing Processing Pipeline

Every healing request SHALL execute through
the following pipeline.

```

Failure Detected

↓

Build Healing Context

↓

Validate Inputs

↓

Classify Failure

↓

Analyze DOM

↓

Generate Candidate Solutions

↓

Evaluate Candidates

↓

Calculate Healing Candidate Score

↓

Policy Validation

↓

Decision Engine Approval

↓

Apply Healing

↓

Verify Success

↓

Persist Result

↓

Publish Event

↓

Learning Update

```

No stage may be bypassed.

---

# 16. Failure Classification Engine

## Purpose

Before healing begins,
the framework must understand
the actual cause of failure.

Healing SHALL NEVER start
without successful classification.

---

## Responsibilities

Classify failures into categories.

Determine whether healing is permitted.

Estimate recoverability.

Provide evidence for Decision Engine.

---

## Failure Categories

Locator

Timing

Rendering

Detached Element

Hidden Element

Shadow DOM

Iframe

Network Delay

Browser Instability

Configuration

Application Defect

Unknown

---

## Classification Output

Every classification includes:

- Failure ID
- Category
- Severity
- Recoverable (Yes/No)
- Confidence
- Evidence
- Recommended Healing Strategy

---

## Example

Failure

Element Not Found

↓

Category

Locator

↓

Recoverable

Yes

↓

Recommended Strategy

Semantic Locator Recovery

---

# 17. Locator Recovery Engine

## Purpose

Recover broken element locators
without changing test intent.

---

## Supported Recovery Sources

- getByTestId()
- getByRole()
- getByLabel()
- getByPlaceholder()
- aria-label
- Accessible Name
- Stable CSS
- Historical Locator
- Knowledge Graph
- DOM Relationships

---

## Locator Priority

The engine SHALL attempt recovery
using the framework locator priority.

1. getByTestId()
2. getByRole()
3. getByLabel()
4. getByPlaceholder()
5. getByText()
6. aria-label
7. name
8. id
9. Stable CSS
10. Relative XPath

Absolute XPath is prohibited.

---

## Recovery Output

Produces:

- Candidate Locator
- Confidence
- Evidence
- Expected Stability
- Historical Success

---

# 18. DOM Analysis Engine

## Purpose

The DOM Analysis Engine examines
the current application state
to determine why automation failed.

---

## Analysis Inputs

Current DOM

Previous DOM

DOM Snapshot

Accessibility Tree

Knowledge Graph

Execution History

Locator History

---

## Responsibilities

Detect:

- Removed Elements
- Renamed Attributes
- Layout Changes
- Parent Changes
- Visibility Changes
- Dynamic Rendering
- Shadow DOM
- Frame Changes

---

## Analysis Output

The engine generates
a structured DOM Analysis Report.

Example

```
DOMAnalysis
{
domVersion

removedNodes

modifiedAttributes

candidateElements

visibility

confidence
}
```

---

# 19. Candidate Healing Generator

## Purpose

Generate one or more
valid healing candidates.

The engine SHALL NEVER
produce only one option
unless no alternatives exist.

---

## Candidate Sources

Knowledge Graph

DOM Analysis

Historical Healing

Locator History

Accessibility Tree

Component Classification

Business Flow

Execution History

---

## Candidate Structure

```
HealingCandidate
{
id

strategy

locator

confidence

risk

reason

expectedOutcome
}
```

---

## Example

Candidate 1

Use getByRole()

Confidence

97%

Candidate 2

Use getByLabel()

Confidence

92%

Candidate 3

Use Stable CSS

Confidence

83%

---

# 20. Healing Candidate Scoring

## Purpose

Estimate a **Healing Candidate Score** for every candidate healing.

The score SHALL be calculated, never guessed.

---

## Ownership Boundary — Engine-Local Score

Per the Architecture Ownership Matrix (*Confidence Ownership*), **Framework
Confidence is owned exclusively by 02**, and there is exactly one Framework
Confidence model.

The value computed in this section is an **engine-local domain score**. It
measures one thing 02's model does not express: how viable a proposed locator
recovery is, given DOM similarity and accessibility evidence.

| | Framework Confidence | Healing Candidate Score |
| --- | --- | --- |
| Owner | **02** | **08** (this engine) |
| Scope | Framework-wide decisions | Locator recovery viability only |
| May be reported as "confidence" | ✓ | ✗ |
| May justify a decision | ✓ (via 02) | ✗ (evidence only) |
| Overrides the other | ✓ | ✗ |

This score SHALL NEVER:

- Be published, reported, or exported as Framework Confidence
- Replace, override, or contradict Framework Confidence
- Justify a decision reserved to 02
- Appear in a report labelled simply "confidence"

Where this score and Framework Confidence disagree, **Framework Confidence
prevails**.

**Consumes Framework Confidence through the public contract defined by
`02_Decision_Engine.md` §21.** Framework Confidence is an input to healing
approval; this score is domain evidence supporting it.

Healing approval remains a **02 decision** (§23, §33). This score informs that
decision and never substitutes for it.

---

## Scoring Factors

Evidence Quality

Locator Stability

Historical Success

DOM Similarity

Accessibility Metadata

Knowledge Graph Match

Component Classification

Environment Stability

---

## Score Scale

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

Reject Healing

The band names describe **candidate viability**, not Framework Confidence. A
score of 95 does not assert 95% Framework Confidence and SHALL NOT be reported as
such.

The reject threshold below 60 is a healing **safety floor** (§32), enforced by
this engine before a candidate reaches 02.

---

## Example Formula — Engine-Local

The formula below computes the **Healing Candidate Score** only. It is not a
Framework Confidence formula, and it does not compete with `02` §21.

```
Healing Candidate Score

=

DOM Similarity × 0.30

+

Historical Success × 0.25

+

Accessibility Match × 0.20

+

Knowledge Graph Match × 0.15

+

Environment Stability × 0.10
```

---

# 21. Validation Engine

## Purpose

Candidate healing
must be validated
before execution.

---

## Validation Rules

Validate:

- Locator uniqueness
- Element visibility
- Element interactability
- Expected role
- Expected label
- Business context
- Component type
- Policy compliance

---

## Validation Outcomes

PASS

WARNING

FAIL

Only PASS candidates
may proceed automatically.

---

# 22. Policy Engine

## Purpose

The Policy Engine ensures
all healing complies with
organizational governance.

---

## Example Policies

No Assertion Healing

No Business Logic Modification

No Authentication Bypass

No Authorization Bypass

No Production Risk

No Unsafe Locator

No Infinite Healing

---

## Policy Outcomes

Allow

Warn

Require Approval

Reject

Rejected candidates
shall be removed
before Decision Engine evaluation.

---

# 23. Healing Decision Builder

## Purpose

Construct the final
Healing Recommendation.

---

## Decision Inputs

Candidate Rankings

Confidence

Policies

Validation

Risk

Historical Learning

---

## Output

Produces:

- Winning Candidate
- Alternative Candidates
- Confidence
- Evidence
- Validation Summary
- Audit Metadata

The recommendation
is forwarded to
the Decision Engine
for approval.

---

# 24. Design Principles

Every internal subsystem SHALL satisfy:

- Single Responsibility
- Deterministic Execution
- Immutable Inputs
- Explainable Outputs
- Versioned Contracts
- Independent Testing
- Auditability
- Plugin Compatibility
- Future Extensibility
- Zero Hidden Side Effects

---

# End of Part 2

# ------------------------------------------------------------------

# Section 25 — Healing Strategy Engine

# ------------------------------------------------------------------

# 25. Healing Strategy Engine

## Purpose

The Healing Strategy Engine determines the safest recovery strategy
based on the classified failure, available evidence, organizational
policies, and historical execution data.

The objective is to restore automation stability while preserving
the original business intent.

---

## Responsibilities

The engine shall:

- Select appropriate healing strategy
- Reject unsafe strategies
- Rank multiple strategies
- Estimate expected success
- Forward recommendations to the Decision Engine

---

## Supported Strategies

- Locator Replacement
- Semantic Locator Upgrade
- Wait Strategy Adaptation
- DOM Re-query
- Shadow DOM Traversal
- Frame Re-discovery
- Lazy Loading Recovery
- Scroll Recovery
- Browser Refresh (Policy Controlled)
- Browser Restart (Decision Controlled)

---

## Strategy Selection Flow

```
Failure

↓

Classification

↓

Available Strategies

↓

Policy Filtering

↓

Confidence Calculation

↓

Candidate Ranking

↓

Decision Engine
```

---

# 26. Healing Candidate Ranking

## Purpose

Multiple healing candidates may exist.

The engine ranks each candidate using deterministic scoring.

---

## Ranking Factors

Each candidate is evaluated using:

- Confidence
- Evidence Quality
- Historical Success
- Locator Stability
- Business Context Match
- Risk Level
- Validation Result
- Execution Cost

---

## Example Score

```
Final Score

=

Confidence

+

Evidence Quality

+

Historical Success

+

Business Context

-

Risk

-

Execution Cost
```

---

## Example

| Candidate    | Score |
| ------------ | ----: |
| getByRole()  |    96 |
| getByLabel() |    91 |
| aria-label   |    87 |
| CSS Selector |    79 |

The highest valid score becomes
the recommended healing action.

---

# 27. Multi-Strategy Healing

## Purpose

Some failures require multiple coordinated recovery strategies.

---

## Example

Failure

Element Not Clickable

↓

Scroll Into View

↓

Wait Until Visible

↓

Retry Click

Instead of only replacing the locator.

---

## Supported Strategy Chains

- Wait → Retry
- Scroll → Retry
- Locator Recovery → Retry
- Frame Detection → Retry
- Shadow DOM Discovery → Retry
- Refresh → Retry (Policy Controlled)

---

## Rules

Strategies shall execute:

Sequentially

↓

Validation

↓

Continue

If any strategy fails,
the Decision Engine determines
whether another strategy is permitted.

---

# 28. Healing State Machine

## Purpose

Every healing request maintains
an explicit lifecycle state.

---

## States

REQUESTED

↓

VALIDATING

↓

CLASSIFYING

↓

DOM_ANALYSIS

↓

GENERATING_CANDIDATES

↓

RANKING

↓

POLICY_VALIDATION

↓

DECISION_APPROVAL

↓

APPLYING

↓

VERIFYING

↓

AUDITING

↓

LEARNING

↓

COMPLETED

---

## Failure States

VALIDATION_FAILED

POLICY_REJECTED

INSUFFICIENT_CONFIDENCE

VERIFICATION_FAILED

DECISION_REJECTED

FAILED

Every transition SHALL be logged.

---

# 29. Healing Validation Workflow

## Purpose

Every applied healing must be verified
before execution resumes.

---

## Validation Checks

Verify:

- Correct element selected
- Element is visible
- Element is enabled
- Expected role matches
- Expected label matches
- Business workflow preserved
- No policy violations

---

## Validation Outcomes

PASS

WARNING

FAIL

Healing SHALL proceed only after PASS.

---

## Verification Flow

```
Apply Healing

↓

Execute Action

↓

Validate Result

↓

Pass?

↓

YES

Continue Execution

↓

NO

Report Failure
```

---

# 30. Rollback Engine

## Purpose

Rollback reverses temporary healing
when verification fails.

Rollback affects only framework state.

Application state SHALL NOT be modified.

---

## Rollback Actions

- Discard candidate locator
- Restore previous locator
- Cancel pending retry
- Remove temporary DOM mapping
- Restore healing context

---

## Rollback Metadata

Every rollback records:

- Rollback ID
- Reason
- Timestamp
- Previous State
- New State
- Operator
- Evidence

---

# 31. Locator Evolution

## Purpose

Track how locators change
throughout the lifetime
of the application.

---

## Evolution Record

Each locator version stores:

- Locator ID
- Previous Locator
- New Locator
- Confidence
- Validation Count
- Success Count
- Failure Count
- Promotion Status

---

## Promotion Rules

A new locator SHALL NOT replace
the primary locator immediately.

Promotion requires:

- Repeated successful execution
- Stable confidence
- Policy approval
- Decision Engine approval

---

# 32. Healing Limits

## Purpose

Prevent excessive healing
that could hide application defects.

---

## Default Limits

Maximum healing attempts per test

3

Maximum locator replacements

2

Maximum wait adaptation

1

Maximum browser restart

1

---

## Stop Conditions

Healing SHALL stop when:

- Confidence is too low
- Policy rejects healing
- Validation fails
- Retry limit reached
- Business intent changes

---

# 33. Retry Integration

## Purpose

Healing and Retry Engines
work together under the
Decision Engine.

---

## Example

Attempt 1

Retry

↓

Attempt 2

Adaptive Wait

↓

Attempt 3

Locator Healing

↓

Attempt 4

Decision Engine

Stop

or

Escalate

---

## Rules

Retry SHALL NEVER apply healing
without Decision Engine approval.

---

# 34. AI-Assisted Healing

## Purpose

AI may recommend healing candidates,
but AI SHALL NEVER apply healing directly.

---

## AI Responsibilities

Suggest:

- Alternative locators
- DOM similarities
- Component mappings
- Accessibility matches

---

## AI Restrictions

AI SHALL NOT:

- Skip failed assertions
- Ignore validation
- Override policies
- Fabricate confidence
- Modify business intent

Every AI recommendation
must be validated
using deterministic rules.

---

# 35. Learning Integration

## Purpose

Successful and failed healing attempts
improve future recommendations.

---

## Learning Sources

- Successful healing
- Failed healing
- Locator evolution
- Retry history
- DOM changes
- Browser stability
- Execution metrics

---

## Learning Rules

Learning SHALL:

- Recommend improvements
- Never modify automation directly
- Require Decision Engine approval
  before influencing future execution

---

# 36. Enterprise Rules

The Self-Healing Engine SHALL support:

- Multi-browser healing
- Parallel execution
- Versioned locator history
- Plugin extensions
- Enterprise audit logging
- CI/CD compatibility
- Cross-environment execution
- Historical analysis

---

# 37. Metrics & Telemetry

The engine shall collect:

- Healing Success Rate
- Healing Failure Rate
- Average Confidence
- Average Recovery Time
- Locator Promotion Count
- Retry-Healing Ratio
- Browser Restart Count
- Validation Failure Rate

These metrics support continuous improvement.

---

# 38. Event Bus Integration

## Published Events

HealingRequested

HealingStarted

HealingCandidateGenerated

HealingValidated

HealingApplied

HealingRejected

HealingCompleted

LocatorPromoted

LocatorRejected

---

## Subscribed Events

ExecutionFailed

RetryApproved

DecisionApproved

DOMUpdated

LocatorChanged

FrameworkShutdown

Every event SHALL include:

- Event ID
- Timestamp
- Correlation ID
- Execution ID
- Payload Version

---

# 39. Healing Audit

Every healing attempt SHALL produce
an immutable audit record.

The audit includes:

- Healing ID
- Failure Category
- Candidate List
- Selected Strategy
- Confidence
- Validation Result
- Policy Decisions
- Decision Engine Approval
- Verification Result
- Final Outcome

Audit records SHALL NOT be modified
after creation.

---

# 40. Summary

The Self-Healing Engine enables
safe, deterministic recovery from
recoverable automation failures.

It does not replace engineering judgment.

All healing actions remain under
the authority of the Decision Engine.

Core capabilities include:

- Failure Classification
- Locator Recovery
- DOM Analysis
- Strategy Selection
- Validation
- Policy Enforcement
- Retry Integration
- AI-Assisted Recommendations
- Historical Learning
- Enterprise Auditability

The engine improves automation resilience
while ensuring genuine application defects
remain visible and actionable.

---

# End of 08_Self_Healing_Engine.md
