---
name: qa-automation
description: Autonomous web-app QA — discover, plan, generate, run Playwright tests
---

# QA Automation

Autonomous, evidence-driven QA engineering for web applications. Given an
authorized URL it understands the app, plans and generates a Playwright +
TypeScript suite from verified locators, executes it, diagnoses failures,
records every decision with measured confidence, and reports honestly what was
and was not covered.

> **This `SKILL.md` is a ROUTER.** It tells the coordinator *when to mount this
> skill and which reference file to hand an executor*. It is not the operational
> manual — the executor-facing playbooks live in `reference/`, and the
> authoritative specification lives in `docs/`. See [Reference](#reference).

---

## Purpose

- **Problem solved:** turns an authorized web application into a running,
  self-healing, production-grade automated test suite with traceable evidence —
  without a human hand-writing locators, plans, or assertions.
- **Engagement phase:** **QA / verification** — functional, forms, auth,
  navigation, accessibility (axe-core), and **Tier-1 passive** security
  observation (headers, cookie flags, OWASP mapping). It is a **defensive** skill,
  not an exploitation skill.
- **Target type:** web applications (any framework), public surface and, when
  credentials are supplied, authenticated surface.
- **When the coordinator invokes it:** "verify / regression-test / build an E2E
  or Playwright suite for an in-scope web app", or as a defensive counterpart
  after recon has mapped a web surface. Mount for verification work — **not** to
  produce exploit findings.

**Not** a Playwright script generator, a Selenium wrapper, or a raw
HTML-to-test converter. A run that emits scripts without discovery, decisions,
measured evidence, and disclosed scope has not done the job.

---

## Phases

Sixteen phases over a 21-state machine. This table is the **routing map**; the
authoritative specification for each phase is its owning document under `docs/`.
Read the owning document before entering a phase.

| # | Phase | Owner | Routing note |
|---|---|---|---|
| 0 | CONFIGURING | Config · `01` §29 | Resolve config, assign Execution + Correlation ID, health pre-flight (`PLAYBOOK` §18) |
| 0A | DETECTING_CHANGES *(opt)* | `13` | Skip → full discovery if no prior snapshot |
| 1 | DISCOVERING | `03` | Crawl, observe runtime, classify components — observational only |
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

Reports MUST disclose skipped scope, degraded engines, and unexecuted work. A
capped, single-browser, or credential-limited run is NEVER presented as complete.

---

## Tools

- `node` (≥20), `npm`, `npx playwright`
- `chromium` (Playwright-managed build; **no** `install-deps`, **no** `sudo`)
- `axe-core` (accessibility)
- `jq` (JSON handling in reference playbooks)
- *Optional:* OWASP `zap` daemon — Tier-2 passive proxy only; absent → Tier-1
  passive observation still runs

**Installation:** `npm i -D @playwright/test axe-core typescript` then
`npx playwright install chromium` (Chromium only). Standard Kali otherwise.

---

## Related Skills

- `/reconnaissance` — run first; its surface map seeds discovery targets
- `/techstack-identification` — fingerprint the app before planning coverage
- `/api-security` — pair for API surfaces discovered during the crawl
- `/authentication` — when login/session testing is in scope
- `/regression-sweep` — downstream: re-validate this suite's results on a schedule

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
7. **Chromium-only unless the browser matrix authorizes more.** A
   declared-but-unrun browser project misrepresents scope.
8. **Report honestly.** Reconcile every number against the runner's output;
   separate target defects from suite defects from environment artifacts; never
   present a partial run as complete.
9. **Preserve prior artifacts.** A previous execution's output is immutable
   evidence — never modify, overwrite, or append to it.
10. **Non-interactive.** All context via env vars; sensible defaults; clean up
    processes and temp files on exit; deterministic output for identical input.

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

**Not applicable to this skill** (attack-finding constructs; QA is defensive):
`reference/scenarios/` exploit recipes · `PATT` payload URLs ·
`findings/finding-NNN/` + CVSS · the skeptic/validator finding-verification loop.
QA produces functional results and Tier-1 passive observations, not exploit
findings; its evidence lives in `qa/decision-history/` and `qa/reports/`.
