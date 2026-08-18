
Absolutely. Based on everything we've designed together, **this is not just a Claude skill** anymore. It is becoming an **AI QA Engineering Platform**. Think of it as a senior QA engineer + automation architect + test analyst + execution engine working together.

I'll explain it the same way you could explain it to your seniors.

---

# What is this Skill?

**This is the backend methodology for the RedOps Site Explorer.**

Imagine hiring a **Senior QA Automation Engineer with 15+ years of experience**.

Now imagine that engineer never gets tired, always follows the same standards, learns from previous executions, generates automation automatically, and produces enterprise-grade reports.

That is what this skill is trying to become.

It is **not**:

* a Playwright script generator
* a Selenium wrapper
* an AI chatbot
* a test case generator

It is an **AI-driven QA Engineering Framework** — and, in RedOps, the engine
behind the Site Explorer.

## What it is *not* responsible for

**Security assessment.** This skill discovers and verifies. It never decides
that something is vulnerable, never suggests which security skill to run, and
never produces a finding.

```
qa-automation (Site Explorer)         then, separately:
─────────────────────────────         ────────────────────────────────────
Discovery                             User selects a security skill
  → Application understanding           → RedOps mounts it
  → QA test planning                    → It reads Site Explorer evidence
  → QA test generation                  → It picks its own targets
  → Playwright execution                → It picks its own methodology
  → QA results                          → Security execution → validation
  → Evidence / reports                  → Finding

"What exists in the application,      "Given the selected methodology, what
 and what evidence do we have?"         security tests should be performed?"
```

Removed from this skill: accessibility testing · performance testing ·
responsive testing · cross-browser testing · security testing · passive security
scanning · CVSS/severity/findings · security-skill recommendation.

---

# Main Goal

Instead of telling AI

> "Write a Playwright test."

you simply say

> "Test this application."

The framework figures out everything else.

---

# Complete QA Lifecycle Coverage

It covers almost the entire QA lifecycle.

```
Requirement
      │
      ▼
Understand Application
      │
      ▼
Discover UI
      │
      ▼
Build Knowledge Graph
      │
      ▼
Risk Analysis
      │
      ▼
Test Planning
      │
      ▼
Generate Tests
      │
      ▼
Execute Tests
      │
      ▼
Retry
      │
      ▼
Self Healing
      │
      ▼
Collect Evidence
      │
      ▼
Analyze Failures
      │
      ▼
Generate Reports
      │
      ▼
Learn From Execution
      │
      ▼
Improve Next Run
```

Instead of treating QA as separate activities, the framework connects them into one continuous workflow.

---

# How It Works

## Step 1 — Explore the Application

First it explores the application to understand it.

It discovers

* pages and links
* routes, including dynamic route templates
* `robots.txt` and `sitemap.xml` entries
* JavaScript routes and GraphQL endpoints
* hidden endpoints (found in JS/sitemap/robots, not page-linked)
* forms, inputs and parameters
* buttons, navigation and components
* authentication surfaces
* network / API traffic, with masked request and response capture
* user roles and business flows

Instead of blindly clicking buttons, it understands what the application is.

### What it guarantees — and what it doesn't

It does **not** promise "every page". It promises **all reachable, in-scope
surfaces within the configured exploration budget**.

Anything discovered but not reached is written down with a reason:

```
inaccessible · blocked · capped · excluded
unavailable state · unavailable credentials
```

### Two exploration modes

| Mode | Credentials | Explores |
|---|---|---|
| **Unauthenticated Exploration** | none | publicly reachable in-scope surfaces |
| **Authenticated Exploration** | supplied | plus the authenticated surfaces that account can reach |

Neither is *authorization*. Approved scope is mandatory either way.

### It records facts, not conclusions

✅ "A parameter exists." ✅ "A GraphQL endpoint was observed."
❌ "This parameter is injectable." ❌ "Run the injection skill here."

The second kind is the security skill's job, later, outside this skill.

---

## Step 2 — Build Knowledge

It creates an internal model.

Think of it like Google Maps.

Instead of roads, it maps

```
Login

↓

Dashboard

↓

Employee

↓

Leave

↓

Recruitment

↓

Reports
```

Now AI knows

* where everything is
* how pages connect
* what depends on what

---

## Step 3 — Understand Risk

Now it asks

Which areas are most important?

Example

```
Login
★★★★★

Payment
★★★★★

Delete Employee
★★★★★

Theme Switch
★☆☆☆☆
```

High-risk areas get more testing.

---

## Step 4 — Plan Testing

Instead of running random tests,

it creates a plan.

Example

```
Critical

↓

High

↓

Medium

↓

Low
```

This saves execution time.

---

## Step 5 — Generate Automation

Now AI generates

* Playwright tests
* Page Objects
* fixtures
* reusable functions
* assertions

Everything follows the architecture rules.

---

## Step 6 — Execute Tests

Now it runs the tests.

During execution it monitors

* browser
* application
* timing
* failures
* environment
* screenshots
* videos
* traces
* logs

Everything is recorded.

---

## Step 7 — Retry

If something fails

it doesn't immediately say

"Test Failed"

It asks

```
Is this temporary?

Network?

Timing?

Animation?

Browser?

Server?
```

Then retries intelligently.

---

## Step 8 — Self Healing

Suppose

```
Login Button

Old locator

↓

#loginBtn
```

Developer changes it

```
button.submit
```

Instead of failing immediately

the framework searches for

* role
* label
* accessible name
* text
* placeholder
* nearby elements

Then repairs the locator automatically.

> Note: role, label and accessible name are **locator and self-healing
> evidence**. They are why removing *accessibility testing* did not remove
> accessibility *semantics* — the healing score still weights an accessibility
> match at 0.20.

---

## Step 9 — Evidence Collection

Every execution stores evidence.

Example

```
Screenshot

Video

Trace

Logs

DOM

Semantic / role metadata

Network (masked request + response)

Timing
```

Nothing is hidden — and nothing sensitive is stored raw. Secrets, tokens,
cookies, auth headers and PII are masked **before** anything is written to
disk.

---

## Step 10 — Failure Analysis

Instead of

```
Test Failed
```

it explains

```
Why?

Where?

What changed?

Who is responsible?

Application?

Environment?

Framework?

Test?

Infrastructure?
```

This saves developers and QA engineers significant investigation time.

---

## Step 11 — Reporting

Instead of a simple HTML report

it produces

Executive Report

↓

Coverage Report

↓

Risk Report

↓

Failure Report

↓

**Exploration Disclosure** — what was explored, what wasn't, and why

↓

Learning Report

↓

Optimization Report

Managers, developers and QA engineers each get the information they need. The
Exploration Disclosure is mandatory: it names the exploration mode, the budget,
every cap that was hit, and every surface that went unreached with its reason. A
capped or credential-limited run is never presented as complete.

---

## Step 12 — Learning

This is one of the biggest differences.

Normal automation

```
Run

↓

Forget
```

This framework

```
Run

↓

Learn

↓

Remember

↓

Improve

↓

Next Run Better
```

Over time it learns

* good locators
* flaky tests
* recovery strategies
* execution history
* browser reliability
* workflow behavior
* optimization opportunities

---

# Which QA Activities Does It Cover?

Almost every major QA activity.

| QA Activity                        | Covered                        |
| ---------------------------------- | ------------------------------ |
| Application Exploration            | ✅ (bounded, evidence-backed)  |
| Test Planning                      | ✅                             |
| Requirement Traceability (planned) | 🟡                             |
| Functional Testing                 | ✅                             |
| Smoke Testing                      | ✅                             |
| Sanity Testing                     | ✅                             |
| Regression Testing                 | ✅                             |
| End-to-End Testing                 | ✅                             |
| UI Testing                         | ✅                             |
| Forms Testing                      | ✅                             |
| Authentication Testing             | ✅                             |
| Navigation Testing                 | ✅                             |
| API Testing                        | ✅                             |
| Dashboard Testing                  | ⚪ optional — off by default   |
| Table Testing                      | ⚪ optional — off by default   |
| Visual Testing                     | ⚪ optional — off by default   |
| Cross Browser Testing              | ❌ not this skill's job        |
| Accessibility Testing              | ❌ not this skill's job        |
| Performance Testing                | ❌ not this skill's job        |
| Responsive Testing                 | ❌ not this skill's job        |
| Security Testing                   | ❌ separate security skill     |
| Mobile Testing                     | Future                         |
| Database Validation                | Future                         |
| CI/CD Integration                  | ✅                             |
| Reporting                          | ✅                             |
| Learning                           | ✅                             |

---

# What Types of Testing Can It Perform?

Exactly ten categories.

| # | Category | Default |
|---|---|---|
| 1 | Smoke | on |
| 2 | Functional | on |
| 3 | UI | on |
| 4 | Forms | on |
| 5 | Authentication | on |
| 6 | Navigation | on |
| 7 | API | on |
| 8 | Dashboard | **off** |
| 9 | Table | **off** |
| 10 | Visual | **off** |

**Functional** breaks down into:

* Positive testing
* Boundary testing
* Business-rule testing
* CRUD testing — **except Delete**
* State-transition testing

Also covered inside those categories: workflow testing, form validation, session
testing, error handling, role-based application behaviour, negative testing,
retry validation and self-healing validation.

**On Delete.** The explorer records *that* delete functionality exists — that is
a fact worth knowing. It never generates a test that deletes application data. A
test that creates its own synthetic data still cleans that data up afterwards;
that is housekeeping, not a Delete test.

**Never generated:** SQL injection · XSS · SSRF · IDOR · command injection ·
path traversal · accessibility/WCAG checks · performance budgets · viewport
matrices · cross-browser matrices.

---

# What Makes It Different?

Most automation tools do this:

```
Generate Test

↓

Run

↓

Fail
```

Your framework does this:

```
Understand

↓

Model

↓

Plan

↓

Generate

↓

Execute

↓

Retry

↓

Heal

↓

Collect Evidence

↓

Analyze

↓

Report

↓

Learn

↓

Improve
```

That's a much more complete engineering workflow.

---

# Why So Many Documents?

Each document has a single responsibility.

For example:

* **01** defines the overall architecture.
* **02** makes decisions.
* **04** models application knowledge.
* **05** decides what to test.
* **06** generates tests.
* **07** executes them.
* **08** heals failures and locators.
* **09** reports results.
* **10** stores learning.
* **13** detects application changes.
* **16** defines the QA engineering methodology.

Because each capability has exactly one owner, the framework stays modular, avoids duplicated logic, and is easier to maintain.

---

# How Much of the QA Lifecycle Does It Cover?

If we compare it to a traditional QA process:

| QA Lifecycle Stage      | Coverage |
| ----------------------- | -------- |
| Test Analysis           | ✅       |
| Test Planning           | ✅       |
| Test Design             | ✅       |
| Test Data Strategy      | ✅       |
| Test Generation         | ✅       |
| Test Execution          | ✅       |
| Defect Detection        | ✅       |
| Failure Analysis        | ✅       |
| Reporting               | ✅       |
| Learning & Optimization | ✅       |
| Continuous Improvement  | ✅       |

It covers the QA automation lifecycle stages listed above. It deliberately does
**not** attempt a coverage percentage — a figure like that is a claim about work
that was never measured, and this framework's own governance forbids claiming
coverage it did not execute. What a run actually covered is reported per run, in
the Exploration Disclosure and the coverage report.

Out of scope by design: security assessment (a separate skill), accessibility,
performance, responsive and cross-browser testing. Deferred to future phases:
enterprise ALM integrations (Jira/Azure DevOps), mobile testing, visual AI
testing, and large-scale distributed execution.

---

# One-Sentence Executive Summary

> **This framework is the backend methodology for the RedOps Site Explorer: an
> AI-powered QA engineering platform that explores an authorized web application
> within a bounded budget, builds an evidence-backed model of what exists, plans
> risk-based QA testing, generates enterprise-grade Playwright automation,
> executes it with retry and self-healing, collects masked and provenanced
> evidence, analyzes failures, reports honestly what was and was not covered,
> learns from every execution, and maintains deterministic and auditable
> behaviour — while drawing a hard line at security interpretation, which belongs
> to a separately selected security skill.**

That's the description to use with senior engineers or architects — it reflects
the architecture accurately, including its boundary, without overstating
capabilities.
