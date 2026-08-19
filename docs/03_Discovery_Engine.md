# QA Automation Framework

# Discovery Engine Specification

**Document:** 03_Discovery_Engine.md

**Version:** 3.0

**Status:** Draft

**Depends On:** 01_Master_Architecture.md

**Controlled By:** 02_Decision_Engine.md

---

# Purpose

This document defines the architecture, responsibilities, execution
model, algorithms, governance, contracts, and quality standards of the
Discovery Engine.

The Discovery Engine is responsible for understanding an unknown web
application before any planning or test generation begins.

It transforms an unknown application into structured engineering
knowledge that can be consumed by the Knowledge Graph, Planning Engine,
and Test Generation Engine.

Discovery SHALL always occur before planning unless explicitly disabled
through framework configuration.

---

# Table of Contents

1. Purpose
2. Scope
3. Responsibilities
4. Architectural Position
5. Design Principles
6. Discovery Philosophy
7. Discovery Domains
8. Discovery Lifecycle
9. Discovery Context
10. Inputs
11. Outputs
12. Discovery Contracts
13. Non-Goals

---

# 1. Purpose

The Discovery Engine exists to answer one fundamental engineering
question:

"What does this application actually look like?"

Instead of generating automation immediately, the framework first builds
an understanding of the application.

Discovery converts an unknown website into structured knowledge.

The output becomes the foundation for every later engineering decision.

---

# 2. Scope

The Discovery Engine is responsible for discovering:

- Application entry points
- Routes
- Pages
- Navigation
- Components
- Forms
- Tables
- Dialogs
- Drawers
- Authentication
- Authorization indicators
- API endpoints
- Browser storage
- Cookies
- Session storage
- Local storage
- Service Workers
- Network activity
- Business workflows
- Client-side technologies
- Accessibility metadata
- Feature flags (when detectable)

The Discovery Engine SHALL NOT generate Playwright code.

The Discovery Engine SHALL NOT create Page Objects.

The Discovery Engine SHALL NOT execute assertions.

Those responsibilities belong to later engines.

---

# 3. Responsibilities

The Discovery Engine is responsible for answering questions such as:

What pages exist?

Which routes are reachable?

Which routes require authentication?

Which navigation paths exist?

Which components are interactive?

Which forms exist?

Which APIs are called?

Which storage mechanisms are used?

Which business workflows appear to exist?

Which technologies power the application?

Which components are reused?

Which pages appear critical?

Which elements are dynamically loaded?

Which areas require deeper analysis?

Every answer must be evidence-based.

---

# 4. Architectural Position

```

```

    User
                       │
                       ▼
               Configuration Layer
                       │
                       ▼
                 Decision Engine
                       │
                       ▼
                Discovery Engine
        ┌──────────┼──────────┐
        ▼          ▼          ▼
 Browser     Network Monitor   Storage
        │          │          │
        └──────────┼──────────┘
                   ▼
          Discovery Artifacts
                   │
                   ▼
           Knowledge Graph
                   │
                   ▼
           Planning Engine

```

The Discovery Engine never communicates directly with downstream
engines.

All execution decisions originate from the Decision Engine.

---

# 5. Design Principles

The Discovery Engine follows the principles defined by the Master
Architecture.

Additional principles include:

## 5.1 Observe Before Acting

Discovery is read-only.

It MAY navigate and activate navigation disclosures (menus, tabs, expanders),
because many applications render navigation only on activation and a link-only
crawl silently misses most of the application. It MUST NOT submit a form, and
MUST NOT activate a control whose accessible name matches destructive or
write intent.

It must never modify application data unless explicitly configured.

---

## 5.2 Same Input → Same Output

Given identical application state and configuration, discovery shall
produce equivalent outputs.

Random crawling behavior is prohibited.

---

## 5.3 Evidence First

Every discovered object must include supporting evidence.

Example:

Page

Dashboard

Evidence

Navigation Link

/dashboard

Observed

Navigation Menu

Confidence

99%

---

## 5.4 Non-Destructive

Default discovery must never:

Delete data

Create users

Submit payments

Trigger destructive APIs

Modify production records

---

## 5.5 Explainability

Every discovery result shall explain:

How it was found

Why it was classified

Confidence

Supporting observations

---

# 6. Discovery Philosophy

Discovery follows a progressive understanding model.

Observe

↓

Collect

↓

Classify

↓

Relate

↓

Validate

↓

Persist

↓

Publish

↓

Learn

Each stage increases application understanding.

Discovery SHALL NEVER skip classification.

---

# 7. Discovery Domains

Discovery operates across multiple domains.

## Navigation Discovery

Examples

Menus

Breadcrumbs

Sidebar

Header

Footer

Deep Links

---

## Route Discovery

Examples

URL Patterns

Dynamic Routes

Redirects

Protected Routes

Nested Routes

---

## Component Discovery

Examples

Buttons

Forms

Inputs

Tables

Cards

Tabs

Modals

Accordions

Drawers

---

## Data Discovery

Examples

API Calls

GraphQL

REST

WebSockets

Server Events

---

## Storage Discovery

Examples

Cookies

Session Storage

Local Storage

IndexedDB

Cache Storage

---

## Technology Discovery

Examples

React

Angular

Vue

Next.js

Nuxt

Svelte

ASP.NET

Laravel

Django

Spring

---

## Accessible Locator Metadata

Captured for locator and component identification (Playwright `getByRole` /
accessible-name locators) and structural understanding — **not** for accessibility
testing, which is out of scope for the Site Explorer.

Examples

ARIA

Roles

Labels

Landmarks

Heading Structure

Focus Order

---

# 8. Discovery Lifecycle

Every discovery execution follows the same lifecycle.

Request

↓

Initialize Browser

↓

Load Configuration

↓

Seed URLs

↓

Crawl Application

↓

Observe Runtime

↓

Collect Artifacts

↓

Classify Findings

↓

Validate Results

↓

Persist Artifacts

↓

Publish Events

↓

Return Discovery Report

No lifecycle stage may be skipped.

---

# 9. Discovery Context

Every discovery request receives an immutable Discovery Context.

Example

```

DiscoveryContext
{
    contextId
    executionId
    correlationId
    application
    environment
    configuration
    browserProfile
    authentication
    capabilities
    runtime
    timestamp
}

```

Once created, the Discovery Context SHALL NOT be modified.

---

# 10. Inputs

The Discovery Engine accepts structured contracts only.

Primary inputs include:

Configuration

Seed URLs

Authentication Context

Capability Registry

Browser Configuration

Environment Metadata

Execution Metadata

Feature Flags

User Overrides

Plugin Configuration

Each input shall include:

Schema Version

Timestamp

Correlation ID

Producer

Validation Status

Payload

Invalid contracts SHALL be rejected.

---

# 11. Outputs

Discovery produces structured artifacts.

Minimum outputs include:

application.json

pages.json

routes.json

navigation.json

components.json

forms.json

apis.json

storage.json

authentication.json

discovery-report.json

technology.json

metrics.json

Each artifact is independently versioned.

Outputs become inputs for the Knowledge Graph.

---

# 12. Discovery Contracts

Every Discovery module communicates through immutable contracts.

Example

Crawler

↓

Discovery Contract

↓

Classifier

↓

Validated Discovery

↓

Knowledge Graph

Each contract contains:

Schema Version

Producer

Consumer

Timestamp

Correlation ID

Validation Result

Payload

Direct object mutation is prohibited.

---

# 13. Non-Goals

The Discovery Engine SHALL NOT:

Generate Playwright tests

Generate Page Objects

Execute assertions

Modify business data

Perform penetration testing

Perform load testing

Perform destructive actions

Generate reports

Apply self-healing

Retry failed tests

Make planning decisions

These responsibilities belong to their respective framework components.

---

# Summary

The Discovery Engine is responsible for transforming an unknown web
application into structured engineering knowledge.

It discovers routes, pages, navigation, components, APIs, storage,
authentication, technologies, and runtime behavior while remaining
strictly read-only (navigation and disclosure activation only).

Its outputs become the foundation for the Knowledge Graph and all
subsequent planning and test generation.

The following sections define:

- Internal Architecture
- Crawl Engine
- URL Normalization Engine
- Navigation Discovery
- Component Discovery
- API Discovery
- Authentication Discovery
- Technology Detection
- Discovery State Machine
- Discovery Validation
- Persistence
- Event Integration
- Audit Model

---

# End of Part 1

# ------------------------------------------------------------------
# Section 14 — Internal Architecture
# ------------------------------------------------------------------

# 14. Internal Architecture

The Discovery Engine is not a single crawler.

It is a collection of specialized discovery subsystems working together
to construct a complete understanding of the application.

Each subsystem owns one bounded responsibility.

Subsystems communicate only through immutable discovery contracts.

The Discovery Engine SHALL remain deterministic, observable,
auditable, and extensible.

---

## 14.1 Internal Component Diagram

```

    Discovery Request
                          │
                          ▼
                 Discovery Coordinator
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
 Context Builder   Configuration Loader   Session Manager
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ▼
                    Crawl Engine
                          │
        ┌─────────────────┼───────────────────────────┐
        ▼                 ▼                           ▼
 Route Discovery   Navigation Discovery     Runtime Observer
        │                 │                           │
        └─────────────────┼───────────────────────────┘
                          ▼
                Component Discovery Engine
                          │
        ┌─────────────────┼───────────────────────────┐
        ▼                 ▼                           ▼
 API Discovery    Storage Discovery      Authentication Discovery
        │                 │                           │
        └─────────────────┼───────────────────────────┘
                          ▼
             Technology Detection Engine
                          │
                          ▼
              Discovery Validation Engine
                          │
                          ▼
                Artifact Persistence Engine
                          │
                          ▼
                     Event Publisher
                          │
                          ▼
                   Knowledge Graph

```

No subsystem may bypass another subsystem without approval from the
Decision Engine.

---

# 15. Discovery Coordinator

## Purpose

The Discovery Coordinator manages the complete discovery lifecycle.

It is responsible for orchestrating every discovery subsystem.

It SHALL NOT perform crawling itself.

---

## Responsibilities

The coordinator shall:

- Receive discovery requests
- Initialize execution context
- Load configuration
- Create browser sessions
- Schedule discovery tasks
- Track discovery progress
- Detect failures
- Collect subsystem outputs
- Produce final discovery status

---

## Coordinator Principles

The coordinator SHALL:

Remain stateless between executions.

Never perform classification.

Never modify discovery artifacts.

Never make planning decisions.

---

## Coordinator Outputs

The coordinator produces:

- Discovery Status
- Execution Metrics
- Discovery Timeline
- Discovery Events
- Final Discovery Report

---

# 16. Context Builder

## Purpose

The Context Builder creates the immutable Discovery Context used by
every discovery subsystem.

The Discovery Engine shall never directly query arbitrary modules during
execution.

Instead, all required information is assembled before discovery begins.

---

## Discovery Context

Example

```text
DiscoveryContext
{
    contextId
    executionId
    correlationId
    application
    environment
    configuration
    browser
    authentication
    capabilities
    runtime
    plugins
    timestamp
}
```

---

## Responsibilities

Collect:

Configuration

Execution Metadata

Authentication State

Browser Configuration

Environment Information

Enabled Plugins

Capability Registry

Runtime Parameters

Feature Flags

User Overrides

---

Once created,

DiscoveryContext SHALL NOT be modified.

---

# 17. Crawl Engine

## Purpose

The Crawl Engine is responsible for systematically exploring the
application.

Its objective is maximum application understanding while minimizing
duplicate exploration.

---

## Responsibilities

The Crawl Engine shall:

Visit seed URLs

Discover hyperlinks

Observe client-side navigation

Handle SPA routing

Queue newly discovered routes

Respect crawl policies

Avoid duplicate pages

Track crawl depth

Collect crawl metadata

---

## Crawl Principles

The crawler SHALL:

Remain deterministic.

Stay within configured scope.

Respect maximum depth.

Respect exclusion rules.

Avoid logout actions.

Avoid destructive actions.

---

## Crawl Outputs

The Crawl Engine produces:

Discovered URLs

Route Graph

Navigation Graph

Visited Pages

Unvisited Queue

Discovery Statistics

---

# 18. URL Normalization Engine

## Purpose

Different URLs may represent the same logical page.

The URL Normalization Engine removes duplicates before discovery
continues.

---

## Responsibilities

Normalize:

Trailing slashes

Case sensitivity

Default ports

Duplicate query parameters

Tracking parameters

Fragment identifiers

Repeated path separators

Relative paths

---

## Example

Input

```text
/products?id=10&utm_source=email
```

Normalized

```text
/products?id=10
```

---

## Benefits

Reduces duplicate crawling.

Improves discovery speed.

Produces stable Knowledge Graph nodes.

Improves coverage calculations.

---

# 19. Route Discovery Engine

## Purpose

The Route Discovery Engine discovers every reachable route within the
application.

Routes may originate from:

Links

JavaScript navigation

Forms

Redirects

API responses

Client-side routers

---

## Responsibilities

Identify:

Static Routes

Dynamic Routes

Protected Routes

Nested Routes

Redirect Routes

Parameterized Routes

404 Routes

Error Pages

---

## Route Metadata

Every discovered route shall include:

Route ID

URL

Parent Route

Discovery Method

Authentication Required

HTTP Status

Confidence

Timestamp

---

## Route Relationships

Routes may reference:

Parent Route

Child Route

Redirect Target

Navigation Source

Workflow

Knowledge Graph Node

---

# 20. Navigation Discovery Engine

## Purpose

The Navigation Discovery Engine understands how users move throughout
the application.

Navigation is essential for workflow detection.

---

## Navigation Sources

Navigation may originate from:

Header menus

Sidebar menus

Footer links

Breadcrumbs

Tabs

Cards

Buttons

Context menus

Client-side routing

Keyboard shortcuts (where observable)

---

## Navigation Metadata

Each navigation element records:

Navigation ID

Label

Source Page

Destination

Navigation Type

Visibility

Enabled State

Confidence

---

## Navigation Relationships

Navigation nodes connect:

Pages

Components

Business Workflows

User Roles

Knowledge Graph entities

---

# 21. Component Discovery Engine

## Purpose

Raw HTML elements provide little engineering value.

The Component Discovery Engine identifies meaningful application
components.

---

## Responsibilities

Discover:

Buttons

Inputs

Text Areas

Dropdowns

Tables

Cards

Accordions

Tabs

Date Pickers

Dialogs

Modals

Drawers

Pagination

Infinite Scroll

Search Controls

Charts

File Uploads

Rich Text Editors

---

## Component Metadata

Each component shall include:

Component ID

Component Type

HTML Tag

Role

Accessible Name

Locator Candidates

Parent Component

Page

Confidence

Discovery Evidence

---

## Component Relationships

Components may reference:

Parent Component

Child Components

Associated Forms

Associated APIs

Associated Workflows

Knowledge Graph Nodes

---

# 22. API Discovery Engine

## Purpose

Modern applications rely heavily on APIs.

The API Discovery Engine observes runtime communication without altering
application behavior.

---

## Responsibilities

Observe:

REST APIs

GraphQL

WebSockets

SSE

XHR

Fetch Requests

Background Synchronization

Third-party APIs

---

## API Metadata

Each API shall include:

API ID

Method

Endpoint

Status Code

Content Type

Request Size

Response Size

Authentication Required

Average Duration

Observed Frequency

Confidence

---

## API Relationships

Each API may relate to:

Pages

Components

Forms

Business Workflows

Authentication

Knowledge Graph Nodes

---

# 23. Authentication Discovery Engine

## Purpose

Authentication significantly influences application behavior.

The Discovery Engine shall identify authentication mechanisms without
attempting to bypass them.

---

## Responsibilities

Detect:

Login Pages

Logout Endpoints

Session Cookies

JWT Tokens

OAuth Flows

OIDC Flows

SSO Indicators

MFA Indicators

Role-based Navigation

Permission Boundaries

Session Expiration

---

## Authentication Metadata

Each authentication mechanism records:

Authentication ID

Mechanism

Provider

Protected Routes

Session Type

Token Storage

Cookie Names

Confidence

---

## Principles

The engine SHALL NEVER:

Bypass authentication.

Guess credentials.

Brute-force accounts.

Modify authentication state.

---

# 24. Storage Discovery Engine

## Purpose

Applications store important runtime information within browser storage.

Understanding storage improves workflow detection and diagnostics.

---

## Responsibilities

Discover:

Cookies

Session Storage

Local Storage

IndexedDB

Cache Storage

Service Worker Cache

Storage Events

Browser Quotas

---

## Storage Metadata

Each storage object records:

Storage ID

Storage Type

Key

Value Type

Expiration

Origin

Security Attributes

Observed Usage

Confidence

Sensitive values SHALL be masked before persistence.

---

## Discovery Principles

The Storage Discovery Engine SHALL:

Observe only.

Never modify stored values.

Never expose secrets.

Never persist sensitive information unmasked.

---

# End of Part 2

# ------------------------------------------------------------------

# Section 25 — Runtime Observation Engine

# ------------------------------------------------------------------

# 25. Runtime Observation Engine

## Purpose

Modern web applications change continuously during execution.

Static HTML alone cannot accurately represent application behavior.

The Runtime Observation Engine monitors application behavior while pages
are executing to discover dynamic content and runtime interactions.

---

## Responsibilities

Observe:

DOM mutations

AJAX activity

Lazy loading

Client-side rendering

Timers

Animations

JavaScript events

Resource loading

Background tasks

Dynamic component creation

---

## Observation Sources

The engine observes:

Browser Events

DOM Mutation Observer

Performance API

Network API

Console Events

Navigation Events

Resource Timing API

---

## Runtime Outputs

The engine produces:

runtime.json

dom-events.json

mutation-report.json

performance-events.json

dynamic-components.json

---

# 26. Technology Detection Engine

## Purpose

Understanding the technology stack enables intelligent automation
generation.

---

## Responsibilities

Identify:

Frontend Framework

Backend Framework (when observable)

Rendering Strategy

Bundler

Package Signatures

CSS Framework

UI Libraries

Chart Libraries

Authentication Libraries

State Management Libraries

Analytics Providers

Monitoring SDKs

Payment SDKs

---

## Example

Application

↓

Detected

React

↓

Router

React Router

↓

Build Tool

Vite

↓

State

Redux Toolkit

↓

Charts

Chart.js

---

## Detection Sources

Evidence includes:

HTML

JavaScript Bundles

Global Variables

Meta Tags

Script URLs

DOM Attributes

Source Maps (if available)

HTTP Headers

---

# 27. SPA Detection Engine

## Purpose

Single Page Applications behave differently from traditional websites.

The framework shall detect SPA behavior automatically.

---

## Detection Indicators

History API usage

Client-side routing

Root application container

Bundle loading

Hydration events

Dynamic route transitions

Minimal page reloads

---

## Supported Types

Traditional Multi-Page Application

Single Page Application

Hybrid Application

Static Site

Progressive Web Application

Micro Frontend

---

## Outputs

spa.json

application-type.json

routing-model.json

---

# 28. Dynamic Content Discovery

## Purpose

Many components appear only after user interaction.

The Discovery Engine shall identify dynamic content safely.

---

## Dynamic Elements

Infinite Scroll

Lazy Images

Deferred Components

Virtual Lists

Collapsible Sections

Context Menus

Hover Menus

Tabs

Modals

Drawers

Tooltips

Popovers

---

## Discovery Strategy

Observe

↓

Trigger Safe Interaction

↓

Observe DOM Changes

↓

Record New Components

↓

Classify

↓

Persist

Only non-destructive interactions are permitted.

---

# 29. Discovery Validation Engine

## Purpose

Discovery outputs must be validated before they become part of the
Knowledge Graph.

---

## Validation Rules

Validate:

Duplicate Routes

Duplicate Pages

Broken Relationships

Missing IDs

Invalid URLs

Circular References

Missing Metadata

Schema Compliance

Artifact Completeness

Timestamp Validity

---

## Validation Outcomes

PASS

WARNING

FAIL

Warnings may continue.

Failures stop discovery.

---

## Example

Duplicate Route IDs

↓

FAIL

↓

Discovery Halted

↓

Diagnostic Report Generated

---

# 30. Discovery State Machine

Every discovery execution maintains an explicit state.

Allowed States

INITIALIZING

↓

LOADING_CONFIGURATION

↓

BUILDING_CONTEXT

↓

STARTING_BROWSER

↓

CRAWLING

↓

OBSERVING_RUNTIME

↓

CLASSIFYING

↓

VALIDATING

↓

PERSISTING

↓

PUBLISHING_EVENTS

↓

COMPLETED

---

## Failure States

CONFIGURATION_FAILED

BROWSER_FAILED

DISCOVERY_FAILED

VALIDATION_FAILED

PERSISTENCE_FAILED

EVENT_PUBLICATION_FAILED

FAILED

CANCELLED

Every transition shall be logged.

---

# 31. Discovery Persistence

## Purpose

Discovery artifacts become engineering assets.

Every successful execution shall persist discovery outputs.

---

## Persistence Structure

Example

```
discovery/

    application.json

    pages.json

    routes.json

    navigation.json

    components.json

    forms.json

    apis.json

    storage.json

    technology.json

    runtime.json

    discovery-report.json

    metrics.json
```

---

## Versioning

Every execution creates a new snapshot.

Example

```
discovery/

v1/

v2/

v3/
```

Historical snapshots shall never be overwritten.

---

# 32. Discovery Metrics

Every execution produces measurable metrics.

Examples

Pages Discovered

Routes Discovered

Components Discovered

Forms Discovered

API Endpoints

Navigation Links

Storage Objects

Discovery Duration

Average Crawl Time

Average Response Time

Coverage Percentage

Duplicate Reduction Rate

Discovery Confidence

Technology Detection Confidence

---

## Usage

Metrics support:

Planning

Risk Analysis

Learning Repository

Framework Health

Historical Comparisons

---

# 33. Event Bus Integration

The Discovery Engine communicates exclusively through the Event Bus.

---

## Lifecycle Events

DiscoveryRequested

DiscoveryStarted

DiscoveryProgressUpdated

DiscoveryPaused

DiscoveryResumed

DiscoveryCompleted

DiscoveryCancelled

DiscoveryFailed

---

## Entity Events

RouteDiscovered

PageDiscovered

ComponentDiscovered

FormDiscovered

ApiDiscovered

NavigationDiscovered

StorageDiscovered

TechnologyDetected

AuthenticationDetected

---

## Validation Events

ValidationStarted

ValidationPassed

ValidationFailed

ArtifactPersisted

---

All published events shall include:

Event ID

Timestamp

Execution ID

Correlation ID

Producer

Payload

---

# 34. Discovery Audit

Every discovery execution shall produce an immutable audit record.

---

## Audit Contents

Execution ID

Correlation ID

Framework Version

Configuration Version

Discovery Version

Knowledge Graph Version

Start Time

End Time

Duration

Visited URLs

Skipped URLs

Discovered Entities

Warnings

Errors

Validation Status

Generated Artifacts

Metrics Summary

---

## Audit Principles

Audit records SHALL:

Be immutable

Be timestamped

Be reproducible

Support compliance

Support debugging

---

# 35. Discovery Quality Gates

Before discovery begins

✓ Configuration Valid

✓ Browser Available

✓ Authentication Ready

✓ Seed URLs Available

✓ Capability Registered

---

During discovery

✓ Crawl Progress Recorded

✓ Runtime Observed

✓ Components Classified

✓ Metrics Collected

✓ Events Published

---

Before completion

✓ Validation Passed

✓ Artifacts Persisted

✓ Audit Generated

✓ Metrics Saved

✓ Discovery Report Created

If any mandatory gate fails,

Discovery shall terminate safely.

---

# 36. Discovery Security Rules

Discovery SHALL remain read-only by default: navigation and disclosure
activation are permitted; form submission and destructive/write-intent controls
are not. Discovery SHALL be depth-first and exhaustive — no URL cap, no
per-category cap, no truncation. Any item not explored SHALL be recorded as a
coverage-ledger exclusion with a reason, never omitted silently.

The engine SHALL NOT:

Delete records

Create accounts

Submit payments

Upload files

Modify production data

Trigger destructive APIs

Bypass authentication

Store secrets in artifacts

Sensitive information shall be masked before persistence.

---

# 37. Integration with Knowledge Graph

The Discovery Engine does not build the Knowledge Graph.

It produces validated discovery artifacts.

The Knowledge Graph consumes those artifacts.

Flow

Discovery

↓

Validated Artifacts

↓

Knowledge Graph Builder

↓

Graph Validation

↓

Knowledge Graph Snapshot

Discovery never modifies existing graph versions.

---

# 38. Discovery Example

Configuration Loaded

↓

Seed URL

https://example.com

↓

Application Crawled

↓

Routes Identified

↓

Navigation Collected

↓

Components Classified

↓

REST APIs Observed

↓

Authentication Detected

↓

Technology Stack Identified

↓

Artifacts Validated

↓

Artifacts Persisted

↓

Events Published

↓

Discovery Completed

---

# 39. Architecture Compliance

Every Discovery Engine implementation SHALL satisfy:

✓ Deterministic Execution

✓ Immutable Contracts

✓ Structured Outputs

✓ Versioned Artifacts

✓ Safe Observation

✓ Complete Audit Trail

✓ Event-Based Communication

✓ Enterprise Scalability

✓ Explainable Discovery

✓ Reproducible Results

Any implementation violating these principles is non-compliant with
Framework Architecture v3.0.

---

# 40. Summary

The Discovery Engine is the foundation of the QA Automation Framework.

It transforms an unknown application into structured engineering
knowledge by discovering pages, routes, navigation, components,
authentication, APIs, browser storage, runtime behavior, and technology
stack.

Its outputs are deterministic, validated, versioned, auditable, and
consumed by the Knowledge Graph, enabling intelligent planning,
generation, execution, and continuous learning.

Discovery is read-only, safe by default, and governed by the
Decision Engine.

---

# End of 03_Discovery_Engine.md
