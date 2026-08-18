# qa-automation — Reference Navigator

Navigator for this skill's reference set. The coordinator passes one or two of
these files to an executor via `SKILL_FILES`; `SKILL.md` itself is never passed
to an executor.

| File | Give to an executor when… | Contents |
|------|---------------------------|----------|
| [`quickstart.md`](quickstart.md) | Running the skill end to end | Minimal executor playbook: scope → explore → plan → generate → execute → report, exact commands, exploration guarantee, masking and dedup rules |
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
- `docs/IMPLEMENTATION_PLAYBOOK.md` — tactics; QA category catalogue §5; Evidence Quality §19; F-1 terms §20.1–§20.3; health pre-flight §18; **deep discovery, API capture, bounded exploration, dedup/no-re-crawl and exploration modes §21**
- `docs/Architecture_Ownership_Matrix.md` — one owner per capability/dataset

## Not applicable

**Attack-only constructs.** No `scenarios/` exploit recipes, no `PATT` URL, no
`findings/finding-NNN/` + CVSS, and no participation in the skeptic/validator
finding-verification loop.

**Removed in W8 and not to be reintroduced.** Accessibility/WCAG testing ·
performance testing · responsive testing · cross-browser testing · security
testing · passive security scanning · offensive testing · severity/CVSS/finding
generation · Skill Applicability Tagging · security-skill recommendation ·
security probability or confidence · attack-surface routing · any Test Catalogue
used as a recommendation engine.

**Retained.** The Test Catalogue as an internal test-identity and
execution/audit record (`06` §22.1, §31) — facts only, never applicability.
Semantic metadata (ARIA, role, label, accessible name) as locator and
self-healing evidence (`03` §7, `06` §26, `08`) — removing accessibility
*testing* did not remove them.

This skill's output is exploration evidence and QA results, recorded in
`qa/aic/`, `qa/decision-history/` and `qa/reports/`.
