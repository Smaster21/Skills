# QA Automation — Quick Start

> You received this as `SKILL_FILES`. Follow it directly. You have `OUTPUT_DIR`,
> `TARGET` (or `TARGET_DOMAIN` / `BASE_URL`), and `CHAIN_CONTEXT`. This is the
> minimal executor playbook: scope → discover → plan → generate → execute →
> report. Deeper contracts are referenced inline but you do not need them to run.

This skill is the **Site Explorer** backend. It explores and verifies; it never
exploits, brute-forces, or bypasses authentication, and it never assesses
security. Discovery is observational.

**Exploration guarantee.** Never claim 100% or "every page". You guarantee *all
reachable, in-scope surfaces within the configured budget*. Every
discovered-but-not-reached surface gets exactly one reason: `inaccessible ·
blocked · capped · excluded · unavailable state · unavailable credentials`.

**Exploration mode.** `TEST_USER`/`TEST_PASS` present → **Authenticated
Exploration**; absent → **Unauthenticated Exploration**. Neither is
authorization — the scope check in §1 is mandatory in both.

## 0. Resolve inputs

```bash
BASE_URL="${BASE_URL:-$TARGET}"                 # full URL to the app entry point
[ -z "$BASE_URL" ] && { echo "[ABORT] no BASE_URL/TARGET"; exit 2; }
QA_OUT="$OUTPUT_DIR/qa"
mkdir -p "$QA_OUT"/{discovery,knowledge,planning,tests,pages,fixtures,decision-history,execution-history,reports,raw}
mkdir -p "$QA_OUT"/aic/evidence/{requests,responses,screenshots}
# Credentials are OPTIONAL — absence narrows scope, it does not stop the run.
#   TEST_USER / TEST_PASS   → Authenticated Exploration
#   ALLOW_WRITE_TESTS=1     → permits state-mutating tests (synthetic data +
#                             mandatory cleanup + disclosed mutation). Never
#                             enables application-delete tests.
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
npm i -D @playwright/test typescript @types/node >/dev/null
npx playwright install chromium        # NO --with-deps, NO sudo
```

Expected: `chromium` present under the Playwright cache; no system packages
touched.

## 3. Explore (Phase 1) — observational

Drive Chromium over the in-scope surface. Discovery sources — all of them, each
tagged with `provenance.discoverySource` (`PLAYBOOK` §21):

| Source | Notes |
|---|---|
| page/link crawl | BFS within caps, same-origin or `SCOPE_FILE` hosts |
| routes + dynamic route templates | collapse `/users/123` → `/users/{id}` |
| `robots.txt` | parse; a `Disallow` is intelligence — **record it, never crawl it** |
| `sitemap.xml` | parse `<loc>` routes |
| JavaScript routes | route/API strings in already-loaded JS; a string is **not** proof |
| GraphQL endpoints | indicators in JS and network |
| hidden endpoints | discovered from js/sitemap/robots and **not** page-linked — record, never auto-exercise |
| forms · inputs · parameters | component extraction |
| authentication surfaces | login pages, session cookies, token storage, OAuth/OIDC/SSO, MFA indicators |
| network/API capture | every XHR/fetch witnessed → masked request + response evidence |

Per page: navigate, `networkidle`, screenshot, extract components, capture
XHR/fetch, record auth indicators.

**Three non-negotiables:**

1. **Mask before persistence.** Sensitive header/body names *and* value shapes
   (Bearer, JWT, `AKIA…`, `sk_live_…`, high entropy) → `masked`. JSON/form bodies
   → `redacted`. Binary → `binary` repr (contentType + size + sha256, never raw).
   Unknown sensitivity defaults to `masked`.
2. **Deduplicate.** Deterministic identity per surface. Revisit only for a
   recorded reason: distinct state · distinct auth context · workflow transition
   · validation · authorized re-discovery.
3. **Record every unreached surface** with exactly one reason.

State ladder: `DISCOVERED → OBSERVED → EXERCISED → VALIDATED`. **Never emit
`OFFENSIVELY_VALIDATED` or `VULNERABILITY_CONFIRMED`** — they are not yours.

Write:

```
qa/discovery/discovery-report.json     # pages, network, auth, unreached (each with a reason)
qa/discovery/component-inventory.json  # UNVERIFIED locator candidates
qa/aic/{pages,routes,apis,api-calls,forms,parameters,auth-surfaces,robots,javascript-routes,relationships}.json
qa/aic/evidence/{requests,responses}/*.json   # masked
qa/raw/discovery-screenshots/*.png
```

Expected: every visited page with HTTP status and load time; a candidate
inventory marked `verified:false`; `attack-surface` absent or `NOT_PRODUCED`. A
cap is fine — record what was skipped, never hide it.

**Record facts, never conclusions.** "A parameter exists" ✅. "This parameter is
injectable" ❌ — that is the security skill's call, not yours.

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

Derive coverage, workflows and risk-ordered test cases across the ten QA
categories — smoke · functional · UI · forms · authentication · navigation · API,
plus dashboard/table/visual **only if explicitly enabled**. Browser matrix is
**chromium**, full stop.

Functional = positive · boundary · business-rule · **CRUD except Delete** ·
state-transition. Never plan a delete test, an accessibility/WCAG check, a
performance budget, a viewport matrix, or any security test.

Emit the plan and the decision records — every decision carries measured
evidence (see `evidence-and-confidence.md`).

```
qa/planning/test-plan.json
qa/decision-history/decision-000N.json
```

## 6. Generate (Phase 5) + validate (Phase 6)

Generate Page Objects, fixtures, and specs **only** from verified locators.

**Never generate:** SQL injection, XSS, SSRF, IDOR, command-injection or
traversal tests · attack-shaped input payloads · application-delete tests ·
accessibility/performance/responsive/cross-browser tests. A test that creates
synthetic data still cleans up **its own** data in `afterEach` — that is cleanup,
not a Delete test.

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
before any healing (`08` §16): `LOCATOR` may heal; `ASSERTION` / business-rule /
auth / environment may **not**.

A failed QA test is a **target defect, suite defect, or environment artifact**.
It is NEVER a security finding, and it never earns a severity or a CVSS score.

## 8. Report (Phase 10)

Aggregate per test (never per attempt). Reconcile every number against
`qa/reports/results.json`. Disclose skipped scope, degraded/absent optional
engines, and every gap.

**The Exploration Disclosure is mandatory** (`09` §12A): exploration mode
(Authenticated/Unauthenticated), the configured budget and whether any cap was
hit, every unreached surface with its reason, every revisit with its reason, and
which optional categories did not run.

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
- Never present a capped or credential-limited run as complete.
- Never claim 100% coverage or that "every page" was explored.
- Never `sudo`, `install-deps`, or modify the host environment.
- Never conclude that a surface is vulnerable, that an attack class applies to
  it, or that a security skill should be run against it.
- Never emit `OFFENSIVELY_VALIDATED` or `VULNERABILITY_CONFIRMED`.
- Never persist a raw secret.
