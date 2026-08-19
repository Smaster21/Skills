# Extended Rules (11–25)

> Executor reference. Rules 1–10 are in `SKILL.md`. These extend them and carry
> equal force. Each states the required behaviour positively; the failure it
> prevents is named in `known-failure-modes.md`.


11. **Target-agnostic.** No application-specific selector, class, URL prefix or
    text literal may be hand-written. Locators come from web standards or from
    measurements in `qa/discovery/discovery-summary.json`
    (`reference/site-agnostic-discovery.md`).
12. **No caps, no silent drops.** No URL cap, per-category cap, or `slice(n)`.
    Every discovered artifact ends as `TESTED` or `EXCLUDED` with a reason in
    `qa/coverage/coverage-summary.json`; `unaccounted > 0` blocks a completeness
    claim (`reference/coverage-ledger.md`).
13. **Intent over availability.** A planned case binds only to a verified locator
    whose accessible name matches its stated intent. If none matches, DROP the
    case and ledger it `NO_INTENT_MATCH` — never substitute another element. A
    test that passes while driving the wrong element is worse than a failure.
14. **Forms are workflows.** Form coverage means populate → submit → assert the
    resulting state (and reset where offered). Asserting visibility and that a
    field accepts text is NOT form coverage.
15. **One auth state per spec file.** A file-scoped state override applies to the
    whole file; mixing anonymous and authenticated tests in one file silently
    strips sessions (`known-failure-modes.md` FM-1, 21 failures).
16. **Assert measured responses, never observed requests.** Expectations come
    from W7-C's **already-captured** response evidence — never from the existence
    of a request, and never from a bulk replay. Where a response is
    `NOT_OBSERVED`, a targeted probe recorded as `discoverySource: verification`
    is the narrow exception. Never assume `200` (FM-3,
    `reference/w7-api-evidence-contract.md`).
17. **Classification needs evidence.** A failure may be called a target defect
    only when a probe against the live target supports it; a probe result
    overrides any text heuristic and MUST be cited. Otherwise `UNCLASSIFIED` —
    never default to blaming the target (FM-5).
18. **Sessions are not assumed to survive.** Detect landed-on-login,
    re-authenticate once, and **report every recovery** — a recovered pass is not
    a clean pass (FM-4).
19. **Baselines are per route.** A measurement on one route is evidence about
    that route only; never promote it to a global invariant (FM-2).
20. **Writes are gated by `ALLOW_WRITE_TESTS`.** State-mutating QA tests require
    `ALLOW_WRITE_TESTS=1` (W7-A BD-W7-3) plus synthetic data, mandatory cleanup,
    scope + RoE, and disclosed mutation; application-delete tests are never
    enabled. Absent or `0` ⇒ writes are ledgered, not silently skipped
    (`reference/write-operations-and-test-data.md`).
21. **QA output only — no security layer.** The output is limited to QA and
    application-exploration content: coverage is **Application Exploration
    Coverage** and failures use QA categories. Vulnerability findings, CVSS,
    exploitability, attack priority, security coverage, security
    recommendations, and any judgement about a target being secure are all out
    of scope and are owned by separate security skills.
22. **Stable IDs and traceability — reuse, never re-invent.** Where an AIC record
    already carries an identifier, reuse it verbatim; readable aliases are a 1:1
    display mapping only. Never create a second API identity system. Page →
    workflow → test case → spec → result → evidence must be traceable end to end.
23. **Provenance in W7-B's vocabulary.** `provenance.discoverySource` ∈
    `crawl · js · sitemap · robots · network · verification`; the state ladder
    `DISCOVERED → OBSERVED → EXERCISED → VALIDATED` is never collapsed and
    `VALIDATED` is never emitted by QA. The report's observed/discovered/inferred
    wording presents these fields — it is not a parallel scheme. Inferred or
    generated material is never rendered as observed.
24. **No overclaimed healing.** Report retries as retries; record locator healing
    only when it was actually attempted. `blocked` and `failed` stay distinct.
25. **Artifacts are immutable and namespaced.** Filenames scoped per pass and run;
    identifiers namespaced per producing pass (FM-7).

---
