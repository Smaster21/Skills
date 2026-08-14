# qa-automation — Reference Navigator

Navigator for this skill's reference set. The coordinator passes one or two of
these files to an executor via `SKILL_FILES`; `SKILL.md` itself is never passed
to an executor.

| File | Give to an executor when… | Contents |
|------|---------------------------|----------|
| [`quickstart.md`](quickstart.md) | Running the skill end to end | Minimal executor playbook: scope → discover → plan → generate → execute → report, exact commands |
| [`evidence-and-confidence.md`](evidence-and-confidence.md) | A run must produce/verify the evidence chain | Evidence Object → Evidence Quality → Framework Confidence, routing into `docs/` §19/§20/§21; unavailable-state vocabulary |
| [`output-and-scope.md`](output-and-scope.md) | Setting up output/env, or enforcing scope | `$OUTPUT_DIR/qa/…` mapping, env-var contract, scope-abort pattern, cleanup/determinism |
| [`execution-entry-point.md`](execution-entry-point.md) | A reviewer needs the full operational contract | The complete WHEN / lifecycle / governance spec, preserved verbatim from the standalone framework |

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

This is a **defensive** QA skill. It defines no `scenarios/` exploit recipes, no
`PATT` URL, no `findings/finding-NNN/` + CVSS, and does not participate in the
skeptic/validator finding-verification loop. Its results are functional and
Tier-1 passive observations, recorded in `qa/decision-history/` and `qa/reports/`.
