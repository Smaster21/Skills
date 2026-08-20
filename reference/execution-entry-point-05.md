<!-- PRESERVED VERBATIM (part 5 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
# 5. Phase Dispatch

16 phases, per `01` §16. Read the owning document **before** entering its phase.

### Phase 0 — CONFIGURING *(required)*
**Owner:** Configuration Layer · `01` §29, §7
Resolve configuration by priority — CLI → env → project → default. Register
capabilities. Assign Execution ID and Correlation ID. Initialize the **Framework
Cache Service** (`01` §39 — available from here onward, not from `11`'s phase).
**Playbook:** §1 scaffold · §2 config · §18 health pre-flight
**Gate:** configuration valid; capabilities registered.

### Phase 0A — DETECTING_CHANGES *(optional — 13)*
Load previous snapshot → fingerprint → verify stability (`13` §26) → compare
live → emit **Discovery Delta** to `04`, **Regeneration Request** to `06`.
**Emit:** snapshot, Discovery Delta · **Gate:** `13` §24
**Degrade:** no snapshot, unstable fingerprints, any failure → full discovery.
**Never:** mutate/version/diff the graph (`04`) · determine regeneration scope
(`06`) · implement cache storage (`11`) · calculate risk (`02`).

### Phase 1 — DISCOVERING *(required)*
**Owner:** `03`
Crawl, observe runtime, classify components, detect flows, validate, persist a
versioned snapshot. **Playbook:** §3 · **Gate:** `03` §35
**Constraint:** observational only (`03` §36). Never delete data, create
accounts, submit payments, bypass authentication, or store unmasked secrets.

### Phase 2 — BUILDING_GRAPH *(required)*
**Owner:** `04`
Normalize, deduplicate, resolve entities, create nodes and relationships with
evidence, verify integrity, version, diff. **Gate:** `04` §30
**Constraint:** an invalid graph SHALL NEVER be persisted.

### Phase 2A — INTELLIGENCE *(optional — 11)*
Project `04` into the reuse-optimized package; serve unchanged work from cache.
**Emit:** `unified-intelligence.json` stamped with **source graph version**
**Gate:** `11` §31 · **Degrade:** any failure → consume `04` directly.
**Constraint:** derived, **never canonical**. Where it disagrees with `04`, `04`
is correct.

### Phase 3 — Locator Verification *(required)*
**Owner:** `03` §29 → `06` §26
Probe every locator candidate; keep only those that resolve **and** return the
expected count. **Playbook:** §4 and the first two Hard-Won Rules in §15
**Gate:** no unverified locator reaches generation.

### Phase 4 — PLANNING *(required)*
**Owner:** `05`
Coverage analysis → workflow selection → risk prioritization (risk consumed from
`02`) → execution strategy → dependencies → resources → schedule → artifacts →
validate → approve. **Emit:** `test-plan.json` *(canonical)* + supporting plans
**Gate:** `01` §15 Intelligence Layer gate. Plan validation FAIL prevents
generation (`05` §32).
**Constraint:** the plan SHALL be executable with **no optimizer present**
(`05` §23A).

### Phase 4A — PLANNING_REVIEW *(optional — 15)*
Review coverage sufficiency (from `05`), effort-vs-risk proportionality (risk
from `02`), planned-strategy quality, cost, cross-engine consistency. Score and
submit **Planning Review** to `02`.
**Gate:** `15` §23 · iteration cap `15` §28A · **Degrade:** proceed unreviewed, flagged.
**Constraint:** **advisory only.** Never approves, rejects, or blocks. Never
calculates risk or confidence, computes coverage, performs plan validation, or
assesses generated artifacts — none exist yet.

### Phase 5 — GENERATING *(required)*
**Owner:** `06`
Select templates, expand workflows, generate Page Objects, components, fixtures,
utilities, tests, assertions; resolve locators and dependencies; assemble; stamp
traceability (`06` §40). **Playbook:** §5, §6, §7
**Constraint:** prohibited before plan approval. Direct HTML-to-test generation
prohibited.

### Phase 6 — VALIDATING *(required)*
**Owner:** `01` §18 · `06` §38
TypeScript compilation, Playwright syntax, imports, duplicates, circular
dependencies, missing assertions, invalid selectors, hardcoded waits, unsafe XPath.
**Gate:** WARNING may proceed. **FAIL stops** — poor automation never executes.

### Phase 6A — OPTIMIZATION *(optional — 12)*
Consume plan, `05`'s dependency plan and parallel candidacy, historical runtime.
Produce batch plan, runtime prediction, **Optimization Proposal**.
**Gate:** `12` §32 · **Degrade:** absent/failed/rejected → plan's own ordering.
**Constraint:** a **constraint-bounded permutation** — MAY reorder, batch, hint;
SHALL NEVER add, remove, or re-scope a test. Never schedules, allocates browsers
or workers, or sets timeouts.

### Phase 7 — EXECUTING *(required)*
**Owner:** `07` — **sole runtime authority**
**Validate the proposal first** (`07` §30A): reject unless its test set is
**exactly** the plan's test set. Rejected or absent → plan ordering. Log; never halt.
Then: build context, prepare environment, allocate resources, launch browsers,
isolate contexts, initialize fixtures, schedule workers, execute, capture.
**Emit:** `runtime-schedule.json` *(canonical)*, execution history, artifacts,
deviation record · **Gate:** `07` §40 · **Playbook:** §11, §12, §15, §17
**Constraint:** executes only — does not decide, retry autonomously, or heal.
No artifact owned elsewhere overrides it.

#### Deterministic Checkpoints within Phase 7 *(optional — 14)*
At **batch or workflow-group boundaries only** — never per test completion, which
is non-deterministic under parallel execution. Consume coverage from `05` and
confidence from `02`; rank gaps; score value; prioritize `06`'s redundancy
findings; submit stop-or-continue **recommendation** to `02`.
**Degrade:** absent, or no history on a first run → full plan executes (`14` §30).
**Constraint:** never stops execution, skips a test, calculates confidence, or
computes coverage. **Every skipped test and early stop SHALL be disclosed to `09`**
(`14` §29) — a skipped test is never passed and never coverage; an early-stopped
run is never complete.

### Phase 8 — RETRYING / SELF_HEALING *(optional — 08, under 02)*
Classify the failure first (`08` §16). Then analyse DOM, generate ranked
candidates, score, apply policy, request **02 approval**, apply, verify, audit.
**Playbook:** §8 mechanics only · **Limits:** `08` §32 — 3 healing attempts,
2 locator replacements, 1 wait adaptation, 1 browser restart.
**Retry ownership:** `05` plans · `02` approves · `07` enforces.
**Constraint:** score below 60 → reject. Never heal failed assertions, business
logic, backend defects, API failures, auth failures, security vulnerabilities, or
data corruption (`08` §13). A healed locator is a candidate; promotion needs
repeated success plus approval.

### Phase 9 — COLLECTING_ARTIFACTS / DIAGNOSTICS *(required)*
**Owner:** `01` §22, §23
Collect the planned artifact set, link each to Execution/Test/Workflow/Correlation
ID, then convert failures into root-cause findings with category, severity,
confidence *(from `02`)*, evidence, suggested fix, historical occurrence.
**Playbook:** §12 · **Constraint:** mask secrets before persistence.

### Phase 10 — REPORTING *(required)*
**Owner:** `09`
Collect, validate, normalize, aggregate, compute metrics, generate analytics,
render every format from one normalized model, archive, publish.
**Playbook:** §10, §13, §16 · **Gate:** `09` §33
**Constraint:** never fabricate, suppress, or alter a result. Disclose skipped
scope, degraded engines, and unexecuted work.

### Phase 11 — LEARNING *(optional — 10)*
Collect, validate, normalize, analyse, detect patterns, generate recommendations
with confidence *(from `02`)*, persist as immutable versions, publish.
**Constraint:** recommends only. Never modifies tests, Page Objects, or
configuration. `02` decides whether a recommendation applies.

### Throughout — Decision Audit
**Owner:** `02` §38 · **Emit:** `decision-history/decision-{NNNN}.json` —
inputs, evidence, candidates, rejected alternatives, winner, confidence, audit.
**Constraint:** persistence failure SHALL NEVER silently continue.

---

# 5A. Evidence, Evidence Quality, And Framework Confidence

This chain is **operationally mandatory**. A run that produces tests and reports
but not this chain has not satisfied the framework.

```
Real execution observation
        ↓
Evidence Object                  02 §18   (id · type · source · confidence · timestamp · payload)
        ↓
Per-item evidence confidence     PLAYBOOK §19
        ↓
Evidence Quality                 PLAYBOOK §19 → 02 §21's Evidence Quality term
        ↓
02 §21 Confidence Engine         + Historical Reliability · Rule Agreement · Environment Stability
        ↓
Framework Confidence             02 §21 — the only value reportable as "confidence"
        ↓
Decision · execution · report artifacts
```

## Evidence Objects

- Evidence Objects are produced from **real execution observations** — a live DOM
  read, a probe result, a measured latency, a persisted artifact.
- Evidence SHALL NEVER be fabricated, manually manufactured, or back-filled to
  satisfy a criterion (`01` §31).
- Specification text is **not** runtime evidence and SHALL NEVER be converted
  into it (§7A).
- Every applicable Evidence Object carries a **measured** `confidence`. A
  hand-authored confidence value is a defect, not evidence.
- Evidence with an empty required field is still evidence — it scores lower. It
  is never silently completed.

---

<!-- nav -->
*← [part 4](execution-entry-point-04.md)  ·  part 5 of 7  ·  [part 6](execution-entry-point-06.md) →*  ·  [reference index](INDEX.md)
