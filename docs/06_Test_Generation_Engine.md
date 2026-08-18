# QA Automation Framework

# Test Generation Engine Specification

**Document:** 06_Test_Generation_Engine.md

**Version:** 4.1

**Status:** Draft

> **Revision 4.1 — W8, Site Explorer boundary.** Accessibility, Performance and
> Security test outputs, templates, assertion types and plugin types removed;
> Visual marked OPTIONAL — DISABLED BY DEFAULT. §12 Non-Goals extended with an
> explicit prohibition on generating security tests and application-delete tests.
> §22.1 and §31.1–§31.5 **Test Case Identity and the Test Catalogue are retained
> unchanged**, with an added clause confirming the Catalogue records execution
> facts only and never applicability, probability, or a recommended skill.
> §26 Locator Resolution and its priority order are **unchanged** (C7).

**Depends On:**

- 01_Master_Architecture.md
- 02_Decision_Engine.md
- 03_Discovery_Engine.md
- 04_Knowledge_Graph.md
- 05_Test_Planning_Engine.md

---

# Purpose

This document specifies the architecture, responsibilities, workflows,
contracts, governance rules, AI generation strategy, and engineering
standards of the Test Generation Engine.

The Test Generation Engine transforms an approved Test Plan into
enterprise-grade Playwright automation.

It SHALL generate deterministic, maintainable, scalable, reusable,
auditable, and production-ready automation assets.

The engine SHALL NOT execute tests.

Execution belongs exclusively to the Execution Engine.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architectural Position
5. Design Principles
6. Generation Philosophy
7. Inputs
8. Outputs
9. Generation Lifecycle
10. Generation Context
11. Generation Contracts
12. Non-Goals

---

# 1. Purpose

The purpose of the Test Generation Engine is to convert planning
artifacts into executable automation.

Rather than generating isolated test scripts, the engine constructs
a complete automation project following enterprise engineering
standards.

Every generated artifact shall be deterministic, reusable,
readable, versionable, and suitable for CI/CD environments.

---

# 2. Scope

The Test Generation Engine is responsible for generating:

- Page Objects
- Test Specifications
- Fixtures
- Helper Utilities
- Test Data Models
- Assertions
- API Tests
- Visual Tests *(OPTIONAL — DISABLED BY DEFAULT)*
- Shared Components
- Configuration Files
- Project Structure

The engine SHALL NOT:

- Execute tests
- Retry failures
- Heal locators
- Capture artifacts
- Produce reports
- Modify application state outside planned test actions

---

# 3. Responsibilities

The Test Generation Engine shall:

- Read the approved Test Plan
- Read the Knowledge Graph
- Read Discovery artifacts
- Read Risk Analysis
- Select appropriate templates
- Generate maintainable Playwright code
- Generate reusable Page Objects
- Create meaningful assertions
- Produce deterministic test suites
- Generate supporting utilities
- Validate generated output
- Produce generation metadata

Every generated file shall be traceable to its planning source.

---

# 4. Architectural Position

```

```

    Test Planning Engine
                         │
                         ▼
               Test Generation Engine
                         │
      ┌──────────────────┼──────────────────┐
      ▼                  ▼                  ▼
 Page Objects      Test Suites        Shared Utilities
      │                  │                  │
      └──────────────────┼──────────────────┘
                         ▼
               Validation Engine
                         │
                         ▼
               Execution Engine

```

The Generation Engine consumes planning decisions.

It never decides:

- what to test
- when to retry
- execution priority

Those decisions belong to the Decision Engine.

---

# 5. Design Principles

Every generated artifact shall satisfy the following principles.

---

## 5.1 Deterministic Generation

Given identical inputs,
the engine shall generate equivalent outputs.

Generation shall not depend upon randomness.

---

## 5.2 Readability

Generated code shall resemble code written by an experienced
Senior QA Automation Engineer.

Readable code is preferred over compact code.

---

## 5.3 Reusability

Common logic shall never be duplicated.

Shared behaviors belong inside:

- Page Objects
- Components
- Fixtures
- Utilities

---

## 5.4 Maintainability

Generated projects shall minimize future maintenance costs.

Examples include:

- Stable locators
- Shared selectors
- Centralized configuration
- Common helper methods

---

## 5.5 Explainability

Every generated artifact shall contain sufficient metadata
to explain:

- why it exists
- which workflow it validates
- which planning decision produced it
- associated risk level

---

## 5.6 Enterprise First

Generation shall prioritize:

- scalability
- readability
- modularity
- CI compatibility
- version control friendliness

over rapid script creation.

---

# 6. Generation Philosophy

The framework does not generate tests from HTML alone.

Generation is driven by engineering knowledge.

Pipeline

```

Discovery

↓

Knowledge Graph

↓

Risk Analysis

↓

Planning

↓

Generation

↓

Validation

```

Generation SHALL always occur after planning approval.

Direct HTML-to-test generation is prohibited.

---

# 7. Inputs

The Generation Engine consumes only approved engineering artifacts.

Primary inputs include:

- Test Plan
- Knowledge Graph
- Discovery Results
- Component Classification
- Business Workflows
- Risk Analysis
- Capability Registry
- Configuration
- Environment Profile
- User Overrides

Each input shall be versioned.

---

# 8. Outputs

The engine produces structured automation assets.

Examples

Project Structure

Page Objects

Tests

Fixtures

Utilities

Configuration

Metadata

Validation Results

Generation Report

Each generated file shall include generation metadata.

---

# 9. Generation Lifecycle

Every generation follows the same lifecycle.

```

Receive Plan

↓

Validate Inputs

↓

Build Generation Context

↓

Select Templates

↓

Generate Assets

↓

Generate Assertions

↓

Validate Code

↓

Resolve Dependencies

↓

Package Project

↓

Produce Metadata

↓

Return Generation Result

```

No stage may be skipped.

---

# 10. Generation Context

The engine constructs an immutable Generation Context.

Example

GenerationContext

```

{
contextId

executionId

planId

knowledgeGraphVersion

configurationVersion

workflow

risk

components

capabilities

environment

timestamp
}

```

The Generation Context SHALL remain immutable
throughout generation.

---

# 11. Generation Contracts

All communication occurs through immutable contracts.

Example

Planning Engine

↓

Generation Request

↓

Generation Engine

↓

Generation Response

↓

Execution Engine

No module may directly modify generated artifacts
outside defined contracts.

---

# 12. Non-Goals

The Generation Engine SHALL NOT:

Execute Playwright

Retry failures

Heal locators

Restart browsers

Analyze reports

Perform learning

Manage execution scheduling

Generate fake test results

---

## Prohibited Generation (W8)

The Generation Engine SHALL NEVER generate:

- **Any security test** — SQL injection, XSS, SSRF, IDOR, command injection,
  path traversal, CSRF exploitation, authentication bypass, privilege
  escalation, or any other exploitation attempt.
- **Attack-shaped input payloads** in any category, including Forms. Malformed
  input for boundary and validation testing uses the canonical data set
  (`PLAYBOOK` §6), never attack strings.
- **Application-delete tests.** Delete affordances are discovered and recorded,
  never exercised (`16` §44, C9). Cleanup of a test's *own* synthetic data is a
  separate obligation and is not a Delete test.
- **Accessibility / WCAG compliance tests**, performance tests, responsive
  viewport matrices, or cross-browser matrices.
- **Any artifact that asserts, implies, or scores that a surface is vulnerable**,
  that an attack class applies to it, or that a security skill should be run
  against it (`01` §2.1).

A separately selected security skill reads this framework's evidence and
determines its own methodology. That generation happens outside this engine.

All execution responsibilities belong to the
Execution Engine.

---

# Summary

The Test Generation Engine converts engineering knowledge into
enterprise-grade Playwright automation.

Subsequent sections define:

- Internal Architecture
- Template Engine
- Page Object Generation
- Assertion Generation
- Fixture Generation
- Validation
- AI-Assisted Generation
- Incremental Regeneration
- Code Quality Standards
- Plugin Integration
- Generation Metrics

---

# End of Part 1

# ------------------------------------------------------------------
# Section 13 — Internal Architecture
# ------------------------------------------------------------------

# 13. Internal Architecture

The Test Generation Engine is composed of specialized generation
subsystems.

Each subsystem owns one bounded responsibility.

Generation SHALL be modular, deterministic, observable,
and independently testable.

No subsystem may directly modify another subsystem's internal state.

Communication SHALL occur only through immutable contracts.

---

## 13.1 Internal Component Diagram

```

    Generation Request
                            │
                            ▼
                 Generation Context Builder
                            │
                            ▼
                   Input Validation Engine
                            │
                            ▼
                  Template Selection Engine
                            │
                            ▼
                 Workflow Generation Engine
                            │
          ┌─────────────────┼─────────────────┐
          ▼                 ▼                 ▼
   Page Object      Test Case Generator   Fixture Generator
      Generator
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
                 Assertion Generation Engine
                            │
                            ▼
                 Locator Resolution Engine
                            │
                            ▼
                  Shared Utility Generator
                            │
                            ▼
                  Dependency Resolution Engine
                            │
                            ▼
                  Project Assembly Engine
                            │
                            ▼
                   Validation Preparation
                            │
                            ▼
                    Generation Response

```

Every component performs one responsibility.

Business decisions remain under the authority of the Decision Engine.

---

# 14. Generation Pipeline

Every generation SHALL execute through the following pipeline.

```

Generation Request

↓

Context Build

↓

Validation

↓

Template Selection

↓

Workflow Expansion

↓

Page Object Generation

↓

Fixture Generation

↓

Assertion Generation

↓

Locator Resolution

↓

Utility Generation

↓

Dependency Resolution

↓

Project Assembly

↓

Metadata Generation

↓

Validation

↓

Generation Complete

```

Every stage is mandatory.

Skipping stages is prohibited.

---

# 15. Generation Context Builder

## Purpose

The Generation Context Builder prepares all information required
for deterministic code generation.

Rather than querying multiple engines during generation,
all required engineering knowledge is assembled beforehand.

---

## Responsibilities

Collect

Approved Test Plan

Knowledge Graph

Discovery Artifacts

Business Flows

Component Metadata

Risk Scores

Capabilities

Environment Configuration

Naming Standards

Project Configuration

Generation Rules

AI Configuration

---

## Output

```

GenerationContext
{
contextId
executionId
planId
knowledgeGraph
discovery
workflows
components
risk
capabilities
configuration
environment
timestamp
}

```

The Generation Context SHALL remain immutable.

---

# 16. Input Validation Engine

## Purpose

Generation shall begin only after successful validation.

Invalid planning data shall never produce automation.

---

## Validation Rules

Validate

Schema Version

Planning Completeness

Knowledge Graph Availability

Discovery Completeness

Workflow Integrity

Configuration

Capability Registration

Environment Compatibility

Template Availability

Naming Rules

---

## Validation Results

PASS

WARNING

FAIL

Warnings may continue.

Failures terminate generation.

---

## Example

Missing Test Plan

↓

FAIL

↓

Generation Aborted

Reason

Planning not completed

---

# 17. Template Selection Engine

## Purpose

The framework generates automation using engineering templates.

Templates provide consistency,
maintainability,
and deterministic output.

---

## Responsibilities

Select templates for

Page Objects

Tests

Fixtures

Utilities

API Tests

Visual Tests *(OPTIONAL — DISABLED BY DEFAULT)*

Configuration Files

Reports

---

## Selection Factors

Application Type

Workflow Type

Risk Level

Technology Stack

Framework Configuration

Enabled Capabilities

Project Standards

User Preferences

---

## Example

Workflow

Login

↓

Template

Authentication Template

↓

Output

login.spec.ts

---

# 18. Workflow Generation Engine

## Purpose

Tests are generated from business workflows,
not individual pages.

Every workflow becomes one or more automation scenarios.

---

## Workflow Sources

Knowledge Graph

Business Flow Detection

Planning Engine

Risk Analysis

User Configuration

Historical Learning

---

## Workflow Expansion

Example

Workflow

Checkout

↓

Scenario

Successful Checkout

↓

Scenario

Invalid Payment

↓

Scenario

Coupon Applied

↓

Scenario

Guest Checkout

↓

Scenario

Payment Failure

Each scenario becomes an independent test.

---

# 19. Page Object Generation Engine

## Purpose

Every significant page shall receive
a dedicated Page Object.

Generated Page Objects shall encapsulate
all UI interactions.

---

## Responsibilities

Generate

Selectors

Actions

Navigation Methods

Assertions Helpers

Reusable Components

Synchronization Logic

Documentation

---

## Rules

Business logic SHALL NOT exist
inside test files.

Page Objects SHALL expose
high-level business actions.

Example

```

loginPage.login()

dashboardPage.openUsers()

checkoutPage.completePayment()

```

Direct locator usage inside tests
is discouraged.

---

# 20. Component Generator

Large applications often reuse UI components.

The framework SHALL generate
shared component objects.

Examples

Navigation Bar

Sidebar

Header

Footer

Modal

Search Panel

Date Picker

Pagination

File Upload

Toast Notification

Table

Chart

Each reusable component
shall exist only once.

---

# 21. Test Case Generator

## Purpose

The Test Case Generator transforms workflows
into executable Playwright tests.

Each generated test shall validate
one clear engineering objective.

---

## Responsibilities

Generate

Test Suites

Test Cases

Test Steps

Assertions

Setup

Cleanup

Metadata

Tags

Annotations

---

## Test Structure

```

Describe

↓

BeforeEach

↓

Test

↓

Assertions

↓

Cleanup

```

Generated tests shall follow
consistent formatting.

---

# 22. Test Naming Standards

Generated names shall be descriptive.

Examples

```

Login - Valid Credentials

Login - Invalid Password

Checkout - Successful Payment

User Management - Create User

Search - Filter Products

```

Avoid generic names such as

```

Test1

Scenario2

ExampleTest

```

Every name shall describe
the business objective.

---

## 22.1 Test Case Identity

### Ownership

**This engine owns Test Case Identity.**

Every Test Case has **exactly one canonical Test Case ID**, minted here.

This engine is responsible for **minting**, **preserving**, and **retiring** Test
Case IDs. No other component may mint one, and no competing identifier may stand
in for one.

Other components MAY reference a Test Case ID. Referencing confers no ownership.

### The Test Catalogue

The **Test Catalogue** is owned by this engine and is the **canonical identity
layer** for Generated Tests.

| Concern | What it is |
| ------- | ---------- |
| **Generated Tests** | The specification files this engine produces |
| **Test Catalogue** | The identity layer over those files — which behaviour each ID denotes |

Both belong to the `Generated Tests` canonical dataset already owned here. The
catalogue is **not** a second dataset, and there SHALL NEVER be a second
catalogue or a second identity system.

### Identity Denotes Behaviour, Not Implementation

A Test Case ID identifies **a behaviour**, not a file, a locator, a position, or
an implementation.

Two specifications that validate the same behaviour are the same Test Case and
carry the same ID. One ID SHALL NEVER denote two behaviours.

### Distinct From Knowledge Graph Entities

A Test Case is a **framework-generated artifact**, not an application fact. Pages,
routes, components, workflows and APIs exist whether or not anyone tests them; a
Test Case exists only because this engine created it.

| Concern | Owner |
| ------- | ----- |
| Test Case Identity | **06** (this engine) |
| Test Catalogue | **06** (this engine) |
| Knowledge Graph representation of Test Cases | 04 |
| Test Case coverage relationships | 04 |

### The Catalogue Is A Factual Record, Never A Recommendation (W8)

The Test Catalogue is an **internal identity and execution/audit record**. It
MAY record:

test ID · category · test case · target · execution status · evidence reference ·
failure information · retry information · timestamp / execution identity

It SHALL NEVER record, imply, or derive:

- "SQL injection applicable" / "XSS probably applicable" or any attack-class tag
- a probability, likelihood, or confidence that a surface is vulnerable
- a recommended, suggested, or applicable security skill
- a filtered security test plan
- a severity, CVSS score, or finding

There is **no** Skill Applicability Tagging layer and **no** Security Test
Catalogue in this framework. Security interpretation belongs to the separately
selected security skill (`01` §2.1,
`Architecture_Ownership_Matrix.md`).

`04` models Test Case nodes and relationships such as
`Test Case → covers → Checkout` and `Test Case → validates → Login`, **referencing
identifiers minted here**. `04` SHALL NOT mint them (`04` §15).

---

# 23. Metadata Generation

Every generated file shall include metadata.

Example

```

Generated By

Framework Version

Generation Timestamp

Workflow

Risk Level

Knowledge Graph Version

Template Version

Generation ID

```

Metadata improves
traceability,
auditing,
and incremental regeneration.

---

# 24. Generation Design Principles

Every generation subsystem shall satisfy

Single Responsibility

Immutable Inputs

Deterministic Output

Reusable Components

Observable Execution

Versioned Contracts

Independent Testing

Plugin Compatibility

Auditability

Future Extensibility

No Circular Dependencies

---

# End of Part 2


# ------------------------------------------------------------------
# Section 25 — Assertion Generation Engine
# ------------------------------------------------------------------

# 25. Assertion Generation Engine

## Purpose

Assertions verify that the application behaves as expected.

The framework SHALL generate meaningful business assertions rather than
simple existence checks.

Assertions shall validate business outcomes, not merely UI rendering.

---

## Responsibilities

The Assertion Generation Engine shall:

- Generate business-focused assertions
- Avoid redundant assertions
- Validate UI state
- Validate application state
- Validate API responses (where applicable)
- Validate semantic attributes — role, label, accessible name — as **functional
  UI assertions** (C7). This is not a WCAG compliance check and SHALL NOT be
  reported as accessibility coverage.
- Validate navigation outcomes
- Validate persisted data
- Generate negative assertions when required

---

## Assertion Types

Supported assertion categories include:

- Visibility
- Text Content
- Element State
- URL
- Navigation
- Form Validation
- API Response
- Semantic Attribute *(role / label / accessible name — functional, not WCAG)*
- Storage
- Download
- Upload
- Table Data
- Toast Messages
- Modal State
- Authentication
- Authorization

---

## Assertion Rules

Assertions SHALL:

- Be deterministic
- Be readable
- Be meaningful
- Avoid unnecessary duplication
- Clearly describe failures

Example

Preferred

```ts
await expect(page).toHaveURL("/dashboard");
await expect(successToast).toBeVisible();
await expect(userName).toHaveText("John Doe");
```

Avoid

```ts
expect(true).toBe(true);
```

---

# 26. Locator Resolution Engine

## Purpose

The Locator Resolution Engine determines the most reliable locator
strategy for every generated interaction.

It SHALL always prioritize stable, semantic selectors.

---

## Locator Priority

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

## Responsibilities

Select

Best locator

↓

Calculate confidence

↓

Record metadata

↓

Generate Playwright locator

↓

Register locator history

---

## Locator Metadata

Each generated locator includes:

- Locator Type
- Confidence Score
- Selection Reason
- Supporting Evidence
- Fallback Candidates

---

## Example

```ts
page.getByRole("button", { name: "Login" });
```

Confidence

98%

Reason

Accessible role with unique accessible name.

---

# 27. Fixture Generation Engine

## Purpose

Fixtures provide reusable setup and teardown logic.

They prevent duplicated initialization code across test suites.

---

## Responsibilities

Generate

- Browser Fixtures
- Authentication Fixtures
- API Fixtures
- Database Fixtures
- Test Data Fixtures
- Environment Fixtures
- Cleanup Fixtures

---

## Fixture Rules

Fixtures SHALL:

- Be reusable
- Be isolated
- Avoid hidden state
- Support parallel execution
- Minimize startup cost

---

## Example

```ts
test.use({
    storageState: "storage/admin.json"
});
```

---

# 28. Shared Utility Generator

## Purpose

Common functionality shall exist only once.

Utilities reduce duplicated code.

---

## Generated Utilities

Examples

Authentication Helper

Date Helper

Random Data Generator

Screenshot Helper

API Helper

Retry Helper

Download Helper

Upload Helper

Storage Helper

Environment Helper

---

## Utility Principles

Utilities SHALL

- Be stateless
- Be reusable
- Avoid business logic
- Be independently testable

---

# 29. Dependency Resolution Engine

## Purpose

Generated files frequently depend upon one another.

The Dependency Resolution Engine ensures all imports,
exports, references, and module relationships remain valid.

---

## Responsibilities

Resolve

Imports

Exports

Page Objects

Utilities

Fixtures

Shared Components

Type Definitions

Configuration

---

## Validation

Prevent

Broken imports

Circular dependencies

Unused modules

Duplicate utilities

Version conflicts

---

# 30. Project Assembly Engine

## Purpose

After generation,
all artifacts are assembled into a complete Playwright project.

---

## Responsibilities

Create

Project Structure

Configuration

Tests

Pages

Fixtures

Utilities

Reports Directory

Artifacts Directory

Documentation

---

## Example Structure

```
tests/

pages/

fixtures/

utils/

components/

data/

reports/

artifacts/

playwright.config.ts

README.md
```

---

# 31. Incremental Generation

## Purpose

Regenerating the entire project after every discovery is inefficient.

The framework supports incremental generation.

---

## Strategy

Detect Changes

↓

Compare Knowledge Graph Versions

↓

Identify Affected Workflows

↓

Regenerate Only Impacted Assets

↓

Validate Dependencies

↓

Update Metadata

---

## Benefits

Reduced execution time

Smaller code changes

Cleaner Git history

Improved maintainability

---

## 31.1 Identity Preservation Across Regeneration

Regeneration is the operation that threatens Test Case Identity. The five rules
below are **mandatory** and are owned by this engine.

### Rule 1 — Preserve

A behaviour that survives regeneration **SHALL retain its existing Test Case ID**.

```text
Generation 1:  TC-0010 = Login
Behaviour survives
Generation 2:  TC-0010 = Login
```

The ID SHALL NOT change because the test was regenerated, the file changed, a
locator changed, the test order changed, the implementation changed, or other
tests were added or removed.

### Rule 2 — Retire

When a behaviour is removed, its Test Case ID becomes **retired**.

```text
TC-0010 = Old behaviour
Behaviour removed
TC-0010 = RETIRED
```

A retired ID remains in the catalogue and remains traceable to the behaviour it
denoted. It SHALL NEVER be silently deleted from history.

### Rule 3 — Never Reuse

> **A retired Test Case ID SHALL NEVER be reused.**

```text
TC-0020 → RETIRED
New behaviour → SHALL NOT receive TC-0020
```

This holds even when the number becomes numerically available. An identifier
remains permanently bound to the behaviour it denoted.

**This rule exists because it was violated.** Across two executions of one
application, `TC-0020` denoted *"forgot-password link is reachable from the login
page"* and then *"valid credentials establish a session and land on the
dashboard."* Every record keyed on `TC-0020` silently de-referenced. Rule 3
prevents recurrence.

### Rule 4 — New Behaviours

A genuinely new behaviour receives the **next unused Test Case ID**.

**Unused** means never previously assigned — *including retired identifiers*.

```text
TC-0020 retired
TC-0021 active
New behaviour → TC-0022
```

Numeric gaps SHALL NEVER be filled. The catalogue is append-only in its
identifier space.

### Rule 5 — Catalogue / Specification Bijection

Every **active** catalogue entry corresponds to exactly one generated
specification, and every generated specification corresponds to exactly one
catalogue entry.

```text
1 active catalogue ID  ↔  1 generated specification
```

The validation gate (`01` §18) SHALL detect:

- duplicate catalogue IDs
- duplicate specifications
- a specification with no catalogue entry
- an active catalogue entry with no specification
- one ID denoting multiple behaviours
- multiple IDs denoting one surviving behaviour

Retired entries are exempt from bijection: they intentionally have no
specification.

---

## 31.2 Identity Lifecycle

```text
New behaviour
      ↓
06 mints a new Test Case ID
      ↓
Specification generated
      ↓
Regeneration ── behaviour survives ──► same ID   (Rule 1)
      │
      └──────── behaviour removed ───► ID retired (Rule 2)
                                            ↓
                                   NEVER reused  (Rule 3)
```

**Retired** is a catalogue condition of an identifier. It is **not** a framework
lifecycle state and introduces no state in `01` §17 and no phase in `01` §16.

---

## 31.3 Determinism

Identity assignment SHALL be deterministic.

Given the same prior catalogue, the same discovery delta, and the same identity
history, regeneration SHALL produce the same catalogue.

IDs SHALL NOT be random, timestamp-derived, or dependent on file order, worker
order, or generation order.

---

## 31.4 Catalogue Version Is Not Identity

```text
Catalogue Version  ≠  Test Case Identity
```

A **Catalogue Version** identifies a snapshot of the catalogue. A **Test Case ID**
identifies a behaviour *across* catalogue versions.

```text
Catalogue v1:  TC-0010 = Login
Catalogue v2:  TC-0010 = Login        ← identity stable
               TC-0011 = Logout       ← catalogue grew
```

The version changes as the catalogue evolves; the identity does not.

---

## 31.5 Historical Records

Historical records SHALL NEVER be silently remapped.

Where a historical record references an identifier whose meaning cannot be
reliably established under these rules, the record SHALL be **retained and
marked unmappable** — never deleted, and never assigned an invented mapping.

A mapping that cannot be proven SHALL NOT be asserted (`01` §31).

---

# 32. Duplicate Detection

The framework SHALL detect duplicated:

Tests

Assertions

Utilities

Fixtures

Components

Selectors

Page Objects

Imports

Duplicate code shall be consolidated automatically before output.

---

# 33. AI-Assisted Generation

## Purpose

Artificial Intelligence assists engineering decisions during generation.

AI SHALL NOT replace deterministic engineering rules.

---

## AI Responsibilities

Recommend

Naming

Assertions

Test Data

Comments

Documentation

Helper Methods

Edge Cases

Additional Scenarios

---

## AI Restrictions

AI SHALL NOT

Invent application behavior

Fabricate assertions

Generate fake results

Ignore planning decisions

Bypass framework policies

Every AI recommendation shall include

Confidence

Evidence

Reasoning Metadata

---

# 34. Code Quality Standards

Every generated project SHALL satisfy

✓ No duplicated logic

✓ No duplicated locators

✓ No TypeScript errors

✓ No lint violations

✓ No hardcoded waits

✓ Stable selectors

✓ Independent tests

✓ Parallel-safe execution

✓ Reusable fixtures

✓ Readable Page Objects

✓ Meaningful assertions

✓ Consistent formatting

---

# 35. Generation Metrics

The engine records metrics including

Generation Duration

Generated Files

Generated Tests

Generated Page Objects

Generated Fixtures

Generated Utilities

Generated Assertions

Average Locator Confidence

Average Assertion Count

Lines of Code

Template Usage

Validation Success Rate

---

# 36. Generation Events

The engine publishes events to the Event Bus.

Examples

GenerationStarted

TemplateSelected

PageObjectGenerated

FixtureGenerated

TestGenerated

AssertionGenerated

UtilityGenerated

ValidationStarted

ValidationCompleted

GenerationCompleted

GenerationFailed

These events enable plugins and reporting components to observe progress.

---

# End of Part 3

# ------------------------------------------------------------------

# Section 37 — Plugin Integration

# ------------------------------------------------------------------

# 37. Plugin Integration

## Purpose

The Test Generation Engine SHALL support extensibility through plugins.

Plugins may contribute additional generation capabilities without
modifying the core framework.

The core engine remains responsible for orchestration while plugins
extend supported functionality.

---

## Supported Plugin Types

Generation plugins may include:

- Page Object Plugins
- Test Template Plugins
- Assertion Plugins
- Locator Strategy Plugins
- API Generation Plugins
- Visual Testing Plugins *(OPTIONAL — DISABLED BY DEFAULT)*
- AI Generation Plugins
- Framework Adapter Plugins
- Reporting Plugins

---

## Plugin Lifecycle

Every plugin follows the lifecycle below.

```
Discover

↓

Load

↓

Validate Compatibility

↓

Register Capabilities

↓

Subscribe Events

↓

Execute

↓

Return Generated Assets

↓

Shutdown
```

---

## Plugin Requirements

Every plugin SHALL declare:

- Plugin Name
- Version
- Author
- Description
- Framework Compatibility
- Dependencies
- Supported Features
- Configuration Schema

---

## Plugin Isolation

Plugins SHALL NOT:

- Modify internal framework state
- Bypass the Decision Engine
- Override generated assets directly
- Access private framework APIs

Plugins communicate only through:

- Public Contracts
- Event Bus
- Capability Registry

---

# 38. Validation Integration

## Purpose

Generated automation shall be validated before execution.

Generation success does not guarantee executable automation.

Validation ensures that generated artifacts comply with engineering
standards before they reach the Execution Engine.

---

## Validation Categories

The Validation Engine shall verify:

### Code Validation

- TypeScript compilation
- Playwright syntax
- Import resolution
- Export resolution
- Duplicate declarations

---

### Architecture Validation

- Project structure
- Naming conventions
- Folder organization
- Dependency graph
- Circular dependencies

---

### Automation Validation

- Valid locators
- Valid assertions
- Fixture availability
- Page Object integrity
- Test independence

---

### Configuration Validation

- Browser configuration
- Environment variables
- Capability availability
- Required plugins
- Framework version compatibility

---

## Validation Outcomes

PASS

Generation may continue.

WARNING

Generation continues but records findings.

FAIL

Generation stops.

Execution SHALL NOT begin.

---

# 39. Security & Governance

> **Governance only (W8 / C5).** Secret masking, credential handling, scope
> enforcement, Rules of Engagement enforcement and evidence protection. This
> section governs the **safety of the generated code**; it does not authorize
> security testing of the target (§12).

## Purpose

Generated automation must follow secure engineering practices.

The Generation Engine SHALL never generate unsafe automation.

---

## Security Rules

Generated code SHALL NOT:

- Expose secrets
- Store passwords
- Hardcode API keys
- Log authentication tokens
- Log session cookies
- Disable security protections
- Circumvent authorization

---

## Sensitive Data Handling

Sensitive information shall be masked.

Examples include:

- Passwords
- API Keys
- OAuth Tokens
- Session IDs
- Personal Information
- Financial Data

---

## Governance Principles

Every generated artifact shall be:

- Traceable
- Auditable
- Reproducible
- Versioned
- Explainable

---

# 40. Audit & Traceability

## Purpose

Every generated artifact must be traceable to its engineering origin.

This enables reproducibility, debugging, compliance,
and historical analysis.

---

## Every Generated File Shall Include

**Test Case ID**

**Catalogue Version**

Generation ID

Execution ID

Correlation ID

Framework Version

Knowledge Graph Version

Planning Version

Template Version

Configuration Version

Timestamp

Workflow ID

Risk Level

Test Case ID and Catalogue Version are **additive**. Every field previously
required remains required and unchanged.

Test Case ID is defined by §22.1 and preserved by §31.1. Catalogue Version
identifies the catalogue snapshot the file was generated from (§31.4).

---

## Identity Traceability Chain

The Test Case ID SHALL remain traceable end to end:

```text
Test Catalogue
      ↓
Generated specification
      ↓
Execution
      ↓
Result
      ↓
Report  ·  Learning records
```

Because the ID is stable across regeneration (§31.1 Rule 1) and never reused
(Rule 3), a record keyed on it in `09` or `10` continues to denote the same
behaviour for the life of that behaviour.

This chain uses the existing traceability fields above. It creates **no new
canonical dataset**.

---

## Audit Record

Each generation produces an immutable audit record.

The audit contains:

Generation Request

↓

Inputs

↓

Templates Selected

↓

Generated Files

↓

Validation Results

↓

Warnings

↓

Errors

↓

Metrics

↓

Completion Status

---

## Audit Storage

Example

```
generation-history/

generation-0001.json

generation-0002.json

generation-0003.json
```

Audit records SHALL never be modified after persistence.

---

# 41. Future Extensibility

The architecture SHALL support future capabilities without redesign.

Examples include:

- Native Mobile Automation
- Desktop Application Automation
- Electron Automation
- Flutter Automation
- React Native Automation
- AI Visual Validation
- Autonomous Test Refactoring
- Mutation Testing
- Contract Testing
- Cloud Device Farms
- Distributed Generation
- Multi-language Generation
- Voice Interface Testing
- AR/VR Automation

New capabilities SHALL integrate through:

- Plugins
- Contracts
- Event Bus
- Capability Registry

Core architecture changes SHOULD NOT be required.

---

# 42. Success Criteria

The Test Generation Engine is considered successful if it consistently
produces automation that is:

✓ Deterministic

✓ Readable

✓ Maintainable

✓ Reusable

✓ Enterprise-ready

✓ Parallel-safe

✓ CI/CD compatible

✓ Versioned

✓ Fully traceable

✓ Plugin extensible

✓ AI-assisted but deterministic

✓ Free from duplicate logic

✓ Based on approved engineering plans

✓ Successfully validated before execution

✓ Ready for the Execution Engine

without requiring manual restructuring.

---

# Summary

The Test Generation Engine is responsible for transforming engineering
knowledge into production-quality Playwright automation.

It does not decide what should be tested and does not execute tests.

Instead, it converts the approved Test Plan, Knowledge Graph,
Discovery artifacts, and engineering rules into a complete,
validated, enterprise-grade automation project.

The generated project is then handed to the **Execution Engine**,
which becomes the next stage in the QA Automation Framework lifecycle.

---

# End of 06_Test_Generation_Engine.md
