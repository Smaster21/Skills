# QA Automation — Quick Start

> You received this as `SKILL_FILES`. Follow it directly. You have `OUTPUT_DIR`,
> `TARGET` (or `TARGET_DOMAIN` / `BASE_URL`), and `CHAIN_CONTEXT`. This is the minimal
> executor playbook: scope → profile → discover → verify → plan → generate → execute →
> report. Deeper contracts are referenced inline; you do not need them to run.

This skill is **defensive QA**. It never exploits, brute-forces, or bypasses
authentication. Discovery is **read-only**: it navigates and opens
navigation menus, but never submits a form and never activates a destructive control.

Three rules override convenience everywhere below:
**(1)** no application-specific selector is ever hand-written;
**(2)** nothing is capped and nothing is silently dropped;
**(3)** a case binds to a locator matching its *intent*, or it is dropped.

## 0. Resolve inputs

```bash
BASE_URL="${BASE_URL:-$TARGET}"                 # full URL to the app entry point
[ -z "$BASE_URL" ] && { echo "[ABORT] no BASE_URL/TARGET"; exit 2; }
QA_OUT="$OUTPUT_DIR/qa"
# Run isolation is at the ./output/ boundary — one output root per run.
mkdir -p "$QA_OUT"/{report,discovery,network,knowledge,planning,coverage,diagnostics,raw} \
         "$QA_OUT"/tests/{generated,pages,fixtures} \
         "$QA_OUT"/execution \
         "$QA_OUT"/evidence/{tests,screenshots,traces,videos,network,raw}

# One artifact, one category. Stable IDs (PAGE-/ROUTE-/API-/WF-/TC-/EVID-) are the
# join keys; every model item declares observed | inferred | generated.
# QA output only — no security findings, CVSS, or security coverage anywhere.
# Credentials are OPTIONAL — absence narrows scope, it does not stop the run.
#   TEST_USER / TEST_PASS supplied  → Authenticated Exploration
#   TEST_USER / TEST_PASS absent    → Unauthenticated Exploration
# (Authorization/scope governance is mandatory in BOTH modes — see step 1.)
```

## 1. Scope check — before ANY navigation

```bash
# Abort if the target (or its resolved IP) is not in SCOPE_FILE.
if [ -f "$SCOPE_FILE" ]; then
  host=$(echo "$BASE_URL" | sed -E 's#^[a-z]+://##; s#/.*$##; s#:.*$##')
  if ! grep -qE "$(echo "$host" | sed 's/\./\\./g')" "$SCOPE_FILE"; then
    ip=$(dig +short "$host" | head -1)
    if [ -z "$ip" ] || ! grep -qE "$(echo "$ip" | sed 's/\./\\./g')" "$SCOPE_FILE"; then
      echo "[ABORT] $host (or its IP) not in SCOPE_FILE"; exit 1
    fi
  fi
fi
# Also honor RoE: if ROE_FILE defines a blackout/expiry window, respect it.
```

## 2. Scaffold + install (Chromium only)

```bash
cd "$QA_OUT/tests"          # dependencies live with the suite, not with the evidence
npm init -y >/dev/null
npm i -D @playwright/test typescript @types/node >/dev/null
npx playwright install chromium        # NO --with-deps, NO sudo
```

Expected: `chromium` present under the Playwright cache; no system packages
touched.

## 3–4. Profile the target, then discover

Profile before crawling — measure the target's own conventions — then crawl
**depth-first and exhaustively**, expanding navigation disclosures where profiling
showed they reveal routes. Read-only; caps come from configuration only. **Every**
discovered route instance is crawled, within the declared `MAX_ROUTE_INSTANCES`
ceiling — no sampling, no early stop, and depth grants no extra authority
(`scope-enforcement.md`). Steps and artifacts: **`discovery-profiling.md`**.

## 5. Verify locators (Phase 3) — no unverified locator reaches generation

Probe every candidate live, **in the auth state its test will run in**, using the
strategy ladder (testid → role+name → label → name/id → **label-scoped** →
placeholder → text). Keep only `matchCount === 1`.

```
qa/discovery/verified-inventory.json
qa/discovery/assertion-target-probes.json
```

For candidates pruned as ambiguous, retry **label-scoped** (find the field group
by its label, then the control inside it) before giving up — that single strategy
recovers most repeated-placeholder fields. Assertion targets are probed in their
**triggered** state, never assumed.

## 6. Plan (Phase 4) — workflows, intent-bound, fully ledgered

Derive coverage across every declared area, then bind each case to verified
locators **by intent**:

> If no verified locator's accessible name matches the case's stated target,
> **drop the case** and record `NO_INTENT_MATCH` in the ledger. Never fall back to
> "the first verified control on the route" — that produces a test that passes
> while driving the wrong element.

Form cases are **workflows**, not presence checks:

```
populate every verified field (click-then-pick for custom widgets,
selectOption only for native <select>)
  → submit
  → assert the resulting state changed as expected
  → reset where offered, assert it cleared
```

Also plan `business-rule` cases from validation messages captured in their
triggered state, and `CRUD-except-Delete` where the environment permits writes —
if it does not, that is a `POLICY_EXCLUDED` ledger entry **with its consequence
stated**, not an omission. Writes escalate on one **per-surface** ladder; each
irreversible surface needs its own opt-in (`write-operations-and-test-data.md`).

```
qa/planning/test-plan.json
qa/raw/decision-history.jsonl → decision-000N.json
qa/coverage/ + qa/diagnostics/coverage-ledger.json    ← every population accounted, unaccounted = 0
```

## 6a. Review the plan (Phase 4A) — deterministic first, then advisory

Two layers, different authority. **The deterministic half always runs** — arithmetic over
the ledger, needing nothing but the plan. It writes `qa/planning/plan-review.json` with
`deterministic.verdict` (`PASS`/`WARNING`/`FAIL`) and one finding per fired tripwire
(`rule`, `status`, `measuredValue`, `threshold`, `reason`, `affectedArea`).

Then review **yourself** — you are the reviewer; the skill ships no model client. Read
those findings plus `coverage/coverage-summary.json` and `planning/test-plan.json`, then
write `aiReview` into the **same** file, keeping `MEASURED FACT` / `AI INTERPRETATION` /
`RECOMMENDATION` apart and marking each finding `EXPECTED` or `SUSPICIOUS`.
> **Never remove, downgrade, or rewrite a deterministic finding** — explain it, but its
> `status` stands. Cannot review? Set `aiReview.state` to `UNAVAILABLE` with a reason;
> never fabricate one, never default to `PASS`. `WARNING` does not stop the run — 4A is
> advisory; only `02` withholds approval (`planning-coverage-controls.md`).

## 6b. Generate (Phase 5) + validate (Phase 6)

Generate Page Objects, fixtures and specs **only** from verified locators, using
the profile's shell facts **per route** (never one route's anchor applied
globally). Then gate:

```bash
npx tsc --noEmit          # FAIL stops — poor automation never executes
```

## 7. Execute (Phase 7) — Chromium only

```bash
BASE_URL="$BASE_URL" npx playwright test --project=chromium
```

Expected: a real pass/fail result. A failure is triaged, not hidden — classify
before any healing (`08` §16): `LOCATOR` may heal; `ASSERTION` / business /
auth / environment may **not**.

## 8. Report (Phase 10)

Aggregate per test (never per attempt). Reconcile every number against
`qa/execution/results.json`. Write `summary.json` carrying the verdict **and
its qualifiers together**, then write `qa/run.json`, `qa/README.md`, and
`qa/report/final-report.{md,json}`. **Publish the coverage ledger beside the pass counts** —
discovered / tested / excluded per population, plus verified-but-unused. A pass
count describes the suite; only the ledger describes the application
(`coverage-ledger.md`). Disclose skipped scope, degraded/absent optional engines,
and every gap.

```
qa/summary.json                      ← the single file to read first
qa/execution/{results.json,results.xml,html/}
qa/coverage/ + qa/diagnostics/{diagnostics.json,coverage-ledger.json,target-health.json,analytics.json}
qa/report/{final-report.md,final-report.json}   ← the deliverable
qa/run.json  qa/summary.json  qa/README.md
```

Emit the evidence chain in the decision records: per-item confidence →
`evidenceQuality` → Framework Confidence (or the named unavailable terms). See
`evidence-and-confidence.md`.

## Checkpoints worth noting (the executor logs these)

- Profile complete: shell landmarks, label coverage ratio, choice-control style.
- Discovery complete: N pages, M candidates, K skipped **with reasons**; route instances crawled per template, and any ceiling that stopped one.
- Phase-3 gate: candidates verified / pruned.
- Validation gate: PASS / WARNING / FAIL.
- Execution: passed / failed / skipped, reconciled against the runner.
- Ledger: discovered / tested / excluded per population; `unaccounted` MUST be 0.
- Verified-but-unused count; any target vs suite vs environment defect.

## Never

- Never run against an out-of-scope or unauthorized target.
- Never fabricate a result, a locator, or a confidence value.
- Never cap silently — no URL cap, no per-category cap, no `slice(n)`. A limit is a declared exclusion with a reason, or it does not happen.
- Never hand-write an application-specific selector, class, URL prefix or literal.
- Never bind a case to a locator that does not match its stated intent.
- Never present a single-browser run, one with unaccounted artifacts, or one whose ceiling excluded instances, as complete.
- Never `sudo`, `install-deps`, or modify the host environment.
