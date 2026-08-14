# QA Automation — Quick Start

> You received this as `SKILL_FILES`. Follow it directly. You have `OUTPUT_DIR`,
> `TARGET` (or `TARGET_DOMAIN` / `BASE_URL`), and `CHAIN_CONTEXT`. This is the
> minimal executor playbook: scope → discover → plan → generate → execute →
> report. Deeper contracts are referenced inline but you do not need them to run.

This skill is **defensive QA**. It never exploits, brute-forces, or bypasses
authentication. Discovery is observational.

## 0. Resolve inputs

```bash
BASE_URL="${BASE_URL:-$TARGET}"                 # full URL to the app entry point
[ -z "$BASE_URL" ] && { echo "[ABORT] no BASE_URL/TARGET"; exit 2; }
QA_OUT="$OUTPUT_DIR/qa"
mkdir -p "$QA_OUT"/{discovery,knowledge,planning,tests,pages,fixtures,decision-history,execution-history,reports,raw}
# Credentials are OPTIONAL — absence narrows scope, it does not stop the run.
#   TEST_USER / TEST_PASS  → authenticated surface
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
cd "$QA_OUT"
npm init -y >/dev/null
npm i -D @playwright/test axe-core typescript @types/node >/dev/null
npx playwright install chromium        # NO --with-deps, NO sudo
```

Expected: `chromium` present under the Playwright cache; no system packages
touched.

## 3. Discover (Phase 1) — observational crawl

Drive Chromium over the public surface (and authenticated surface if
`TEST_USER`/`TEST_PASS` are set). For each page: navigate, `networkidle`,
screenshot, extract components (forms/inputs/buttons/tables/nav), capture
XHR/fetch, record auth indicators. Write:

```
qa/discovery/discovery-report.json     # pages, network, auth, skipped (with caps disclosed)
qa/discovery/component-inventory.json  # UNVERIFIED locator candidates
qa/raw/discovery-screenshots/*.png
```

Expected: a JSON report listing every visited page with HTTP status and load
time; a candidate inventory marked `verified:false`. A URL cap is fine — record
what was skipped, never hide it.

## 4. Verify locators (Phase 3) — no unverified locator reaches generation

Probe every candidate live, **in the auth state its test will run in** (public
page → clean context; authenticated page → logged-in context). Keep only
candidates that resolve to the expected count.

```
qa/discovery/verified-inventory.json   # probed → verified/pruned, matchCount per candidate
```

Expected: each kept candidate has `matchCount == 1`; pruned candidates are
recorded with their failing count. Assertion-target elements (error banners,
validation messages) are probed in their *triggered* state — never assumed.

## 5. Plan (Phase 4) + decisions

Derive coverage, workflows, risk-ordered test cases, and the browser matrix
(**chromium only** unless authorized otherwise). Emit the plan and the decision
records — every decision carries measured evidence (see
`evidence-and-confidence.md`).

```
qa/planning/test-plan.json
qa/decision-history/decision-000N.json
```

## 6. Generate (Phase 5) + validate (Phase 6)

Generate Page Objects, fixtures, and specs **only** from verified locators.
Then gate:

```bash
npx tsc --noEmit          # FAIL stops — poor automation never executes
```

Expected: `tsc` clean; a validation report with gate `PASS` (WARNING may
proceed, FAIL stops).

## 7. Execute (Phase 7) — Chromium only

```bash
BASE_URL="$BASE_URL" npx playwright test --project=chromium
```

Expected: a real pass/fail result. A failure is triaged, not hidden — classify
before any healing (`08` §16): `LOCATOR` may heal; `ASSERTION` / business /
auth / security / environment may **not**.

## 8. Report (Phase 10)

Aggregate per test (never per attempt). Reconcile every number against
`qa/reports/results.json`. Disclose skipped scope, degraded/absent optional
engines, and every gap.

```
qa/reports/{results.json,results.xml,html/,diagnostics.json,rw-report.md}
```

Emit the evidence chain in the decision records: per-item confidence →
`evidenceQuality` → Framework Confidence (or the named unavailable terms). See
`evidence-and-confidence.md`.

## Checkpoints worth noting (the executor logs these)

- Discovery complete: N pages, M candidates, K skipped (+ cap).
- Phase-3 gate: candidates verified / pruned.
- Validation gate: PASS / WARNING / FAIL.
- Execution: passed / failed / skipped, reconciled against the runner.
- Any target defect vs suite defect vs environment artifact.

## Never

- Never run against an out-of-scope or unauthorized target.
- Never fabricate a result, a locator, or a confidence value.
- Never present a capped or single-browser run as complete.
- Never `sudo`, `install-deps`, or modify the host environment.
