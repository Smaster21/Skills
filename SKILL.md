---
name: qa-automation
description: Site Explorer backend — explore a web app, plan, generate, run Playwright tests
---

# QA Automation

**The backend methodology for the RedOps Site Explorer.** Given an authorized,
in-scope URL it explores the application, builds an evidence-backed
understanding of what exists, plans and generates a Playwright + TypeScript QA
suite from verified locators, executes it, diagnoses failures, records every
decision with measured confidence, and reports honestly what was and was not
covered.

```
URL → scope/authorization validation → browser-based exploration
    → application understanding → evidence/provenance collection
    → QA test planning → QA test generation → Playwright validation
    → Playwright execution → retry/diagnostics/self-healing
    → Site Explorer result
```

**Security assessment is not part of this skill.** It answers *"what exists in
the application, and what evidence do we have?"* — never *"what security tests
should be performed here?"* A separately selected Skillmatrix security skill
reads this skill's evidence and determines its own methodology (§Rules 11–14,
`docs/01` §2.1).

> **This `SKILL.md` is a ROUTER.** It tells the coordinator *when to mount this
> skill and which reference file to hand an executor*. It is not the operational
> manual — the executor-facing playbooks live in `reference/`, and the
> authoritative specification lives in `docs/`. See [Reference](#reference).

---

## Purpose

- **Problem solved:** turns an authorized web application into a running,
  self-healing, production-grade automated test suite with traceable evidence —
  without a human hand-writing locators, plans, or assertions.
- **Engagement phase:** **exploration / QA verification** — smoke, functional,
  UI, forms, authentication, navigation and API, plus optional dashboard, table
  and visual. It is an **observational** skill: it discovers and verifies, it
  never assesses security.
- **Target type:** web applications (any framework), in **Unauthenticated
  Exploration** or, when credentials are supplied, **Authenticated Exploration**.
  Scope and authorization are mandatory in both modes.
- **When the coordinator invokes it:** "explore / map / verify / regression-test
  / build an E2E or Playwright suite for an in-scope web app", or to produce the
  evidence base a later security skill will consume. Mount for exploration and
  verification — **never** to produce security findings or to decide which
  security skill applies.

**Not** a Playwright script generator, a Selenium wrapper, a raw HTML-to-test
converter, a security scanner, or a security-skill recommender. A run that emits
scripts without discovery, decisions, measured evidence, and disclosed scope has
not done the job.

## Exploration guarantee

Never claim that every page of an arbitrary site was explored, and never state a
100% figure. The guarantee is **all reachable, in-scope surfaces discovered
within the configured exploration budget** — via browser navigation, discovered
links and routes, sitemap, robots, JavaScript routes, GraphQL indicators,
network/API observation, forms and interaction discovery, and the supplied
authenticated session where present.

Every discovered-but-not-reached surface carries exactly one recorded reason:
`inaccessible · blocked · capped · excluded · unavailable state · unavailable
credentials` (`docs/01` §2.2, `09` §12A).

## QA test categories

| # | Category | Default | # | Category | Default |
|---|---|---|---|---|---|
| 1 | Smoke | on | 6 | Navigation | on |
| 2 | Functional | on | 7 | API | on |
| 3 | UI | on | 8 | Dashboard | **off** |
| 4 | Forms | on | 9 | Table | **off** |
| 5 | Authentication | on | 10 | Visual | **off** |

Functional covers positive · boundary · business-rule · **CRUD except Delete** ·
state-transition. Delete affordances are discovered, never exercised; a test
still cleans up its **own** synthetic data (`PLAYBOOK` §5–§6, `16` §44).

**Not generated:** accessibility/WCAG · performance · responsive · cross-browser
· any security test (SQLi, XSS, SSRF, IDOR, command injection) · security
findings · severity/CVSS · security-skill applicability or recommendation.

---

## Phases

Sixteen phases over a 21-state machine. This table is the **routing map**; the
authoritative specification for each phase is its owning document under `docs/`.
Read the owning document before entering a phase.

| # | Phase | Owner | Routing note |
|---|---|---|---|
| 0 | CONFIGURING | Config · `01` §29 | Resolve config, assign Execution + Correlation ID, health pre-flight (`PLAYBOOK` §18) |
| 0A | DETECTING_CHANGES *(opt)* | `13` | Skip → full discovery if no prior snapshot |
| 1 | DISCOVERING | `03` | Crawl + robots, sitemap, JS routes, GraphQL, hidden endpoints, route templates, forms, params, auth surfaces, network/API capture; deterministic dedup, masking before persistence — observational only (`PLAYBOOK` §21) |
| 2 | BUILDING_GRAPH | `04` | Canonical application model; invalid graph never persisted |
| 2A | INTELLIGENCE *(opt)* | `11` | Skip → consume `04` directly |
| 3 | Locator Verification | `03` §29 → `06` §26 | Probe every candidate; no unverified locator reaches generation |
| 4 | PLANNING | `05` | Coverage, workflows, risk (from `02`), browser matrix |
| 4A | PLANNING_REVIEW *(opt)* | `15` | Advisory; skip → proceed unreviewed, flagged |
| 5 | GENERATING | `06` | Page Objects, fixtures, specs from verified locators |
| 6 | VALIDATING | `01` §18 · `06` §38 | TypeScript + suite gate; FAIL stops |
| 6A | OPTIMIZATION *(opt)* | `12` | Skip → plan ordering |
| 7 | EXECUTING | `07` | Sole runtime authority; browsers, workers, timeouts |
| 8 | RETRYING / SELF_HEALING *(opt)* | `08` under `02` | Classify first; healing only for locator drift |
| 9 | COLLECTING_ARTIFACTS / DIAGNOSTICS | `01` §22–23 | Failures → root-cause findings |
| 10 | REPORTING | `09` | One normalized model → every format; disclose gaps |
| 11 | LEARNING *(opt)* | `10` | Recommends only; `02` decides application |

**Evidence chain (operationally mandatory).** Execution → Evidence Objects
(`02` §18) → per-item confidence (`PLAYBOOK` §19) → Evidence Quality → `02` §21
Framework Confidence → decision/report artifacts. Framework Confidence has one
authority — `02` §21 — with fixed terms Evidence Quality 0.35 / Historical
Reliability 0.25 / Rule Agreement 0.20 / Environment Stability 0.20
(`PLAYBOOK` §19, §20.1–§20.3). An unavailable term is a **named state**, never a
substituted number; Framework Confidence computes only when all four are
available, else refuses and names the gaps. Full contract:
`reference/evidence-and-confidence.md`.

**Scope & false-positive handling.** Authorized target only; validate against
`SCOPE_FILE` before any navigation (see Rules and `reference/output-and-scope.md`).
Target defects, suite defects, and environment artifacts are classified and
reported separately (`08` §16, `PLAYBOOK` §18).

---

## Output

Writes into a **QA phase subdirectory of `$OUTPUT_DIR`** — never the root. The
internal tree is the framework's canonical artifact set (`docs/` owners in
parentheses). Full mapping and env contract: `reference/output-and-scope.md`.

| Path | Contents | Owner |
|---|---|---|
| `qa/discovery/` | discovery report, component + verified inventory | `03` |
| `qa/knowledge/` | knowledge graph, graph diff | `04` |
| `qa/planning/` | test plan, runtime schedule | `05` / `07` |
| `qa/tests/`, `qa/pages/`, `qa/fixtures/` | generated Playwright suite | `06` |
| `qa/decision-history/` | decision + evidence + confidence records | `02` §36/§38/§39 |
| `qa/execution-history/` | execution history, deviation record | `07` §39 |
| `qa/reports/` | HTML / JSON / JUnit, diagnostics, `rw-report.md` | `09` |
| `qa/raw/` | traces, screenshots, videos | `07` |
| `qa/aic/` | AIC serialization: `pages · routes · apis · api-calls · forms · parameters · auth-surfaces · robots · javascript-routes · relationships` | `04` (`PLAYBOOK` §21) |
| `qa/aic/evidence/` | masked request/response capture, screenshot sidecars | `04` / `07` §41 |

Reports MUST disclose skipped scope, degraded engines, and unexecuted work, and
MUST include the mandatory **Exploration Disclosure** (`09` §12A). A capped or
credential-limited run is NEVER presented as complete.

`qa/aic/attack-surface` is permanently **`NOT_PRODUCED`** — this skill emits no
attack-surface projection, ever (Ownership Matrix, W8 / C1).

---

## Tools

- `node` (≥20), `npm`, `npx playwright`
- `chromium` (Playwright-managed build; **no** `install-deps`, **no** `sudo`)
- `jq` (JSON handling in reference playbooks)

**Installation:** `npm i -D @playwright/test typescript @types/node` then
`npx playwright install chromium` (Chromium only). Standard Kali otherwise.

No accessibility, performance, or security tooling — `axe-core`, Lighthouse and
ZAP were removed in W8 along with the capabilities that used them.

---

## Related Skills

**Upstream**

- `/reconnaissance` — run first; its surface map seeds exploration targets
- `/techstack-identification` — fingerprint the app before planning coverage

**Downstream — consumers of this skill's evidence, not QA activities**

`/api-security` · `/authentication` · `/client-side` · `/server-side` ·
`/injection` · `/web-app-logic`

A security skill is selected **by the user**, mounted by RedOps, and reads the
Site Explorer evidence to determine its own testing targets and methodology.
This skill SHALL NEVER select, suggest, rank, or score one of them.

**Also downstream**

- `/regression-sweep` — re-validate this suite's results on a schedule

---

## Rules

1. **Authorized target only.** Missing or ambiguous authorization → **STOP,
   report `BLOCKED`**. Never discover, invent, or substitute a target; never
   carry authorization forward from a prior run.
2. **Scope before action.** Validate every target against `SCOPE_FILE` (if
   provided) before any navigation; abort out-of-scope with a clear message and
   no output (pattern in `reference/output-and-scope.md`).
3. **Observational discovery.** Never delete data, create accounts, submit
   payments, bypass authentication, or store unmasked secrets.
4. **Evidence is observed, never authored.** Evidence Objects come from real
   execution; each applicable one carries a *measured* confidence. Never
   fabricate evidence or convert specification into runtime evidence.
5. **One confidence model.** `02` §21 is the sole Framework Confidence authority;
   weights are fixed and unchanged. No second confidence or Evidence Quality
   model.
6. **Unavailable is a state, not a number.** Never substitute `0`, `50`, `100`,
   a default, a previous value, or an inference for an unavailable measurement.
7. **Chromium only.** Cross-browser testing is not this skill's responsibility.
   A declared-but-unrun browser project misrepresents scope.
8. **Report honestly.** Reconcile every number against the runner's output;
   separate target defects from suite defects from environment artifacts; never
   present a partial run as complete.
9. **Preserve prior artifacts.** A previous execution's output is immutable
   evidence — never modify, overwrite, or append to it.
10. **Non-interactive.** All context via env vars; sensible defaults; clean up
    processes and temp files on exit; deterministic output for identical input.
11. **No security interpretation.** Record factual observations only — an input
    exists, a form exists, a route exists, an API endpoint exists, a JavaScript
    or GraphQL route was discovered, an authentication surface exists, a
    parameter exists, a request/response was observed. NEVER conclude that an
    attack class applies, that a surface is vulnerable, that a security skill
    should be run, or that any of it is probable. NEVER produce a severity, a
    CVSS score, a finding, or a filtered security test plan.
12. **Reserved states are never emitted.** The lifecycle ladder is
    `DISCOVERED → OBSERVED → EXERCISED → VALIDATED`. This skill SHALL NEVER emit
    `OFFENSIVELY_VALIDATED` or `VULNERABILITY_CONFIRMED` — they belong to the
    security skill and validator downstream. A failed QA test is a target
    defect, suite defect, or environment artifact — never a security finding.
13. **Bounded exploration.** Never claim 100% or "every page". Every
    discovered-but-not-reached surface carries exactly one recorded reason.
14. **Deterministic identity, no re-crawl.** Every page/route/API/action has a
    deterministic identity. A surface is revisited only for a recorded reason —
    distinct state, distinct authentication context, workflow transition,
    validation, or explicitly authorized re-discovery.
15. **Authentication mode is not authorization.** Both **Authenticated** and
    **Unauthenticated Exploration** require approved scope and RoE. Absence of
    credentials narrows scope; it never stops a run and never permits a bypass.

---

## Reference

Executor-facing playbooks and deep specification. The coordinator passes one or
two `reference/` files to an executor via `SKILL_FILES`; it never passes this
`SKILL.md`.

- `reference/quickstart.md` — **executor-ready minimal playbook**: scope check →
  discover → plan → generate → execute → report, with exact commands
- `reference/evidence-and-confidence.md` — the Evidence → Evidence Quality →
  Framework Confidence chain, routing into `docs/` §19/§20/§21 (no formula copied)
- `reference/output-and-scope.md` — `$OUTPUT_DIR/qa/…` mapping, env-var contract,
  and the scope-enforcement pattern
- `reference/execution-entry-point.md` — the full operational contract (WHEN,
  lifecycle, governance), preserved verbatim; deep reference for reviewers
- `reference/INDEX.md` — navigator for this reference set
- `docs/` — the 18-document authoritative specification (`01`–`16`, Ownership
  Matrix, `IMPLEMENTATION_PLAYBOOK`). Authority for every routing note above.

**Not applicable to this skill.** Attack-finding constructs:
`reference/scenarios/` exploit recipes · `PATT` payload URLs ·
`findings/finding-NNN/` + CVSS · the skeptic/validator finding-verification loop.

Removed in **W8** and not to be reintroduced: accessibility/WCAG testing ·
performance testing · responsive testing · cross-browser testing · security
testing · passive security scanning · offensive testing · CVSS/severity/finding
generation · Skill Applicability Tagging · security-skill recommendation ·
security probability or confidence · any Test Catalogue used as a recommendation
or decision engine.

The **Test Catalogue is retained** (`06` §22.1, §31) as the internal test-identity
and execution/audit record — test ID, category, target, execution status,
evidence reference, failure info, retry info, timestamp. It records facts, never
applicability.

This skill produces exploration evidence and QA results, which live in
`qa/aic/`, `qa/decision-history/` and `qa/reports/`.
