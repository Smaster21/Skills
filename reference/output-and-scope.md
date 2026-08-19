# Output Discipline, Environment Contract, and Scope Enforcement

> Executor reference. How this skill maps onto the coordination system's
> `$OUTPUT_DIR` discipline, which environment variables it reads, and the
> mandatory scope check. QA is **defensive** — it writes a QA phase directory,
> not `findings/finding-NNN/`.

## Output root — `./output/qa/`

A Site Explorer run writes **only** under `./output/qa/`. It never writes to
`output/findings/`, `output/security/`, or `output/redops/`, and never creates a
security-style output directory.

**Run isolation** is at the `./output/` boundary: one output root per run, so
`qa/` inside it is flat and deterministic. A run never writes into another run's
output root (`Rule 9`).

```
./output/qa/
├── README.md                     plain-language guide to this directory
├── run.json                      execution metadata only (never credentials)
├── summary.json                  compact machine-readable result
│
├── report/
│   ├── final-report.md           the human deliverable
│   └── final-report.json         structured equivalent, stable for API/UI
│
├── discovery/                    WHAT THE APPLICATION CONTAINS
│   ├── discovery-summary.json    pages.json          routes.json
│   ├── application-map.json      forms.json          inputs.json
│   ├── components.json           states.json
│
├── network/                      DERIVED VIEW over the AIC — never canonical
│   ├── network-summary.json      api-inventory.json  endpoints.json
│   ├── requests.json             responses.json
│   │   (presentation of AIC `apis`/`api-calls`; reuses AIC ids; deleting this
│   │    directory must lose nothing the AIC does not still hold)
│
├── knowledge/                    WHAT WAS LEARNED
│   ├── application-model.json    page-model.json
│   ├── workflow-model.json       interaction-model.json
│
├── planning/
│   ├── exploration-plan.json     workflow-plan.json  test-strategy.json
│
├── tests/
│   ├── catalogue.json            the traceability spine
│   ├── generated/*.spec.ts       pages/*.ts          fixtures/*.ts
│
├── execution/
│   ├── execution-summary.json    test-results.json
│   ├── failures.json             retries.json
│
├── evidence/
│   ├── evidence-index.json
│   ├── tests/TC-…/               per-test evidence (only files that exist)
│   └── screenshots/  traces/  videos/  network/  raw/
│
├── coverage/                     APPLICATION EXPLORATION COVERAGE
│   ├── coverage-summary.json     route-coverage.json
│   ├── workflow-coverage.json    test-coverage.json
│
├── diagnostics/
│   ├── failures.json             locator-healing.json
│   ├── retry-history.json        blocked-actions.json
│
└── raw/                          low-level streams
    ├── agent-events.jsonl        decision-history.jsonl
    └── execution-events.jsonl
```

### Artifact categories — never mixed

`A` run metadata · `B` discovery · `C` network/API · `D` application knowledge ·
`E` planning · `F` test generation · `G` execution · `H` evidence · `I` coverage ·
`J` diagnostics · `K` final report · `L` raw/debug.

A screenshot is evidence. A generated spec is `tests/generated/`. A route list is
discovery. An endpoint inventory is network. A retry history is diagnostics. The
report is `report/`. One artifact, one category.

### Stable identifiers — mandatory

Identity is an assigned ID, **never** a filename or URL:

`PAGE-001` · `ROUTE-001` · `API-001` · `WF-001` · `TC-001` · `EVID-001`

> **Reuse before minting.** Where an AIC record already carries an identifier
> (`sha256[0:16]` of canonical inputs), that id is authoritative and is reused
> verbatim. Readable aliases above are a **1:1 display mapping** that always
> carries the AIC id alongside — never a second API identity system.

IDs are deterministic for identical input (`PLAYBOOK` §19) and are the join keys
for the traceability chain:

```
PAGE → WORKFLOW → TEST CASE → GENERATED SPEC → EXECUTION RESULT → EVIDENCE
PAGE → OBSERVED API → WORKFLOW → TEST CASE
```

Every generated spec maps back through `tests/catalogue.json` to its test case and
its source workflow. Traceability is **mandatory**, not best-effort.

### Provenance — W7-B's vocabulary, presented

Provenance and state come from the **existing** W7-B fields; the report presents
them, it does not define a parallel scheme:

- `provenance.discoverySource` ∈ `crawl · js · sitemap · robots · network · verification`
- state ladder `DISCOVERED → OBSERVED → EXERCISED → VALIDATED` — **never collapsed**,
  and `VALIDATED` is never emitted by QA

| Report wording | Backed by |
|---|---|
| observed | state `OBSERVED` (`discoverySource: network`) |
| discovered | state `DISCOVERED` (`js` / `sitemap` / `robots` / `crawl`) |
| inferred | a derived association, explicitly labelled |
| generated | produced by the framework (a test, a plan) |

**Never present inferred, discovered, or generated material as observed** — a
string found in JavaScript is not proof the endpoint was called. Same discipline
`Rule 4` applies to evidence, extended to the model.
Full contract: `w7-api-evidence-contract.md`.

### This is a QA report, not a security assessment

The output is limited to QA and application-exploration content. Vulnerability
findings, CVSS, exploitability, attack priority, security coverage, security
recommendations, and any security decision layer all fall outside it. Coverage is **Application Exploration Coverage** /
**QA Coverage** — never "security coverage". A failure is a QA failure category
(`locator_failure`, `assertion_failure`, `timeout`, `navigation_failure`,
`authentication_failure`, `network_failure`, `environment_failure`,
`test_data_failure`, `application_behavior_failure`, `blocked`, `unknown`) — never
a vulnerability.

Security assessment is owned separately by the Skillmatrix security skills, which
may consume this evidence later. This skill emits none of it.

### Sensitive data — one masking authority, already implemented

Masking runs **before any persistence through a single existing authority**
(W7-B §B15, `01` §30, `07` §41); reprs `plain · masked · redacted · omitted ·
hashed · binary`, with unknown sensitivity defaulting to `masked`.

**Do not implement a second masking system.** The output layer consumes
already-masked evidence and never weakens, bypasses, or re-derives it. No
password, token, bearer/authorization header, cookie, session secret, API key or
private key reaches any artifact.

Absence uses W7-B §C7 vocabulary — `EMPTY · UNAVAILABLE · NOT_PRODUCED ·
BLOCKED · NOT_EXERCISED · NOT_OBSERVED` — never a bare `null`, `0`, `false` or
`[]` carrying meaning.

### Determinism and backward compatibility

Stable names only — never `report-new.json`, `final-final-v2.json`, or
`output123.json`. Identical input yields identical artifact names and locations.

Before renaming or removing an existing artifact, identify its consumers
(result ingestion, runner logic, tests). Where a consumer exists, provide a
deterministic adapter or a compatibility period. **Never silently break an
existing artifact contract.**

Environment variables, the scope check, cleanup and determinism:
**`scope-enforcement.md`**.
