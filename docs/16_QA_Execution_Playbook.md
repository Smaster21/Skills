# 16_QA_Execution_Playbook.md

Version: 4.0
Status: Non-Normative Operational Playbook
Classification: Operational Playbook

**This document is NOT an engine.**

**The Engine Registry remains Documents 01–15.**

---

# 1. Purpose

This document defines HOW the framework SHALL behave during QA execution.

This document is NOT an architecture document.

This document does NOT own any runtime capability.

Instead, it defines the operational behavior expected from every engine within the framework.

It represents the execution philosophy of an experienced Senior QA Engineer and acts as the operational handbook for autonomous testing.

---

# 1A. Governance

## Depends On

- 01_Master_Architecture.md
- Architecture_Ownership_Matrix.md
- Documents 02–15

## Status

**Non-normative operational playbook.**

This document is NOT an engine. The Engine Registry remains Documents 01–15
(`01` §38). No engine number, lifecycle state, or execution phase is created by
this document.

## Precedence

```
01_Master_Architecture.md          ← always wins
        ↓
Architecture_Ownership_Matrix.md   ← governs all ownership
        ↓
Documents 02–15                    ← override this playbook on any conflict
        ↓
16_QA_Execution_Playbook.md        ← this document (operational doctrine)
```

- `01_Master_Architecture.md` SHALL always take precedence.
- `Architecture_Ownership_Matrix.md` governs ownership allocation, canonical
  datasets, artifact ownership, runtime authority, and conflict resolution.
- Documents `02`–`15` SHALL override this playbook wherever a conflict exists.
- Where this playbook conflicts with any of the above, **this playbook is
  defective** and SHALL be corrected.

## Ownership Compliance

This document **owns no capability**, produces **no canonical dataset**, and holds
**no authority**.

It SHALL NEVER:

- Create or claim engine ownership
- Override any authority defined by `01` or the Ownership Matrix
- Authorize bypassing a lifecycle stage or a mandatory quality gate
- Override Decision Engine (`02`) authority
- Override Execution Engine (`07`) runtime scheduling authority
- Calculate Risk or Framework Confidence (`02` owns both)
- Compute canonical Coverage (`05`)
- Write to the Knowledge Graph (`04`)
- Generate tests (`06`)
- Define retry policy (`05` plans, `02` approves, `07` enforces)

Every capability referenced here is consumed from its owner through that owner's
public contract.

## Relationship To The Implementation Playbook

Two playbooks coexist and do not overlap:

| Document | Layer | Answers |
| -------- | ----- | ------- |
| **16_QA_Execution_Playbook.md** | Operational QA doctrine | **WHAT** good QA behaviour looks like |
| **IMPLEMENTATION_PLAYBOOK.md** | Playwright implementation | **HOW** to build it in code |

`01` and the Ownership Matrix define **WHO**. `SKILL.md` defines **WHEN**.

---

# 2. Objectives

The framework SHALL:

• Think before acting.

• Understand the application before testing.

• Create realistic execution plans.

• Prefer business workflows over isolated actions.

• Generate evidence rather than assumptions.

• Minimize execution cost.

• Maximize confidence.

• Preserve deterministic execution.

---

# 3. Guiding Principles

The framework SHALL behave as though an experienced QA Lead is supervising every execution.

Execution SHALL always prioritize:

1. Safety
2. Correctness
3. Evidence
4. Repeatability
5. Coverage
6. Efficiency
7. Speed

Speed SHALL NEVER be prioritized above correctness.

---

# 4. QA Mindset

The framework SHALL NOT ask:

"What tests can I generate?"

Instead it SHALL ask:

"What evidence is required to prove this functionality works correctly?"

Every execution SHALL be evidence-driven.

---

# 5. QA Thinking Loop

Before executing any workflow, the framework SHALL determine:

• What am I validating?

• Why is it important?

• What could fail?

• What evidence proves success?

• What evidence proves failure?

• What assumptions exist?

• Can those assumptions be verified?

Execution SHALL begin only after these questions are answered.

---

# 6. Evidence First

Every action SHALL produce evidence.

Evidence includes:

• Assertions

• Screenshots

• Videos

• Logs

• Trace files

• API responses

• DOM snapshots

• Network traffic

• Performance metrics

A test without evidence SHALL NOT be considered complete.

---

# 7. Environment Intelligence

Before planning begins, the framework SHALL classify the execution environment.

Supported classifications include:

• Local Development

• Developer Sandbox

• QA

• Integration

• UAT

• Staging

• Production

• Shared Public Demo

No execution SHALL begin before environment classification completes.

---

# 8. Environment Detection

The framework SHALL first attempt automatic detection.

Signals include:

• Domain

• URL

• Deployment metadata

• Environment banners

• Configuration

• CI/CD metadata

If confidence is insufficient, the framework SHALL request user confirmation.

---

# 9. Environment Policies

Each environment SHALL define:

• Allowed mutations

• Allowed authentication methods

• Security level

• Cleanup requirements

• Browser strategy

• Performance limits

• Reporting requirements

Environment policies SHALL be submitted to **Document 02** as execution
constraints. `02` alone validates policy and decides how execution is
constrained. This playbook SHALL NEVER override execution behavior directly.

---

# 10. Environment Safety Levels

Safety Levels:

Level 0
Observation Only

Level 1
Read Operations

Level 2
Controlled Mutation

Level 3
Destructive Testing

The framework SHALL never exceed the environment's permitted safety level.

Per `01` §2, **penetration testing remains a non-goal** and **production
exploitation remains a non-goal**. Security testing — passive or active scanning
included — is **out of scope for qa-automation** and is owned exclusively by the
Skillmatrix security skills.

---

# 11. Risk Awareness

Risk calculation is owned exclusively by **Document 02**. This playbook SHALL
NEVER calculate, weight, or re-derive risk.

The framework SHALL detect and submit **operational signals** relevant to risk:

Business Risk signals

Technical Risk signals

Execution Risk signals

Infrastructure Risk signals

User Impact signals

Compliance Risk signals

Signals SHALL be submitted to `02` as evidence. Risk scores consumed from `02`
SHALL influence execution priority.

---

# 12. Business Workflow First

The framework SHALL prioritize complete business workflows over isolated UI interactions.

Example:

Create Employee

↓

Upload Photo

↓

Assign Department

↓

Assign Supervisor

↓

Save

↓

Search

↓

Modify

↓

Deactivate

↓

Delete

This SHALL be considered a primary workflow.

Individual field validation SHALL support, not replace, workflow validation.

---

# 13. Testing Pyramid

Execution SHALL occur across multiple levels:

Field

↓

Component

↓

Page

↓

Workflow

↓

Integration

↓

End-to-End

All levels are required.

No single level replaces another.

---

# 14. Intelligent Test Planning

The framework SHALL determine:

What MUST be tested.

What SHOULD be tested.

What MAY be tested.

What SHOULD NOT be tested.

Priority SHALL be evidence-driven.

---

# 15. Dynamic Prioritization

Execution priority SHALL consider:

Business criticality

Recent changes

Historical failures

Risk

Dependencies

Execution cost

Confidence impact

---

# 16. Environment Capability Discovery

Before test planning begins, the framework SHALL discover the capabilities of the target environment.

The objective is to understand what CAN be tested rather than assuming capabilities.

Capabilities SHALL include, but are not limited to:

• Authentication methods
• User registration
• Role management
• Password reset
• Email delivery
• SMS delivery
• File upload
• File download
• API availability
• WebSocket support
• Background jobs
• Scheduled tasks
• Third-party integrations
• Payment gateways
• Search functionality
• Reporting modules
• Audit logging
• Multi-language support
• Multi-tenant support

Capability discovery SHALL be completed before execution planning.

---

# 17. Environment Constraint Discovery

The framework SHALL identify all operational constraints before execution.

Examples include:

• Read-only environments
• Production restrictions
• Shared public demo environments
• Rate limiting
• IP restrictions
• MFA enforcement
• CAPTCHA
• WAF protection
• Limited permissions
• Session expiration policies
• API quotas
• Storage limitations

Constraints SHALL influence planning but SHALL NOT reduce reporting transparency.

Every skipped test SHALL reference the constraint responsible.

---

# 18. Execution Objectives

Each execution SHALL define measurable objectives before testing begins.

Objectives MAY include:

• Functional validation
• Regression testing
• Smoke testing
• Sanity testing
• Integration validation
• End-to-End validation
• API verification
• Visual validation *(optional, disabled initially)*
• Compliance verification

Objectives SHALL determine execution strategy.

---

# 19. Coverage Strategy

Canonical coverage is measured by **Document 05**. This playbook consumes it
and SHALL NEVER compute a coverage figure.

Coverage consumed from `05` SHALL be understood across multiple independent
dimensions.

Minimum supported dimensions include:

• Feature Coverage
• Requirement Coverage
• Workflow Coverage
• Page Coverage
• Component Coverage
• API Coverage
• Role Coverage
• Visual Coverage *(optional, disabled initially)*
• Data Coverage
• Risk Coverage

No single coverage metric SHALL represent overall quality.

Coverage SHALL always be multidimensional.

---

# 20. Business Criticality Classification

The framework SHALL classify every feature according to business impact.

Recommended levels:

Critical

High

Medium

Low

Informational

Business criticality SHALL influence:

• Planning
• Prioritization
• Retry strategy
• Evidence collection
• Reporting

Critical workflows SHALL always receive the highest validation priority.

---

# 21. Test Value Assessment

The framework SHALL estimate the expected value of every planned test.

Factors MAY include:

• Business impact
• Historical defect density
• Recent application changes
• Risk exposure
• Dependency count
• User frequency
• Regulatory importance
• Execution cost
• Expected confidence improvement

High-value tests SHOULD execute before low-value tests.

---

# 22. Requirement Traceability

Every test SHALL maintain traceability. Traceability metadata is applied by
**Document 06** during generation.

Minimum relationships:

Requirement

↓

Feature

↓

Workflow

↓

Test Case

↓

Execution

↓

Evidence

↓

Report

Every reported finding SHALL be traceable back to its originating requirement whenever possible.

---

# 23. Preconditions Verification

Before executing any workflow, the framework SHALL verify prerequisites.

Examples include:

• Environment availability
• Required accounts
• Required permissions
• Required services
• Required integrations
• Test data
• Browser readiness
• Network connectivity

Execution SHALL NOT continue when mandatory preconditions fail.

The Decision Engine SHALL determine whether execution stops, retries, or degrades gracefully.

---

# 24. Environment Validation

Immediately before execution begins, the framework SHALL validate environment health.

Validation SHALL include:

• Application availability
• Authentication availability
• API responsiveness
• Database connectivity (when accessible)
• Storage availability
• Email service availability
• File service availability
• Third-party dependency availability

Validation failures SHALL be recorded as environmental findings rather than application defects unless evidence indicates otherwise.

---

# 25. Test Data Strategy

The framework SHALL manage test data intelligently.

Supported strategies include:

• Existing datasets
• Seed datasets
• Synthetic data
• Generated data
• Imported datasets
• Masked production data
• Reusable baseline datasets

The selected strategy SHALL match the execution environment and applicable policies.

---

# 26. Test Data Ownership

Every created test artifact SHALL have a recorded owner.

Ownership SHALL include:

• Creating workflow
• Creation timestamp
• Environment
• Responsible test
• Cleanup status

Untracked data SHALL be considered orphaned and reported.

---

# 27. Test Data Lifecycle

The framework SHALL manage data throughout its lifecycle.

Stages include:

Create

↓

Validate

↓

Use

↓

Update

↓

Verify

↓

Archive or Delete

Where permitted, cleanup SHALL restore the environment to its original state.

---

# 28. Role Discovery Strategy

The framework SHALL discover application roles automatically whenever possible.

Discovery methods MAY include:

• Navigation analysis
• Permission inspection
• Role management pages
• API inspection
• Documentation
• Configuration analysis

Discovered roles SHALL be submitted as discovery information to **Document 04**.

**04 alone updates the Knowledge Graph.** This playbook SHALL NEVER write to it.

---

# 29. Role Provisioning Strategy

When permitted by environment policy, the framework MAY provision missing accounts required for testing.

Provisioning MAY include:

• User creation
• Role assignment
• Password generation
• Initial configuration

Every created account SHALL be tracked for cleanup.

If provisioning is prohibited, the resulting coverage gap SHALL be documented.

---

# 30. Authentication Strategy

The framework SHALL identify supported authentication mechanisms.

Examples include:

• Username and password
• Single Sign-On
• OAuth
• OpenID Connect
• SAML
• MFA
• Passkeys
• Magic links
• API tokens

Authentication testing SHALL adapt to the supported mechanisms.

---

# 31. Session Management Strategy

The framework SHALL validate session behavior.

Validation SHALL include:

• Session creation
• Session persistence
• Token renewal
• Idle timeout
• Concurrent sessions
• Logout behavior
• Session invalidation

Session-related findings SHALL be reported independently from authentication findings.

---

# 32. Browser Strategy

Browser execution is **Chromium-only** for the Site Explorer. Cross-browser
suites and browser-specific matrices (Firefox/WebKit) are **out of scope**
(cross-browser testing removed).

Browser allocation SHALL prioritize meaningful coverage over unnecessary duplication.

---

# 33. Device Strategy *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Device/responsive
> validation (desktop/tablet/mobile emulation and the responsive coverage
> dimension) is removed. Native mobile application automation remains a non-goal
> (`01` §2) and a future capability (`01` §35). Section number retained to
> preserve cross-references.

---

# 34. Execution Readiness Assessment

Before execution begins, the framework SHALL perform a final readiness review.

The review SHALL verify:

✓ Environment validated

✓ Constraints identified

✓ Objectives defined

✓ Test plan approved

✓ Test data available

✓ Required roles available

✓ Required services operational

✓ Reporting initialized

Execution SHALL begin only after readiness is confirmed or an approved degraded mode is selected by the Decision Engine.

---

# 35. Part 1 Summary

Part 1 establishes the operational philosophy governing QA execution.

It defines:

• How environments are understood.

• How objectives are established.

• How planning decisions are made.

• How constraints influence execution.

• How test data, roles, authentication, sessions, browsers, and devices are prepared.

The remaining parts of this document define HOW the framework executes tests, adapts during runtime, recovers from failures, and continuously improves while remaining compliant with the governance defined by Documents 01–15.

---

---

# Part 2 — QA Execution Strategies

---

# 36. Business Workflow Discovery

The framework SHALL identify complete business workflows before generating tests.

A workflow SHALL represent a meaningful business objective rather than an isolated user interaction.

Workflow discovery MAY use:

• Knowledge Graph
• Navigation Graph
• User Journeys
• API Relationships
• Business Rules
• Historical Executions

Example:

Candidate

↓

Apply

↓

Review

↓

Approve

↓

Hire

↓

Onboard

↓

Terminate

The framework SHALL prefer validating complete workflows over isolated pages whenever feasible.

---

# 37. Workflow Decomposition

Each discovered workflow SHALL be decomposed into logical execution units.

Minimum levels include:

• Workflow
• Phase
• Activity
• Task
• UI Action
• Assertion

Each level SHALL be independently executable and traceable.

---

# 38. Workflow Dependencies

The framework SHALL determine execution dependencies.

Dependencies MAY include:

• Authentication
• Required Roles
• Test Data
• APIs
• Feature Flags
• Previous Workflow Steps

Execution SHALL respect dependency ordering.

Circular workflow dependencies SHALL be reported.

---

# 39. Workflow Completion Validation

Completion SHALL NOT be determined solely by successful UI interactions.

Validation SHALL include:

• Database state (when available)
• API responses
• UI confirmation
• Audit records
• Notifications
• Generated artifacts

Workflow completion SHALL require evidence from multiple sources whenever possible.

---

# 40. Form Intelligence

The framework SHALL understand forms semantically.

It SHALL identify:

• Required fields
• Optional fields
• Validation rules
• Dependencies
• Dynamic fields
• Hidden fields
• Conditional sections

Forms SHALL be treated as business objects rather than collections of inputs.

---

# 41. Complete Form Validation

The framework SHALL validate forms at multiple levels.

Field-Level

• Required
• Optional
• Format
• Boundary

Form-Level

• Cross-field validation
• Business rules
• Submission
• Error handling

Workflow-Level

• Downstream impact
• Data persistence
• Integration behavior

Field testing SHALL NOT replace complete form testing.

---

# 42. Intelligent Test Data Population

The framework SHALL generate realistic input values.

Data SHALL respect:

• Data type
• Format
• Business rules
• Locale
• Relationships
• Uniqueness
• Dependencies

Random values SHALL NOT violate application constraints.

---

# 43. Dynamic Validation Discovery

Validation rules SHALL be discovered automatically.

Examples include:

• Required indicators

• Placeholder hints

• Error messages

• Client-side validation

• Server-side validation

• API contracts

Discovered validation behavior SHALL be submitted as evidence supporting a
**Re-plan Request**. Test generation is owned by **Document 06** and SHALL NEVER
occur at runtime.

---

# 44. CRUD Strategy

Every business entity SHALL be evaluated for:

Create

Read

Update

Delete

Search

Filter

Sort

Export

Import

Archive

Restore

Where supported.

The framework SHALL identify unsupported operations explicitly.

---

# 45. File Upload Strategy

When permitted by environment policy, the framework SHALL validate upload functionality.

Validation SHALL include:

Positive Cases

• Valid file

• Supported format

• Maximum supported size

Negative Cases

• Unsupported extension

• Oversized file

• Empty file

• Corrupted file

• Duplicate upload

Upload SHALL be verified through both UI and backend evidence where possible.

---

# 46. File Download Validation

The framework SHALL validate download functionality.

Validation SHALL include:

• Download success

• Correct filename

• File size

• MIME type

• Integrity

• Content validation

• Permission enforcement

Downloaded artifacts SHALL be retained as execution evidence when policy permits.

---

# 47. Password Reset Strategy

Where permitted, the framework SHALL execute the complete password recovery workflow.

The workflow SHALL include:

Forgot Password

↓

Email Delivery

↓

Reset Link

↓

Password Change

↓

Login Verification

↓

Old Password Rejection

↓

Audit Verification

If email access is unavailable, the framework SHALL document the limitation rather than assuming failure.

---

# 48. Authentication Scenario Coverage

Authentication SHALL be validated comprehensively.

Minimum scenarios include:

• Valid login

• Invalid password

• Locked account

• Disabled account

• Expired password

• Session timeout

• Concurrent login

• Logout

• Remember Me

• MFA (when available)

Authentication SHALL be validated across supported mechanisms.

---

# 49. Authorization Validation

Authorization SHALL verify permission boundaries.

Validation SHALL include:

• Menu visibility

• API authorization

• Page authorization

• Object ownership

• Action permissions

• Role inheritance

Unauthorized access SHALL always be reported as an **authorization / RBAC defect**
(a functional QA finding) — never as a security vulnerability claim. Security
assessment is owned by the Skillmatrix security skills.

---

# 50. Role-Based Workflow Execution

The framework SHALL execute business workflows under every available role whenever permitted.

Example:

Administrator

↓

Manager

↓

Employee

↓

Guest

↓

External User

Workflow differences SHALL be analyzed and reported.

If required roles are unavailable:

• Attempt discovery.

• Attempt provisioning (if policy permits).

• Otherwise record a justified coverage gap.

---

---

# 51. API Validation Strategy

The framework SHALL validate every discovered API that supports or participates in business workflows.

Validation SHALL include:

• Endpoint availability

• Authentication

• Authorization

• Request validation

• Response validation

• Status codes

• Schema validation

• Error handling

• Performance

• Rate limiting

• Idempotency

Whenever possible, UI actions SHALL be correlated with the underlying API activity.

---

# 52. UI–API Correlation

The framework SHALL correlate UI actions with backend communication.

Examples include:

User Action

↓

HTTP Request

↓

Business Processing

↓

Database Update

↓

UI Response

The framework SHALL verify consistency across all observable layers.

Discrepancies SHALL be reported.

---

# 53. Navigation Validation

The framework SHALL validate application navigation.

Validation SHALL include:

• Menu navigation

• Breadcrumbs

• Internal links

• External links

• Redirects

• Deep links

• Browser history

• URL consistency

Broken navigation SHALL be treated as functional defects.

---

# 54. Search Validation

The framework SHALL validate every search capability.

Validation SHALL include:

• Exact match

• Partial match

• Case sensitivity

• Empty search

• Invalid search

• Wildcards

• Pagination

• Performance

• Result accuracy

Search SHALL be evaluated for both usability and correctness.

---

# 55. Filter and Sort Validation

Every filter and sorting capability SHALL be verified.

Validation SHALL include:

• Single filter

• Multiple filters

• Combined filters

• Reset filters

• Ascending sort

• Descending sort

• Stable sorting

• Empty result handling

Filter behavior SHALL remain deterministic.

---

# 56. Pagination Validation

Pagination SHALL be validated independently.

Validation SHALL include:

• First page

• Last page

• Previous

• Next

• Page size

• Total count

• Boundary conditions

• Performance

Pagination SHALL preserve data integrity.

---

# 57. Notification Validation

The framework SHALL validate application notifications.

Supported notification types include:

• Success

• Warning

• Error

• Information

• Toast

• Banner

• Modal

Validation SHALL verify:

• Correct message

• Correct timing

• Correct dismissal behavior

---

# 58. Email Validation Strategy

Where permitted, the framework SHALL validate email workflows.

Examples include:

• Registration

• Verification

• Password Reset

• Approval

• Invitation

• Reports

Validation SHALL include:

• Delivery

• Content

• Links

• Attachments

• Formatting

When email infrastructure is unavailable, the framework SHALL document the limitation.

---

# 59. Report Validation

Generated reports SHALL be validated.

Validation SHALL include:

• Content

• Accuracy

• Filters

• Totals

• Export

• Formatting

• Performance

Reports SHALL match the underlying application state.

---

# 60. Accessibility Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Accessibility testing (keyboard, focus order, ARIA, labels, contrast, heading hierarchy, landmarks, alt text) and axe-core are removed. Section
> number retained (not renumbered) to preserve cross-references.

---

# 61. Responsive Design Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Responsive / multi-layout validation (desktop/laptop/tablet/mobile) is removed. Section
> number retained (not renumbered) to preserve cross-references.

---

# 62. Visual Validation *(optional — disabled initially)*

> **Optional, disabled initially.** Visual validation is retained as an
> optional Site Explorer category, disabled by default; enable it explicitly.

Visual validation SHALL compare rendered interfaces against expected presentation.

Validation MAY include:

• Layout

• Alignment

• Spacing

• Colors

• Typography

• Icons

• Broken images

• Rendering differences

Visual findings SHALL remain independent from functional findings.

---

# 63. Performance Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Performance testing (page load, LCP, API latency, network timing, Lighthouse) is removed. Section
> number retained (not renumbered) to preserve cross-references.

---

# 64. Security Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Security testing — passive or active scanning, security headers, input/output security, transport/API security, penetration testing — is owned exclusively by the Skillmatrix security skills, never by qa-automation. Section
> number retained (not renumbered) to preserve cross-references.

---

# 65. Browser Compatibility Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Chromium is the only supported browser; Firefox/WebKit compatibility validation is removed (cross-browser testing removed). Section
> number retained (not renumbered) to preserve cross-references.

---

# 66. Cross-Platform Validation *(removed)*

> **Removed — out of scope for the RedOps Site Explorer.** Cross-platform / cross-OS validation (Windows/Linux/macOS/Android/iOS) is removed. Section
> number retained (not renumbered) to preserve cross-references.

---

# 67. Third-Party Integration Validation

The framework SHALL validate integrated external systems.

Examples include:

• Payment Providers

• Identity Providers

• Email Services

• SMS Providers

• Cloud Storage

• Analytics

Validation SHALL distinguish between internal defects and third-party failures.

---

# 68. Data Integrity Validation

The framework SHALL verify data consistency throughout execution.

Validation SHALL include:

• UI

• API

• Database (when accessible)

• Generated Reports

• Audit Logs

Data inconsistencies SHALL be treated as critical findings.

---

# 69. Business Rule Validation

The framework SHALL discover and validate business rules.

Validation SHALL include:

• Calculations

• Eligibility

• Approval Logic

• Constraints

• State Transitions

• Conditional Logic

Business rules SHALL be validated independently from UI behavior.

---

# 70. Part 2 Summary

Part 2 defines HOW the framework executes testing activities.

It establishes standardized strategies for:

• Business workflows

• Forms

• CRUD operations

• Authentication

• Authorization

• APIs

• Visual validation *(optional, disabled initially)*

• Third-party integrations

• Business rules

These strategies ensure execution remains deterministic, evidence-driven, and aligned with the governance defined by Documents 01–15.

---


# 71. Autonomous Execution Philosophy

The framework SHALL behave as an autonomous Senior QA Engineer rather than a scripted automation runner.

Before every significant action, the framework SHALL evaluate:

• Current objective

• Available evidence

• Business impact

• Risk

• Expected outcome

• Alternative execution paths

Execution SHALL remain deterministic while allowing evidence-driven adaptation.

---

# 72. Continuous Situation Awareness

The framework SHALL continuously monitor the execution environment.

Situational awareness SHALL include:

• Environment health

• Network stability

• Browser stability

• Application responsiveness

• Resource utilization

• Active failures

• Recent recoveries

Situational changes SHALL be reported to the Decision Engine.

---

# 73. Runtime Decision Making

Runtime decisions SHALL be delegated to the Decision Engine.

The playbook defines operational conditions under which decisions are required.

Examples include:

• Retry

• Continue

• Skip

• Pause

• Abort

• Re-plan

The playbook SHALL NEVER override Decision Engine authority.

---

# 74. Adaptive Execution

Execution MAY adapt based on evidence.

Examples include:

• Reduce unnecessary repetition

• Expand validation after failures

• Increase evidence collection

• Switch browser

• Switch execution order

• Increase diagnostics

Every adaptation SHALL be explainable.

---

# 75. Failure Classification

Every failure SHALL be classified before recovery.

Minimum classifications include:

• Application Defect

• Test Defect

• Environment Failure

• Infrastructure Failure

• Third-party Failure

• Configuration Error

• Unknown

Classification SHALL determine recovery strategy.

---

# 76. Intelligent Retry Strategy

Retries SHALL NOT be blind repetitions.

Each retry SHALL determine:

• Why did execution fail?

• Is retry appropriate?

• Has the environment changed?

• Should diagnostics increase?

Retry ownership is as follows:

**05** plans retry policy.

**02** approves retry policy.

**07** enforces retry policy.

This playbook only follows the approved policy. It SHALL NEVER define, alter, or
override it.

---

# 77. Recovery Strategy

Recovery MAY include:

• Reload page

• Re-authenticate

• Recreate session

• Rebuild test data

• Restart browser context

• Restart execution worker

Recovery SHALL preserve collected evidence.

---

# 78. Self-Healing Coordination

Locator recovery SHALL remain the responsibility of Document 08.

The playbook SHALL define operational conditions under which healing is
appropriate. `02` approves; `08` performs.

Healing SHALL never conceal genuine application defects.

Recovered executions SHALL remain fully traceable.

---

# 79. Adaptive Evidence Collection

Evidence collection SHALL increase automatically when uncertainty rises.

Additional evidence MAY include:

• DOM snapshots

• Console logs

• Network traces

• Video

• Memory usage

• Performance timelines

Evidence expansion SHALL stop once sufficient diagnostic confidence is achieved.

---

# 80. Execution Checkpoints

Long-running executions SHALL create checkpoints.

Checkpoints SHALL capture:

• Progress

• Current environment

• Executed workflows

• Pending workflows

• Evidence references

Execution MAY resume from the latest valid checkpoint when supported.

---

# 81. Human-like QA Reasoning

The framework SHALL evaluate outcomes similarly to an experienced QA engineer.

It SHALL consider:

• User expectations

• Business expectations

• Technical correctness

• Historical behavior

• Consistency

Reasoning SHALL always be supported by evidence.

---

# 82. Confidence Escalation

When execution confidence decreases, the framework SHALL:

• Increase validation

• Increase evidence

• Reduce assumptions

• Notify the Decision Engine

Framework confidence SHALL remain owned exclusively by Document 02.

---

# 83. Handling Uncertainty

When sufficient evidence cannot be obtained, the framework SHALL classify the result as:

• Confirmed

• Probable

• Inconclusive

• Blocked

The framework SHALL NOT fabricate conclusions.

---

# 84. Operational Safety

The framework SHALL continuously evaluate operational safety.

Unsafe conditions MAY include:

• Production environment

• High resource utilization

• Excessive failures

• Unexpected destructive behavior

• Unauthorized privilege escalation

Execution SHALL degrade safely when required.

---

# 85. Part 3A Summary

Sections 71–85 establish the principles governing autonomous runtime behavior.

They define:

• Autonomous reasoning

• Runtime awareness

• Recovery

• Failure handling

• Adaptive execution

• Evidence escalation

• Safe operation

These principles govern HOW the framework behaves during execution while respecting the ownership defined by Documents 01–15.

---


# 86. Autonomous Test Expansion

The framework SHALL continuously determine whether additional testing is justified.

Expansion SHALL be evidence-driven rather than rule-driven.

Expansion is realized **only** through a **Re-plan Request** submitted to `02`
(§87). Runtime SHALL NEVER generate tests directly.

The framework MAY request expansion when:

• New workflows are discovered

• New APIs are discovered

• New components appear

• Previously unseen business rules are detected

• High-risk failures are identified

• Significant application changes are observed

Expansion SHALL NOT violate execution budgets or environment policies.

Every expansion decision SHALL be recorded.

---

# 87. Intelligent Scenario Requests

**Runtime SHALL NEVER generate tests directly.** Test generation is owned by
**Document 06**.

Where discovered behavior differs from expected behavior, the framework SHALL
raise a **Re-plan Request** carrying the observation and its evidence.

Observations that justify a Re-plan Request include:

• Unexpected redirects

• Hidden workflows

• Conditional forms

• Dynamic navigation

• Alternative business paths

• Previously unseen validation rules

The full path is:

```
Observation

↓

Evidence

↓

Re-plan Request

↓

02 Decision

↓

05 Planning

↓

15 Review

↓

06 Generation

↓

Validation

↓

Execution
```

Every Re-plan Request and its outcome SHALL become a traceable artifact.

---

# 88. Runtime Coverage Gap Analysis

Coverage consumed from **05** SHALL be reviewed continuously. This playbook
identifies gaps and submits them as evidence; it SHALL NEVER compute coverage.

The framework SHALL identify gaps across:

• Features

• Requirements

• Pages

• Components

• APIs

• Workflows

• Roles

Coverage gaps SHALL be classified as:

Covered

Partial

Missing

Blocked

Not Applicable

---

# 89. Intelligent Risk Escalation

Risk SHALL NEVER be recalculated by this playbook. Risk calculation is owned
exclusively by **Document 02**.

The framework SHALL detect operational signals throughout execution and **request
re-evaluation** from `02`, submitting the supporting evidence.

Re-evaluation MAY be requested when:

• Multiple related failures occur

• Critical workflows fail

• Authentication becomes unstable

• Infrastructure degrades

• Evidence conflicts

• Unknown application behavior increases

Signals and their evidence SHALL be submitted to the Decision Engine, which alone
calculates the resulting risk and decides whether escalation occurs.

---

# 90. Long Running Execution Management

Executions exceeding expected duration SHALL be managed intelligently.

The framework SHALL determine whether to:

• Continue

• Pause

• Resume later

• Split execution

• Re-plan

• Abort

Long-running executions SHALL preserve all collected evidence.

---

# 91. Resource Optimization During Execution

Execution SHALL continuously evaluate resource usage.

Resources include:

• CPU

• Memory

• Disk

• Browser instances

• Network

• Parallel workers

Optimization SHALL never reduce required evidence.

---

# 92. Adaptive Parallel Execution

Parallel execution SHALL remain adaptive.

The framework MAY reduce parallelism when:

• Target becomes unstable

• Infrastructure degrades

• Rate limiting begins

• Browser failures increase

The framework MAY request increased parallelism when stable execution is
observed, **only at deterministic checkpoints** (batch or workflow-group
boundaries). Continuous, arrival-order-driven expansion is prohibited because it
is non-deterministic.

Every adjustment SHALL be deterministic and recorded.

---

# 93. Failure Pattern Recognition

The framework SHALL detect recurring failure patterns.

Examples include:

• Common locator failures

• Repeated API failures

• Browser-specific failures

• Environment instability

• Authentication failures

Recognized patterns SHALL be submitted to the AI Learning Repository.

---

# 94. Root Cause Assistance

The framework SHALL assist engineers by identifying likely root causes.

Possible causes include:

• Product defect

• Automation defect

• Environment issue

• Infrastructure issue

• Third-party dependency

• Test data issue

Root cause suggestions SHALL always include supporting evidence.

---

# 95. Intelligent Cleanup

Cleanup SHALL occur whenever execution completes.

Cleanup MAY include:

• Test users

• Test data

• Temporary files

• Browser state

• Sessions

• Tokens

Cleanup SHALL respect environment policy.

Nothing SHALL be removed without authorization.

---

# 96. Execution Integrity Verification

Before reporting completion, the framework SHALL verify execution integrity.

Verification SHALL confirm:

• Planned tests executed

• Evidence exists

• Reports generated

• Cleanup completed

• Learning updated

• Metrics finalized

Incomplete execution SHALL be reported explicitly.

---

# 97. Knowledge Capture

Every execution SHALL contribute operational knowledge.

Knowledge MAY include:

• Successful strategies

• Failed strategies

• Stable workflows

• Unstable workflows

• Environment observations

• Recovery effectiveness

Knowledge SHALL be forwarded to Document 10.

---

# 98. Continuous Improvement

The framework SHALL continuously improve future executions.

Improvement SHALL consider:

• Historical executions

• Failure trends

• Recovery success

• Coverage gaps

• Runtime performance

• User feedback

No improvement SHALL violate deterministic execution principles.

---

# 99. Human Review Preparation

When human review is required, the framework SHALL prepare concise evidence packages.

Packages SHALL include:

• Executive summary

• Findings

• Severity

• Supporting evidence

• Root cause analysis

• Recommended actions

Reviewers SHALL never need to reconstruct execution manually.

---

# 100. Part 3B Summary

Sections 86–100 define how the framework continuously adapts while maintaining deterministic execution.

These sections establish:

• Autonomous expansion

• Coverage analysis

• Runtime optimization

• Pattern recognition

• Root cause assistance

• Cleanup

• Knowledge capture

• Continuous improvement

The framework SHALL continuously evolve while remaining fully explainable, deterministic, and evidence-driven.

---


# 101. Reporting Philosophy

Reporting SHALL communicate evidence rather than opinion.

Every report SHALL answer:

• What was tested?

• What was not tested?

• Why was it not tested?

• What evidence supports the findings?

• What risks remain?

Reports SHALL never exaggerate confidence or suppress limitations.

---

# 102. Reporting Integrity

The framework SHALL explicitly disclose:

• Skipped tests

• Blocked tests

• Deferred tests

• Environment restrictions

• Missing credentials

• Missing infrastructure

• Safety limitations

• Time budget limitations

No omitted work SHALL be hidden.

Coverage percentages SHALL always distinguish between:

• Planned Coverage

• Executed Coverage

• Verified Coverage

---

# 103. Executive Communication

The framework SHALL generate reports appropriate for different audiences.

Supported audiences include:

• Executive Leadership

• Product Owners

• QA Teams

• Developers

• Security Teams

• Compliance Teams

Each audience SHALL receive information appropriate to its responsibilities while preserving technical accuracy.

---

# 104. AI Operational Boundaries

Artificial Intelligence SHALL assist engineering decisions.

Artificial Intelligence SHALL NOT:

• Invent evidence

• Hide failures

• Modify reports

• Override governance

• Ignore environment policy

• Ignore Decision Engine authority

AI recommendations SHALL always include:

• Confidence

• Evidence

• Reasoning

• Alternative options

---

# 105. Human Override

Authorized engineers MAY override framework recommendations.

Overrides SHALL require:

• Reason

• Timestamp

• Identity

• Scope

• Approval (when required)

Every override SHALL remain permanently auditable.

---

# 106. Compliance Awareness

Compliance validation is **advisory and deferred**. The framework SHALL NOT
assert or certify compliance with any standard.

The framework MAY surface observations relevant to applicable standards, as
evidence for human assessment.

Examples include:

• ISO 27001

• SOC 2

• NIST

• PCI DSS

• HIPAA

• GDPR

Compliance observations SHALL adapt to the selected execution environment, and
SHALL be reported as advisory findings requiring human confirmation. Formal
certification remains outside framework scope.

---

# 107. Auditability

Every significant decision SHALL be auditable.

Audit records SHALL include:

• Decision

• Evidence

• Inputs

• Outputs

• Timestamp

• Responsible Engine

• Decision Authority

Audit records SHALL remain immutable.

---

# 108. Reproducibility

The framework SHALL enable deterministic re-execution.

Reproduction SHALL preserve:

• Configuration

• Environment

• Browser versions

• Test data

• Runtime schedule

• Execution policies

• Evidence references

Identical inputs SHALL produce equivalent outcomes unless the application itself
has changed (`01` §3.1).

---

# 109. Transparency

The framework SHALL explain:

• Why actions were performed.

• Why actions were skipped.

• Why retries occurred.

• Why recovery occurred.

• Why execution stopped.

Every significant runtime decision SHALL remain explainable.

---

# 110. Ethical Testing

The framework SHALL operate responsibly.

The framework SHALL NOT intentionally:

• Damage environments

• Destroy customer data

• Circumvent authorization

• Abuse third-party systems

• Exceed approved permissions

Security testing (passive or active scanning included) is **out of scope for
qa-automation** and is owned exclusively by the Skillmatrix security skills.
Penetration testing and production exploitation remain non-goals (`01` §2).

---

# 111. Collaboration

The framework SHALL produce outputs that support collaboration.

Artifacts SHALL integrate with:

• QA

• Development

• Security

• Operations

• Product Management

• Compliance

Artifacts SHALL remain understandable by both humans and automation.

---

# 112. Operational Metrics

The framework SHALL continuously measure operational quality.

Metrics MAY include:

• Execution duration

• Failure rate

• Retry rate

• Recovery rate

• Coverage

• Confidence

• Resource utilization

• Flakiness

• Evidence completeness

Metrics SHALL support continuous improvement rather than competition.

---

# 113. Continuous Governance

Governance SHALL remain active throughout execution.

Governance SHALL verify:

• Policy compliance

• Ownership compliance

• Decision authority

• Environment policy

• AI boundaries

• Reporting integrity

Governance SHALL never be bypassed.

---

# 114. Future Extensibility

The framework SHALL support future capabilities without architectural redesign.

Future extensions MAY include:

• Mobile-native automation

• Desktop application testing

• API fuzzing

• Contract testing

• Chaos engineering

• AI-assisted debugging

• Cloud-scale distributed execution

Extensions SHALL respect the ownership defined by Documents 01–15.

---

# 115. Operational Excellence

Operational excellence SHALL prioritize:

1. Correctness
2. Safety
3. Evidence
4. Explainability
5. Determinism
6. Maintainability
7. Efficiency
8. Speed

Speed SHALL remain the lowest operational priority whenever conflicts arise.

---

# 116. Enterprise QA Principles

The framework SHALL operate according to the following principles:

• Test what matters.

• Validate complete business outcomes.

• Prefer evidence over assumptions.

• Minimize unnecessary execution.

• Maximize confidence responsibly.

• Learn continuously.

• Remain transparent.

• Remain deterministic.

---

# 117. Definition of Successful Execution

Execution SHALL be considered successful only when:

• Objectives were evaluated.

• Evidence supports conclusions.

• Findings are reproducible.

• Risks are disclosed.

• Coverage is accurately reported.

• Learning has been captured.

A run with zero failures SHALL NOT automatically be considered successful if required validation was omitted.

---

# 118. Definition of Autonomous QA

An autonomous QA system SHALL:

Understand

↓

Plan

↓

Validate

↓

Adapt

↓

Recover

↓

Explain

↓

Learn

↓

Improve

Autonomy SHALL never replace governance.

Autonomy SHALL operate within approved architectural boundaries.

---

# 119. Document Responsibilities

This playbook defines:

• Expected runtime behavior

• QA philosophy

• Operational standards

• Human-equivalent reasoning

This document SHALL NOT own:

• Decision authority

• Execution scheduling

• Runtime orchestration

• Knowledge storage

• Reporting engines

• Self-healing algorithms

Those responsibilities remain owned by Documents 01–15.

---

# 120. Document Summary

The QA Execution Playbook defines HOW the framework behaves.

It complements the architectural documents by establishing:

• Operational philosophy

• QA decision-making behavior

• Environment awareness

• Business workflow validation

• Runtime adaptation

• Autonomous operation

• Reporting standards

• Governance expectations

• Enterprise QA best practices

Together with Documents 01–15, this playbook enables the framework to behave consistently, transparently, and deterministically while executing software quality assurance at an enterprise scale.

---
