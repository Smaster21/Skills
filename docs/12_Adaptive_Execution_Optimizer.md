
# QA Automation Framework

# Adaptive Execution Optimizer Specification

**Document:** 12_Adaptive_Execution_Optimizer.md

**Version:** 4.1

**Status:** Draft

> **Revision 4.1 — W8, Site Explorer boundary.** One batching example updated
> (Accessibility is no longer a category). Historical/browser execution
> performance, batch planning, runtime prediction, advisory-by-construction and
> §31 governance are **unchanged** — they concern framework execution, not target
> performance testing.

**Depends On:**

- Architecture_Ownership_Matrix.md
- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 05_Test_Planning_Engine.md
- 07_Execution_Engine.md

**Owns (per Ownership Matrix):**

- Execution Optimization (runtime optimization only)
- Batch Planning
- Runtime Prediction
- Canonical dataset: **Execution Optimization Proposal**

**Consumes (never owns):**

| Capability | Owner |
| ---------- | ----- |
| Test Planning, Test Strategy | 05 |
| Dependency Planning | 05 |
| Resource Planning (initial allocation) | 05 |
| Execution | 07 |
| Browser Management | 07 |
| Worker Scheduling | 07 |
| Timeout Management | 07 |
| Risk, Confidence, Policy, Final Decision | 02 |
| Incremental Discovery | 13 |
| Coverage Optimization | 14 |

> **Authority boundary.** The Ownership Matrix states *"Only 07 schedules
> runtime execution."* This engine owns the **Execution Optimization Proposal artifact**; it
> does not own the **act of scheduling**. The artifact is advisory input to 07.
> The Test Plan produced by 05 remains the single source of truth for what
> executes.

---

# Purpose

The Adaptive Execution Optimizer is responsible for transforming an
approved Test Plan into an advisory optimization proposal that reduces runtime.

Unlike the Test Planning Engine, which determines *what* should be
tested, the Adaptive Execution Optimizer determines *how* the approved
tests should execute with maximum efficiency while preserving
deterministic behavior.

The optimizer SHALL minimize execution time without reducing required
quality, coverage, traceability, or auditability.

---

# Table of Contents

1. Purpose
2. Scope
3. Objectives
4. Architecture Position
5. Design Principles
6. Responsibilities
7. Optimization Pipeline
8. Execution Context
9. Inputs
10. Outputs
11. Optimization Contracts
12. Non-Goals

---

# 1. Purpose

Enterprise applications may contain thousands of generated tests.

Executing every test sequentially is inefficient and significantly
increases execution time.

The Adaptive Execution Optimizer computes an optimal execution strategy
that reduces runtime while maintaining equivalent engineering confidence.

---

# 2. Scope

The optimizer governs:

- Execution **optimization** of an approved plan
- Batch planning
- Runtime prediction
- Schedule **recommendation**
- Execution budget tracking
- Execution-resource efficiency

The optimizer SHALL NOT execute tests directly.

The optimizer SHALL NOT:

- **Schedule runtime execution** (owned by 07)
- **Allocate browsers** (owned by 07)
- **Allocate or manage workers** (owned by 07)
- **Set timeouts** (owned by 07)
- **Compute dependencies** (owned by 05)
- **Compute resource plans** (owned by 05)
- **Compute parallel groups from scratch** (owned by 05)
- **Perform incremental discovery or change detection** (owned by 13)
- **Calculate Risk or Confidence** (owned by 02)
- **Modify the Test Plan** (owned by 05)

For each of the above, this engine **consumes** the capability through the
public contract of its owner and produces an optimization **recommendation**.

Execution remains the responsibility of the Execution Engine.

---

# 3. Objectives

The optimizer shall:

- Minimize total execution time
- Maximize parallel execution
- Prevent unnecessary execution
- Preserve deterministic ordering
- Respect workflow dependencies
- Optimize browser usage
- Balance worker utilization
- Reduce infrastructure cost
- Improve CI/CD throughput

---

# 4. Architecture Position

```

Decision Engine

↓

Test Planning Engine

↓

Adaptive Execution Optimizer

↓

Execution Engine

↓

Reporting Engine

```

The optimizer is an OPTIONAL advisory stage. Execution proceeds correctly when it is absent.

---

# 5. Design Principles

Every optimization must satisfy:

Deterministic

Observable

Evidence-Based

Risk-Aware

Cost-Aware

Reproducible

Auditable

Optimization shall never change test intent.

---

# 6. Responsibilities

The optimizer **recommends**:

- Execution order, within 05's dependency constraints
- Worker count → 07 decides
- Browser assignment and reuse ordering → 07 decides
- Parallel packing, within 05's eligible set
- Resource ceilings → 05 plans, 07 enforces
- Timeout values, above the p99 floor → 07 decides
- Batch composition

The optimizer **owns**:

- Batch Planning
- Runtime Prediction
- Execution Budget tracking
- The Execution Optimization Proposal artifact

The optimizer **determines nothing at runtime.** Every item above is issued as a
recommendation to its owner. Retry scheduling is planned by 05 §30 and approved
by 02; it is consumed here, not decided.

---

# 7. Optimization Pipeline

Every execution follows:

Load Test Plan

↓

Analyze Dependencies

↓

Analyze Resources

↓

Compute Parallel Groups

↓

Allocate Browsers

↓

Allocate Workers

↓

Optimize Schedule

↓

Validate Plan

↓

Publish Optimization Proposal

---

# 8. Execution Context

Every optimization request receives an immutable
Execution Optimization Context.

The context includes:

- Test Plan
- Knowledge Graph
- Workflow Graph
- Dependency Graph
- Risk Scores
- Browser Matrix
- Worker Limits
- Hardware Metrics
- Historical Performance
- Environment Information

---

# 9. Inputs

The optimizer accepts:

- test-plan.json
- workflow-graph.json
- dependency-graph.json
- risk-analysis.json
- execution-history.json
- configuration
- browser matrix
- hardware profile

---

# 10. Outputs

The optimizer produces:

`execution-optimization-proposal.json` — the canonical **Execution Optimization Proposal**

Containing:

- Recommended parallel packing (within 05's eligible set)
- Sequential groups, as constrained by 05
- Recommended worker count
- Recommended browser assignment
- Predicted duration
- Predicted utilization
- Recommended resource ceilings
- Execution budget state

Supporting artifacts:

- `batch-plan.json`
- `runtime-prediction.json`
- `execution-resource-recommendation.json`
- `execution-efficiency.json`
- `execution-budget.json`

Every field above is a **recommendation**. None overrides 05's Test Plan or
07's runtime authority.

---

# 11. Optimization Contracts

Every optimization request shall contain:

- Schema Version
- Timestamp
- Correlation ID
- Producer
- Consumer
- Optimization Context

Every response shall contain:

- Optimization Proposal (advisory, constraint-bounded)
- Metrics
- Confidence — **consumed from 02**, never computed here
- Prediction accuracy of the previous execution, where available
- Validation Status

Confidence accompanying any recommendation SHALL be obtained through the public
contract defined by `02_Decision_Engine.md` §21. This engine SHALL NOT implement
a confidence formula.

---

# 12. Non-Goals

The optimizer SHALL NOT:

- Execute browsers
- Generate Playwright code
- Modify assertions
- Heal locators
- Perform retries
- Generate reports
- Modify the Test Plan
- Schedule runtime execution *(07)*
- Allocate browsers or workers *(07)*
- Set or enforce timeouts *(07)*
- Compute dependencies or parallel eligibility *(05)*
- Compute the canonical resource plan *(05)*
- Detect application change *(13)*
- Calculate Risk or Confidence *(02)*
- Compute canonical Coverage *(05)*
- Make a final engineering decision *(02)*

Those responsibilities belong to their owners as defined by the Architecture
Ownership Matrix. This engine consumes each through the owner's public contract.

---

# End of Part 1



# ------------------------------------------------------------------

# Section 13 — Internal Architecture

# ------------------------------------------------------------------

# 13. Internal Architecture

The Adaptive Execution Optimizer is composed of multiple specialized
optimization subsystems.

Each subsystem owns exactly one responsibility.

Optimization decisions SHALL remain deterministic and reproducible.

No subsystem may directly modify the Execution Engine.

---

## Internal Component Diagram

```

                Optimization Request
                         │
                         ▼
                Context Builder
                         │
                         ▼
              Dependency Analyzer
                         │
                         ▼
              Parallelization Engine
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
Worker Allocator   Browser Allocator   Resource Optimizer
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
              Batch Planning Engine
                         │
                         ▼
           Execution Schedule Generator
                         │
                         ▼
             Schedule Validation Engine
                         │
                         ▼
              Optimized Execution Plan

```

---

# 14. Optimization Lifecycle

Every optimization request SHALL execute through the following pipeline.

```

Receive Test Plan

↓

Build Context

↓

Analyze Dependencies

↓

Analyze Resources

↓

Determine Parallel Candidates

↓

Allocate Workers

↓

Allocate Browsers

↓

Create Execution Batches

↓

Optimize Runtime Schedule

↓

Validate Schedule

↓

Publish Optimization Proposal

```

No stage may be skipped.

---

# 15. Context Builder

## Purpose

The Context Builder assembles every input required for optimization.

Rather than querying multiple engines independently,
the optimizer receives a single immutable context object.

---

## Responsibilities

Collect:

- Approved Test Plan
- Workflow Graph
- Dependency Graph
- Browser Matrix
- Risk Scores
- Historical Performance
- Hardware Metrics
- Worker Limits
- Environment Configuration
- CI/CD Constraints

---

## Output

```text
OptimizationContext
{
    contextId
    executionId
    correlationId
    testPlan
    workflowGraph
    dependencyGraph
    browserMatrix
    hardwareProfile
    historicalMetrics
    resourceLimits
    configuration
    timestamp
}
```

The Optimization Context SHALL remain immutable throughout execution.

---

# 16. Dependency Consumption

## Purpose

Not every test can execute simultaneously.

Per the Ownership Matrix, `Dependency Planning` is owned by **05**
(*"Planning only"*), with 12 as consumer.

This engine **consumes the dependency graph through the public contract defined
by `05_Test_Planning_Engine.md` §25 (Dependency Planning Engine)**, which
produces `dependency-plan.json`.

It SHALL NOT compute dependency chains, detect cycles, or derive execution
groups. Cycle detection remains a planning failure per 07 §29.

What this engine adds is **ordering and packing optimization** over the
already-computed dependency graph.

| Action | Owner |
| ------ | ----- |
| Identify dependencies, detect cycles | 05 |
| Emit `dependency-plan.json` | 05 |
| Optimize order and packing within those constraints | **12** (this engine) |
| Enforce ordering at runtime | 07 |

---

## Dependency Types Consumed

Workflow Dependency

Example

Login

↓

Dashboard

↓

Settings

Settings depends on Login.

---

Fixture Dependency

Example

Database Seed

↓

User Creation

↓

Role Assignment

---

Shared Resource Dependency

Example

Shared Account

Shared API Key

Shared Browser Context

---

Environment Dependency

Example

Development Environment

↓

Migration

↓

Functional Tests

---

## Consumed Input

`dependency-plan.json` — produced by 05 §25, containing:

- Dependency Chains
- Independent Tests
- Sequential Groups
- Blocking Resources
- Parallel Candidates

This engine reads these values. It does not recompute them, and SHALL NOT emit
a competing `dependency-analysis.json`.

---

# 17. Parallel Packing Optimizer

## Purpose

Determine the most efficient arrangement of the parallel candidates already
identified by planning.

`Dependency Planning` — including which tests *may* run concurrently — is owned
by **05** §26. **Consumes parallel candidacy through the public contract defined
by `05_Test_Planning_Engine.md` §26 (Parallelization Engine).**

This engine does not decide *whether* two tests may run together. It decides
*how* the permitted set is packed for throughput, and recommends that packing
to 07.

---

## Eligibility — Consumed, Not Derived

The safety rules governing concurrency are defined by 05 §26 and enforced by
07 §32 (Test Isolation Policy). They are deliberately not restated here;
duplicating them would create a second correctness authority.

This engine treats 05's parallel candidate set as **authoritative and
non-negotiable**. It SHALL NEVER widen that set, and SHALL NEVER place two
tests in the same parallel group unless 05 marked them eligible.

Optimization operates only within the permitted set.

---

## Parallel Groups

Example

Group A

- Login
- Search
- Reports

Group B

- User Management
- Billing

Group C

- Navigation

Groups execute independently.

---

## Sequential Groups

Examples

Checkout

↓

Payment

↓

Confirmation

These SHALL execute in order.

---

# 18. Worker Count Recommendation

## Purpose

Workers are expensive resources.

Per the Ownership Matrix, `Worker Scheduling` is owned by **07**
(*"Runtime scheduling"*), with 12 as consumer.

This engine **recommends** a worker count. It SHALL NOT allocate workers,
manage worker queues, track worker status, or recover idle workers — all of
which are 07 §20 and 07 §33.

**Consumes worker scheduling through the public contract defined by
`07_Execution_Engine.md` §20 (Worker Scheduler).**

The recommendation is advisory. 07 MAY reduce it for runtime safety and is not
required to accept it.

---

## Inputs

CPU Cores

Memory

Available Browsers

Historical Worker Utilization

Execution Budget

Environment Limits

---

## Allocation Strategy

Example

Available CPU

16 cores

Maximum Workers

8

Selected Workers

6

Reason

Memory constrained.

---

## Objectives

Maximize utilization.

Avoid resource starvation.

Prevent browser thrashing.

---

# 19. Browser Assignment Recommendation

## Purpose

Recommend an efficient mapping of tests to browsers so that unnecessary browser
launches are avoided.

Per the Ownership Matrix, `Browser Management` is owned by **07**
(*"Runtime browser lifecycle"*). The browser **matrix** itself is planned by 05
§27.

This engine therefore recommends *assignment and reuse ordering* only. It SHALL
NOT launch, close, restart, pool, or recycle browsers, and SHALL NOT alter the
planned browser matrix.

**Consumes browser lifecycle through the public contract defined by
`07_Execution_Engine.md` §19 and §26 (Browser Manager).**
**Consumes the browser matrix through `05_Test_Planning_Engine.md` §27.**

Risk-tiered browser breadth below is **consumed** from 02's risk model, never
recalculated here.

---

## Browser Inputs

Supported Browsers

Execution Mode

Risk Profile

Browser Availability

Historical Stability

---

## Allocation Rules

Critical workflows

↓

All supported browsers

Medium risk

↓

Primary browser

Low risk

↓

Single browser

Smoke execution

↓

Configured default browser

---

## Browser Reuse

Whenever safe:

Reuse existing browser processes.

Avoid launching new browser instances.

Reuse browser contexts only when isolation rules permit.

---

# 20. Resource Efficiency Recommendation

## Purpose

Prevent hardware bottlenecks.

Execution speed depends on efficient resource utilization.

Per the Ownership Matrix, `Resource Planning` is owned by **05**
(*"Initial allocation"*), and runtime resource management is owned by **07**
§18/§33.

This engine produces an **efficiency recommendation** over 05's allocation. It
SHALL NOT allocate, monitor, release, or recycle resources.

**Consumes resource planning through the public contract defined by
`05_Test_Planning_Engine.md` §21 (Resource Planning Engine).**
**Consumes runtime resource management through `07_Execution_Engine.md` §33.**

---

## Resources

CPU

Memory

Disk IO

Network

Browser Processes

Worker Threads

Temporary Storage

Artifact Storage

---

## Optimization Rules

Reduce worker count during memory pressure.

Throttle browser launches.

Limit simultaneous video recording.

Compress artifacts asynchronously.

Delay non-critical analytics until execution completes.

---

## Output

`execution-resource-recommendation.json`

Containing:

- Recommended CPU Ceiling
- Recommended Memory Budget
- Recommended Worker Budget
- Recommended Browser Budget
- Recommended Disk Usage
- Recommended Artifact Limits

> **Artifact boundary.** This engine SHALL NOT emit `resource-plan.json`. That
> filename is the canonical output of 05 §21. Two producers of one artifact
> would violate Canonical Data Ownership.

---

# End of Part 2



# ------------------------------------------------------------------

# Section 21 — Dynamic Batch Planning Engine

# ------------------------------------------------------------------

# 21. Dynamic Batch Planning Engine

## Purpose

Executing thousands of tests individually creates unnecessary browser
startup overhead, fixture initialization, and resource contention.

The Dynamic Batch Planning Engine groups compatible tests into optimized
execution batches.

Batch planning SHALL preserve test isolation while minimizing execution
overhead.

---

## Batch Objectives

The engine shall:

- Reduce browser startups
- Reuse compatible fixtures
- Reduce context switching
- Improve worker utilization
- Preserve deterministic execution
- Maintain failure isolation

---

## Batch Inputs

- Test Plan
- Dependency Graph
- Workflow Graph
- Browser Assignment
- Fixture Requirements
- Risk Classification
- Execution History

---

## Batch Rules

Tests may belong to the same batch only if they:

- Require the same browser
- Require identical fixtures
- Have no ordering conflicts
- Do not mutate shared state
- Share execution characteristics

---

## Batch Output

batch-plan.json

Containing:

- Batch ID
- Tests
- Browser
- Worker
- Estimated Duration
- Required Fixtures

---

# 22. Incremental Execution Consumption

## Purpose

Large enterprise applications rarely change completely. Executing only the
tests affected by application change is one of the largest available savings.

That capability is **not owned here.**

Per the Ownership Matrix, `Incremental Discovery` is owned by **13** and
`Graph Diffing` is owned by **04**. The change-detection algorithm previously
specified in this section duplicated both.

**Consumes application change detection through the public contract defined by
`13_Incremental_Discovery_Engine.md` (Discovery Delta).**
**Consumes graph comparison through `04_Knowledge_Graph.md` §18.**
**Consumes the affected-test selection through `05_Test_Planning_Engine.md`,
which incorporates the Discovery Delta into the Test Plan.**

---

## Boundary

| Action | Owner |
| ------ | ------ |
| Detect application change | 13 |
| Compare graph versions | 04 |
| Select which tests are affected | 05 |
| Determine regeneration scope | 06 |
| **Optimize the order and batching of the selected set** | **12** (this engine) |
| Execute the selected set | 07 |

This engine receives an already-reduced test set and optimizes its execution.
It SHALL NOT decide which tests are skipped — that reduction is planning's
output, derived from 13's delta.

---

## Relocation Notice

The change-detection algorithm formerly specified here is fully specified in
**13 §11 (Change Detection Engine)**, **13 §12 (Fingerprint Engine)**, and
**13 §13–16 (Delta Analyzers)**. No capability was lost in removing it from this
document.

---

# 23. Execution Budget Manager

## Purpose

Execution resources are finite.

The Execution Budget Manager ensures that runtime remains within
configured engineering limits.

---

## Budget Categories

Execution Time

CPU

Memory

Network

Browser Sessions

Workers

Artifact Storage

Cloud Minutes

CI Budget

---

## Budget Policies

Hard Limit

Execution stops when exceeded.

Soft Limit

Optimizer attempts recovery.

Adaptive Limit

Decision Engine evaluates continuation.

---

## Budget Output

execution-budget.json

Containing:

- Current Usage
- Remaining Budget
- Predicted Consumption
- Confidence
- Recommendation

---

# 24. Execution Resource Efficiency

## Purpose

Engineering efficiency includes efficient use of execution resources.

Scope is limited to **execution-resource efficiency**. Cloud billing,
infrastructure procurement, and organizational cost accounting are outside the
framework's ownership and are not modelled here.

Retry economics are **consumed**, not decided: retry policy is planned by 05
§30 and approved by 02 (01 §21).

---

## Efficiency Factors

Worker Count (recommended, per §18)

Browser Launches and reuse (recommended, per §19)

Artifact Storage volume

Execution Time

Network Usage

Video Recording

Trace Collection

---

## Optimization Rules

Reuse browsers whenever safe.

Avoid duplicate artifact collection.

Compress artifacts after execution.

Limit expensive diagnostics to failed tests.

Disable unnecessary video capture.

Avoid redundant retries.

---

## Output

`execution-efficiency.json`

Containing:

- Estimated resource consumption
- Projected savings versus the unoptimized plan
- Recommended resource ceilings
- Efficiency breakdown by factor

Figures are **estimates of framework resource use**, not financial cost.

---

# 25. Adaptive Timeout Recommendation

## Purpose

Static timeouts either waste execution time or create flaky tests.

Per the Ownership Matrix, `Timeout Management` is owned by **07**
(*"Runtime timeouts"*), with 12 as consumer.

This engine **recommends** timeout values derived from historical execution
data. It SHALL NOT set, apply, or enforce a timeout.

**Consumes timeout management through the public contract defined by
`07_Execution_Engine.md` §34 (Timeout Manager).**

---

## Safety Floor

A recommendation that lowers a timeout trades quality for speed. 07 §5.5 ranks
reliability and correctness above performance, and 01 §3.5 requires safe
defaults.

Therefore a recommended timeout SHALL NEVER fall below the **observed p99**
duration for that operation, plus a configured safety margin.

Recommendations SHALL be rejected by 07 when:

- Historical sample size is insufficient
- Observed variance exceeds the configured ceiling
- The environment is marked DEGRADED or UNKNOWN (07 §31)
- The resulting value would sit below the p99 floor

Reducing timeouts toward the historical mean manufactures flakiness. Mean-based
reduction is prohibited.

---

## Inputs

Historical Runtime

Network Latency

Environment Speed

Browser Performance

Application Complexity

Historical Timeout Events

---

## Strategy

Fast Environment

↓

Lower Timeout

Slow Environment

↓

Higher Timeout

Historical Stability

↓

Reduce Safety Margin

High Variability

↓

Increase Safety Margin

---

## Example

Historical Mean

2.1 seconds

Observed p99

7.4 seconds

Configured Timeout

30 seconds

Safety Margin

50%

**Recommended Timeout**

11 seconds

Note that the recommendation is derived from **p99 + margin**, not from the
mean. A mean-derived value of ~6 seconds would sit below observed p99 and would
fail roughly one execution in a hundred for no defect.

---

# 26. Historical Runtime Predictor

## Purpose

Predict execution duration before execution begins.

Accurate prediction improves scheduling, CI planning, and resource
allocation.

---

## Prediction Sources

Historical Runtime

Knowledge Graph

Risk Score

Execution Mode

Browser

Worker Count

Retries

Application Size

---

## Prediction Output

runtime-prediction.json

Containing:

- Estimated Duration
- Confidence
- Expected Bottlenecks
- Slowest Workflows
- Longest Tests

---

## Example

Estimated Runtime

18 minutes

Confidence

95%

Longest Workflow

Checkout

Estimated Duration

4 minutes

---

# 27. Optimization Proposal Generator

## Purpose

Convert every optimization result into a deterministic execution
schedule understood by the Execution Engine.

---

## Schedule Contents

Execution Order

Batch Order

Worker Assignment

Browser Assignment

Dependencies

Timeouts

Resource Allocation

Retry Strategy

Expected Completion

---

## Example

Worker 1

Login

↓

Dashboard

↓

Logout

Worker 2

Reports

↓

Export

Worker 3

Search

↓

Filters

↓

Results

---

## Output

`execution-optimization-proposal.json`

This artifact is the canonical **Execution Optimization Proposal** owned by this
engine.

It is a **proposal**, not a schedule. The word matters: an artifact named
"schedule" owned by a non-executor invites the misreading that it carries
scheduling authority. It does not.

| Question | Authority | Artifact |
| -------- | --------- | -------- |
| What executes? | **05** | `test-plan.json` *(canonical intent)* |
| How could it run faster? | **12** | `execution-optimization-proposal.json` *(advisory)* |
| What actually ran, in what order? | **07** | `runtime-schedule.json` *(canonical runtime)* |
| Is a deviation permitted? | **02** | Decision record |

---

## Advisory By Construction

This Proposal is advisory **structurally**, not by policy. Policy decays under
maintenance pressure; structure does not.

### Permitted Content

A Proposal MAY contain only:

- Execution ordering
- Batching and grouping
- Worker-count hint
- Browser-assignment hint
- Timeout hint

### Prohibited Content

A Proposal SHALL NEVER contain:

- A test identifier absent from the Test Plan
- An added or removed test
- Altered scope, eligibility, or assertions
- Any change to what is validated

### Verification

07 SHALL reject any Proposal whose test set is not **exactly** the Test Plan's
test set.

This is a set comparison. It is mechanically verifiable and requires no
interpretation, which is precisely why it is the guarantee rather than a stated
intention.

A Proposal is therefore a **constraint-bounded permutation** of the Test Plan. It
can affect speed. It cannot affect correctness.

---

## Null-Safety — The Replaceability Guarantee

**Absence of a Proposal is a valid state.**

Where no Proposal exists, is rejected, or is stale, 07 SHALL execute the Test
Plan's own ordering, and the result SHALL be correct.

The framework SHALL remain fully functional with this engine removed entirely.

A framework that cannot execute without its optimizer does not have a
replaceable optimizer. This property is what makes the optimizer swappable, and
what allows a plugin to supply a competing Proposal without any core change.

---

## Reproducibility

Every execution records a run manifest:

```text
(test-plan version, proposal version | null, runtime-schedule version)
```

This reproduces any execution exactly — including one in which no optimizer was
present.

---

## Deviation Feedback

07 SHOULD follow the Proposal and SHALL record every deviation with its reason.
07 MAY deviate for runtime safety — resource pressure, browser instability, a
degraded environment — without requesting approval.

Deviations flow 07 → 09 → 10 → back to this engine, so proposal quality becomes
measurable across executions. An optimizer whose proposals are never compared
against actual runtime cannot be shown to help.

This engine SHALL NOT modify the Test Plan, and SHALL NOT instruct 07 to execute
anything the Test Plan did not select.

---

# ------------------------------------------------------------------

# Part 4

# Sections 28–32

# ------------------------------------------------------------------

# 28. Ownership Compliance

## Owned Here

| Capability | Canonical Dataset |
| ---------- | ----------------- |
| Execution Optimization | Execution Optimization Proposal |
| Batch Planning | `batch-plan.json` |
| Runtime Prediction | `runtime-prediction.json` |

## Consumed Through Public Contracts

| Capability | Owner | Contract |
| ---------- | ----- | -------- |
| Test Plan / Test Strategy | 05 | `test-plan.json` |
| Dependency Planning | 05 | §25 `dependency-plan.json` |
| Parallel candidacy | 05 | §26 |
| Resource Planning | 05 | §21 `resource-plan.json` |
| Browser matrix | 05 | §27 |
| Execution | 07 | §22 |
| Browser Management | 07 | §19, §26 |
| Worker Scheduling | 07 | §20 |
| Timeout Management | 07 | §34 |
| Runtime resource management | 07 | §33 |
| Risk / Confidence / Policy / Decision | 02 | Decision contracts |
| Application change detection | 13 | Discovery Delta |
| Coverage Optimization | 14 | Coverage Recommendations |
| Reporting | 09 | Report input contract |

## Explicitly Not Owned Here

This engine SHALL NEVER calculate Risk or Confidence, compute dependencies or
parallel eligibility, allocate browsers or workers, set timeouts, detect
application change, modify the Test Plan, schedule runtime execution, or make a
final engineering decision.

---

# 29. Event Bus Integration

This engine communicates exclusively through the framework Event Bus (01 §27).
Direct invocation of another engine is prohibited.

## Published Events

OptimizationRequested

OptimizationStarted

DependencyPlanConsumed

ParallelPackingCompleted

WorkerCountRecommended

BrowserAssignmentRecommended

BatchPlanCreated

RuntimePredicted

TimeoutRecommended

BudgetThresholdReached

OptimizedScheduleCreated

OptimizationFailed

## Subscribed Events

PlanningCompleted

PlanApproved

GenerationCompleted

IncrementalDiscoveryCompleted

CoverageCalculated

ExecutionStarted

ExecutionCompleted

ScheduleDeviationRecorded

DecisionApproved

FrameworkShutdown

Every event SHALL include Event ID, Event Type, Timestamp, Execution ID,
Correlation ID, Producer, and Payload Version.

---

# 30. Audit & Traceability

Every optimization SHALL produce an immutable audit record containing:

Optimization ID

Execution ID / Correlation ID

Framework Version / Configuration Version

Test Plan Version consumed

Inputs consumed, with producer and version

Recommendations issued

Predicted duration and predicted utilization

Actual duration and actual utilization, once execution completes

Deviations recorded by 07, with reasons

Prediction accuracy

Audit records SHALL NOT be modified after creation.

Recording predicted-versus-actual is mandatory. An optimizer whose predictions
are never checked cannot be shown to help, and 10 requires the history to
improve future recommendations.

---

# 31. Security & Governance

This engine consumes execution metadata and MAY consume historical performance
data. It SHALL NOT expose:

Passwords · Tokens · Cookies · Session identifiers · API keys ·
Personal data · Financial data

Sensitive values SHALL be masked before persistence, per 01 §30 and 07 §41.

Every recommendation SHALL be traceable, auditable, explainable, and versioned.

AI MAY assist prediction and ordering. Per the matrix AI Authority clause, AI
SHALL NEVER override policy, risk, confidence, or an engineering decision, and
SHALL NEVER modify a canonical dataset. Every AI-assisted recommendation SHALL
carry confidence consumed from 02, supporting evidence, and a reasoning summary.

---

# 32. Quality Gates & Summary

## Pre-Optimization Gates

✓ Test Plan approved by 02

✓ Dependency plan available from 05

✓ Resource plan available from 05

✓ Browser matrix available from 05

✓ Capabilities registered

✓ Configuration valid

## Optimization Gates

✓ Parallel candidacy consumed, not derived

✓ Recommendations bounded by owner constraints

✓ Timeout recommendations above the p99 floor

✓ Budget evaluated

✓ Schedule validated

## Post-Optimization Gates

✓ Execution Optimization Proposal persisted and versioned

✓ Audit record created

✓ Events published

✓ Predicted metrics recorded for later comparison

Failure of any mandatory gate SHALL prevent the Proposal from being
published. Execution then proceeds on 05's unoptimized plan, which is always a
valid fallback.

## Summary

This engine converts an approved Test Plan into the most efficient **recommended**
execution arrangement, and predicts its cost before execution begins.

It owns optimization, batching, and prediction. It owns no dependency model, no
resource model, no browser lifecycle, no worker pool, no timeout, and no
scheduling authority. Every one of those is consumed from its matrix owner, and
every output is advisory to 07 under the authority of 02.

Optimization never changes what is tested, only how efficiently it runs.

---

# End of 12_Adaptive_Execution_Optimizer.md
