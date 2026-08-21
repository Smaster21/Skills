# qa-automation — Reference Navigator

Navigator and **loading manifest** for this skill's reference set. `SKILL.md` is
never passed to an executor.

## Loading manifest — which files an executor gets, and when

A file is required **for a phase**, not in the abstract. Load Tier 0 once, then
the Tier 1 row for the phase you are entering. This keeps an executor at roughly
400–600 lines of instruction at any moment instead of the full ~3,200.

**Tier 0 — always loaded, for the whole run (~230 lines)**

| File | Why |
|---|---|
| [`quickstart.md`](quickstart.md) | the step sequence and exact commands |
| [`scope-enforcement.md`](scope-enforcement.md) | env contract + the scope check that precedes any navigation |

**Tier 1 — loaded for the current phase only**

| Phase | Load | ~Lines |
|---|---|---:|
| 0 CONFIGURING | `rules-extended.md` | 67 |
| 0B PROFILING · 1 DISCOVERING | `discovery-profiling.md` + `site-agnostic-discovery.md` + `workflow-flows.md` | 455 |
| 2 BUILDING_GRAPH · 3 Locator Verification | `locator-ladder.md` (+ `site-agnostic-discovery.md`, already loaded) | 70 |
| 4 PLANNING | `coverage-ledger.md` + `planning-coverage-controls.md` + `write-operations-and-test-data.md` + `workflow-flows.md` | 560 |
| 5 GENERATING · 6 VALIDATING | `suite-self-validation.md` + `suite-gates-extended.md` | 285 |
| 7 EXECUTING · 9 DIAGNOSTICS | `known-failure-modes.md` + `suite-gates-extended.md` (Gate C) | 290 |
| 10 REPORTING | `final-report-md.md` + `final-report-json.md` + `output-and-scope.md` + `accessibility-observations.md` | 435 |

**Tier 2 — on demand only**

| File | Load when |
|---|---|
| [`w7-api-evidence-contract.md`](w7-api-evidence-contract.md) | touching API/network output or the AIC |
| [`evidence-and-confidence.md`](evidence-and-confidence.md) | producing or auditing the confidence chain |
| [`execution-lifecycle.md`](execution-lifecycle.md) | a phase's behaviour is unclear |
| `execution-entry-point-01..07.md` | a reviewer needs the full historical contract |

A "Mandatory" label inside a reference file means **mandatory for the phase(s)
that load it**, per the table above — it does not mean every run loads every file.

## Full file list

| File | Give to an executor when… | Contents |
|------|---------------------------|----------|
| [`quickstart.md`](quickstart.md) | Running the skill end to end | Minimal executor playbook: scope → profile → discover → verify → plan → generate → execute → report |
| [`discovery-profiling.md`](discovery-profiling.md) | Profiling a target and crawling it | Phase 0B profiling, then the depth-first exhaustive read-only crawl |
| [`rules-extended.md`](rules-extended.md) | Rules 11–25 | Target-agnostic locators, coverage ledger, intent binding, workflow forms, auth isolation, measured responses, evidence-backed classification, session resilience, per-route baselines, write gating, QA-only output, stable IDs, provenance, healing honesty, immutability |
| [`execution-lifecycle.md`](execution-lifecycle.md) | Understanding phase behaviour | Read-only exhaustive discovery, no-caps + ledger contract, target-agnostic construction, suite-soundness gates, evidence chain, scope handling |
| [`site-agnostic-discovery.md`](site-agnostic-discovery.md) | Building or reviewing discovery or interaction | Site profiling, standards-based labels, behavioural widget detection, depth-first traversal, read-only boundary |
| [`locator-ladder.md`](locator-ladder.md) | Phase 3 verification | Seven-rung ladder incl. label-scoped and container-scoped, single-match gate, intent rule |
| [`suite-gates-extended.md`](suite-gates-extended.md) | Phases 6, 7, 9 | Gates A2/A3 (plan vs emitted source), B2 (sensitivity), C-replay |
| [`planning-coverage-controls.md`](planning-coverage-controls.md) | Phase 4 / 4A | Route sampling, plan-review tripwires, derived test tags |
| [`accessibility-observations.md`](accessibility-observations.md) | Phase 10 reporting | Un-named controls and label coverage as reported QA findings |
| [`suite-self-validation.md`](suite-self-validation.md) | Always — it is the root-cause fix | Assertion provenance, negative control, classification provenance |
| [`coverage-ledger.md`](coverage-ledger.md) | Always — it gates the report | Every discovered artifact ends `TESTED` or `EXCLUDED` with a reason; unexercised capability; route sampling; Phase 4A tripwires; test tags |
| [`workflow-flows.md`](workflow-flows.md) | Target has multi-step business flows | Affordance + transition capture, `INFERRED` flow assembly, per-step gating, `INFERRED → EXERCISED` promotion, partial-walk reporting (`docs/05` §19) |
| [`known-failure-modes.md`](known-failure-modes.md) | Before building any phase | Eleven real defects with measured cost and the rule preventing each |
| [`w7-api-evidence-contract.md`](w7-api-evidence-contract.md) | Any work touching API/network output | W7-A/B/C + AIC v1.0.0 preserved; derived views; one masking authority |
| [`write-operations-and-test-data.md`](write-operations-and-test-data.md) | Before planning a write | `ALLOW_WRITE_TESTS` gate, run-scoped markers, persistence verification |
| [`output-and-scope.md`](output-and-scope.md) | Setting up output | The `./output/qa/` tree, categories, stable IDs, provenance |
| [`scope-enforcement.md`](scope-enforcement.md) | Env contract and scope checks | Environment variables, the scope check, cleanup and determinism |
| [`final-report-md.md`](final-report-md.md) | Writing the human report | The 15 required sections and their content |
| [`final-report-json.md`](final-report-json.md) | Producing machine-readable output | `final-report.json`, `summary.json`, `run.json`, retry/healing, failure evidence, checklist |
| [`evidence-and-confidence.md`](evidence-and-confidence.md) | Verifying the evidence chain | Evidence Object → Evidence Quality → Framework Confidence |
| [`execution-entry-point-01.md`](execution-entry-point-01.md) … [`-07.md`](execution-entry-point-07.md) | A reviewer needs the full operational contract | Preserved verbatim, split only to satisfy the reference cap |

## Deep specification (authority)

`../docs/` holds the 18-document authoritative specification. Routing notes in
`SKILL.md` and in these reference files cite it. Key entries:

- `docs/01_Master_Architecture.md` — lifecycle, states, gates, identity (§30, §30.1)
- `docs/02_Decision_Engine.md` — decisions, evidence (§18/§36), Framework Confidence (§21), audit (§39)
- `docs/07_Execution_Engine.md` — runtime authority, environment verification (§17/§31)
- `docs/09_Reporting_Analytics.md` — reporting integrity
- `docs/10_AI_Learning_Repository.md` — learning, scope partitioning (§7.x)
- `docs/IMPLEMENTATION_PLAYBOOK.md` — tactics; Evidence Quality §19; F-1 terms §20.1–§20.3; health pre-flight §18
- `docs/Architecture_Ownership_Matrix.md` — one owner per capability/dataset

## Not applicable (attack-only constructs)

This is a **defensive** QA skill and performs no security testing. It defines no
`scenarios/` exploit recipes, no `PATT` URL, no `findings/finding-NNN/` + CVSS,
no ZAP / OWASP scanning, and does not participate in the skeptic/validator
finding-verification loop. Its results are **functional QA results only**,
recorded in `qa/raw/decision-history.jsonl` and `qa/report/`.
