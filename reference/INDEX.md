# qa-automation — Reference Navigator

Navigator for this skill's reference set. The coordinator passes one or two of
these files to an executor via `SKILL_FILES`; `SKILL.md` itself is never passed
to an executor.

| File | Give to an executor when… | Contents |
|------|---------------------------|----------|
| [`quickstart.md`](quickstart.md) | Running the skill end to end | Minimal executor playbook: scope → profile → discover → verify → plan → generate → execute → report |
| [`discovery-profiling.md`](discovery-profiling.md) | Profiling a target and crawling it | Phase 0B profiling, then the depth-first exhaustive read-only crawl |
| [`rules-extended.md`](rules-extended.md) | Rules 11–25 | Target-agnostic locators, coverage ledger, intent binding, workflow forms, auth isolation, measured responses, evidence-backed classification, session resilience, per-route baselines, write gating, QA-only output, stable IDs, provenance, healing honesty, immutability |
| [`execution-lifecycle.md`](execution-lifecycle.md) | Understanding phase behaviour | Read-only exhaustive discovery, no-caps + ledger contract, target-agnostic construction, suite-soundness gates, evidence chain, scope handling |
| [`site-agnostic-discovery.md`](site-agnostic-discovery.md) | Building or reviewing discovery, locators, or interaction | Site profiling, standards-based labels, behavioural widget detection, locator ladder, intent rule, depth-first traversal, read-only boundary |
| [`suite-self-validation.md`](suite-self-validation.md) | Always — it is the root-cause fix | Assertion provenance, negative control, classification provenance |
| [`coverage-ledger.md`](coverage-ledger.md) | Always — it gates the report | Every discovered artifact ends `TESTED` or `EXCLUDED` with a reason |
| [`known-failure-modes.md`](known-failure-modes.md) | Before building any phase | Ten real defects with measured cost and the rule preventing each |
| [`w7-api-evidence-contract.md`](w7-api-evidence-contract.md) | Any work touching API/network output | W7-A/B/C + AIC v1.0.0 preserved; derived views; one masking authority |
| [`write-operations-and-test-data.md`](write-operations-and-test-data.md) | Before planning a write | `ALLOW_WRITE_TESTS` gate, run-scoped markers, persistence verification |
| [`output-and-scope.md`](output-and-scope.md) | Setting up output | The `./output/qa/` tree, categories, stable IDs, provenance |
| [`scope-enforcement.md`](scope-enforcement.md) | Env contract and scope checks | Environment variables, the scope check, cleanup and determinism |
| [`final-report-md.md`](final-report-md.md) | Writing the human report | The 15 required sections and their content |
| [`final-report-json.md`](final-report-json.md) | Producing machine-readable output | `final-report.json`, `summary.json`, `run.json`, retry/healing, failure evidence, checklist |
| [`evidence-and-confidence.md`](evidence-and-confidence.md) | Verifying the evidence chain | Evidence Object → Evidence Quality → Framework Confidence |
| [`execution-entry-point-01.md`](execution-entry-point-01.md) … [`-06.md`](execution-entry-point-06.md) | A reviewer needs the full operational contract | Preserved verbatim, split only to satisfy the reference cap |

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
