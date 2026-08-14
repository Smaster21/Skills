# QA Automation Framework

# Implementation Playbook

**Document:** IMPLEMENTATION_PLAYBOOK.md

**Version:** 3.0

**Status:** Non-Normative

**Depends On:** all of `01_Master_Architecture.md` … `10_AI_Learning_Repository.md`

---

# Status of This Document

This document is **non-normative**.

It is the lowest-precedence document in the framework.

It contains the concrete Playwright/TypeScript tactics, tool configuration, and
field-learned failure modes required to implement the numbered specifications on
a real target. It defines **how**, never **what** or **why**.

Precedence rules:

- `01_Master_Architecture.md` SHALL always take precedence.
- Documents `02`–`10` SHALL take precedence over this document.
- Where this document conflicts with any numbered specification, the numbered
  specification wins and this document is defective.
- Nothing in this document may be read as authorising an engine to make a
  decision reserved to the Decision Engine (`02`).

This document SHALL NOT be treated as a specification, and SHALL NOT be used to
justify skipping a lifecycle stage or quality gate defined in `01`.

---

# Table of Contents

1. Project Scaffold
2. Canonical Playwright Configuration
3. Discovery Tactics
4. Locator Verification Probe
5. Test Category Catalogue
6. Test Data Management
7. Base Page Object Template
8. Retry Ladder Implementation
9. Security Tooling — ZAP Tiers
10. OWASP Top 10 Mapping
11. Performance Budgets
12. Failure Artifact Checklist
13. Report Formats
14. CI/CD Integration
15. Hard-Won Rules
16. Reporting Integrity
17. Manual Observation Mode
18. Target Health Pre-Flight
19. Evidence Quality Measurement
20. Framework Confidence — Remaining §21 Term Measurement

---

# 1. Project Scaffold

Implements the Project Assembly Engine (`06` §30).

```
tests/smoke/  tests/regression/  tests/authentication/  tests/ui/
tests/navigation/  tests/forms/  tests/dashboard/  tests/tables/
tests/api/  tests/accessibility/  tests/performance/  tests/security/
tests/visual/  tests/e2e/
pages/  components/  fixtures/  utils/  data/
discovery/  knowledge/  planning/  reports/  artifacts/
playwright.config.ts  README.md
```

Dependency installation:

```bash
npm init -y
npm i -D @playwright/test axe-core
npx playwright install --with-deps chromium
```

Environment inputs:

| Variable | Required | Description |
|---|:---:|---|
| `BASE_URL` | ✅ | Target application URL |
| `TEST_USER` | ❌ | Login username/email |
| `TEST_PASS` | ❌ | Login password |
| `CI` | ❌ | `true` in CI — reduces workers, increases retries |
| `MANUAL` | ❌ | `1` forces headed + 350ms pacing + no retries (§17). Single-window is implied by `--headed` alone. |
| `SLOWMO` | ❌ | Milliseconds of delay per action. Defaults to `350` under `MANUAL=1`, `0` otherwise. |
| `ABORT_IF_UNHEALTHY` | ❌ | `1` makes the target health pre-flight a hard gate (§18). |
| `SECURITY_SCAN` | ❌ | `active` enables ZAP Tier 3. Requires authorization. |
| `ZAP_API_KEY` | ❌ | ZAP daemon API key. Generated per session if absent (§9). Required for T2/T3 only. |

Credentials SHALL arrive via environment variables only, and SHALL be masked
before any persistence (`01` §30, `07` §41).

## Degradation Without Credentials

`TEST_USER` / `TEST_PASS` are optional. Their absence **narrows scope; it does
not stop the run.** Only `BASE_URL` is required.

With no credentials supplied:

| Stage | Behaviour |
|---|---|
| Discovery (`03`) | Runs fully on the public surface. Still detects login pages, session cookies, JWT storage, OAuth/OIDC/SSO and MFA indicators, and marks protected routes — detection never requires logging in (`03` §23). |
| Knowledge Graph (`04`) | Protected pages exist as nodes with `requires → authentication`. The graph records what is unreachable, not a smaller application. |
| Planning (`05`) | Environment validation records credentials as unavailable (`05` §28). Role coverage for authenticated roles is reported as a **coverage gap with a stated cause**, not silently dropped (`05` §18). Auth-dependent workflows are excluded from the plan via the Authentication Dependency (`05` §25). |
| Generation (`06`) | Generates everything the plan selected. Unauthenticated smoke, navigation, forms, public API, accessibility, visual, responsive, performance and T1/T2 security all generate normally. |
| Execution (`07`) | Runs the generated suite. Nothing is attempted against a protected route it cannot reach. |
| Reporting (`09`) | **Must state that authenticated coverage did not run, and why** — an unsupplied credential, not a defect and not a pass. |

Rules:

- The framework SHALL NEVER guess, brute-force, or bypass credentials to close
  the gap (`01` §2 non-goals, `03` §23).
- Missing credentials SHALL NEVER be reported as passing coverage, and an
  auth-gated test SHALL NEVER be counted as verified because it was skipped
  (§16, `09` §5.1).
- A partial-scope run is a legitimate result. Report the scope honestly rather
  than presenting public-surface coverage as complete.

If the target is entirely behind a login wall, discovery yields the login page
and little else. Say so at the gate and request credentials rather than
generating a suite against a single page.

---

# 2. Canonical Playwright Configuration

Use verbatim. Set `testIdAttribute` from what Discovery actually observed —
never assume `data-testid`.

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  outputDir: './test-results',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 1,
  workers: process.env.CI ? 1 : undefined,
  globalTimeout: 0, // set from measured single-pass duration + 50% headroom
  reporter: [
    ['list'],
    ['html', { open: 'never', outputFolder: 'reports/html' }],
    ['json', { outputFile: 'reports/results.json' }],
    ['junit', { outputFile: 'reports/results.xml' }],
    // For Allure: npm i -D allure-playwright, then add ['allure-playwright']
  ],
  use: {
    baseURL: process.env.BASE_URL,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'retain-on-failure',
    // testIdAttribute: detect during Discovery — data-testid, data-qa, data-cy, data-test
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 5'] } },
    { name: 'mobile-safari', use: { ...devices['iPhone 13'] } },
  ],
});
```

The browser matrix is planned by the Browser Planning Engine (`05` §27). This
template is the shape of the output, not the decision.

---

# 3. Discovery Tactics

Implements `03`. Discovery is observational and non-destructive (`03` §5.4, §36).

## Crawl Procedure

1. Navigate to `BASE_URL`, wait for `networkidle`.
2. Extract same-origin `<a href>` links.
3. Extract programmatic routes from `<script>` route tables, `/sitemap.xml`,
   and `robots.txt` (record disallowed paths; do not crawl them).
4. BFS traversal, max depth 3, same-origin only. Per URL:
   a. Navigate, wait for `networkidle`.
   b. Run the interstitial health check (§15).
   c. Screenshot.
   d. Extract components.
   e. Capture XHR/fetch — method, URL, status, content-type.
   f. Record auth indicators: login forms, JWT in storage, session cookies.
5. Normalize URLs per the URL Normalization Engine (`03` §18): trailing slashes,
   case, default ports, duplicate and tracking query parameters, fragments.
6. Emit the artifact set required by `03` §11 and §31.

If ZAP is available, proxy all discovery traffic through it — this yields Tier 2
passive coverage at zero additional request cost.

## Component Extraction

- **Forms**: `form`, fieldsets, input groups
- **Inputs**: `input` (text, email, password, phone, date, file, checkbox,
  radio), `select`, `textarea`, autocomplete widgets, sliders
- **Buttons**: `button`, `[role="button"]`, submit inputs
- **Tables**: `table`, `[role="grid"]`, `[role="treegrid"]`
- **Navigation**: `nav`, `[role="navigation"]`, breadcrumbs, sidebars, headers,
  footers
- **Modals/Overlays**: `[role="dialog"]`, `[aria-modal="true"]`, `.modal`,
  drawers, toasts, alerts, tooltips
- **Data Display**: cards, accordions, tabs, carousels, charts
- **Interactive**: search bars, filters, pagination, infinite scroll, dropdowns
- **Lazy/Dynamic**: intersection-observer content, dynamic routes

Every extracted component carries the metadata required by `03` §21 —
including locator candidates, confidence, and evidence.

---

# 4. Locator Verification Probe

Runs between Discovery validation (`03` §29) and Generation (`06`).

For every locator candidate in the component inventory, execute a throwaway
assertion that the locator **resolves and returns the expected count**. Prune
the inventory to verified candidates only, and emit `verified-inventory.json`.

An unverified locator SHALL NOT reach the Test Generation Engine. See §15 for
the failure modes this exists to prevent.

---

# 5. Test Category Catalogue

Generated per the workflow expansion model in `06` §18. Category order:

smoke → auth → navigation → forms → functional → tables → dashboard → API →
accessibility → visual → responsive → performance → security

| Category | Verifies |
|---|---|
| Smoke | App loads, critical pages 200, primary nav, login completes |
| Functional | Positive, negative, boundary, validation; state transitions, CRUD, permissions, business rules |
| UI | Visibility, alignment, responsive correctness, broken assets, overflow, spacing, typography, theme, dark mode |
| Forms | Empty submit, required, max/min length, unicode, emoji, special chars, injection strings, paste, autofill, disabled/readonly, keyboard order |
| Authentication | Login, logout, register, reset, session timeout, invalid credentials, expiry, JWT, cookie security, CSRF, RBAC |
| Navigation | Menus, sidebar, header/footer, breadcrumbs, deep links, 404, redirects, back/forward, refresh state |
| Dashboard | Widget render, chart accuracy, filters, sorting, search, export, pagination, refresh, permission-based visibility |
| Table | Search, per-column sort, filter, pagination, bulk actions, row select, column resize, sticky header, export, responsive collapse |
| Accessibility | axe-core: WCAG 2.1 AA, ARIA, keyboard, screen reader, focus, labels, contrast ≥4.5:1, heading order, alt text, landmarks |
| Visual | Layout shift, missing elements, alignment drift, spacing, pixel diff |
| Responsive | 1920×1080, 768×1024, 375×667; landscape and portrait |
| Cross-browser | Chromium, Firefox, WebKit + device descriptors |
| Performance | Lighthouse against budgets (§11) |
| API | Endpoints from Discovery network capture: 200/400/401/403/404/500, timeout, retry, schema, contract |
| Security | Tiered ZAP model (§9) |

Naming follows `06` §22 — descriptive business objectives, never `Test1`:

```
login.page.ts       login.spec.ts
dashboard.page.ts   dashboard.spec.ts
table.spec.ts       profile.spec.ts
```

Never combine unrelated tests in one file.

---

# 6. Test Data Management

- **Never use production data.** Generate synthetic fixtures.
- Fixtures in `fixtures/`, one JSON per domain entity.
- `fixtures/factory.ts` exposes builder functions.
- **Cleanup contract**: any test that creates data deletes it in `afterEach`.

## Canonical Test Data

| Field Type | Valid | Invalid | Boundary |
|---|---|---|---|
| Email | `test@example.com` | `not-an-email` | 254-char local part |
| Password | `P@ssw0rd!2024` | `short` | 128 chars |
| Phone | `+1-555-0100` | `abc` | `+999999999999999` |
| Text | `Hello World` | *(empty string)* | 10,000 chars |
| Number | `42` | `abc` | `0`, `-1`, `2147483647` |
| Date | `2024-01-15` | `not-a-date` | `1970-01-01`, `2099-12-31` |
| URL | `https://example.com` | `not-a-url` | 2,083 chars |

## Input-Validation Payloads

Used to verify that the application sanitises input. These are **assertions
about validation behaviour**, not exploitation:

- SQL: `' OR '1'='1`, `'; DROP TABLE users;--`, `1 UNION SELECT null`
- XSS: `<script>alert(1)</script>`, `<img onerror=alert(1) src=x>`,
  `javascript:alert(1)`
- Path traversal: `../../etc/passwd`, `..\\..\\windows\\system32`

---

# 7. Base Page Object Template

Implements `06` §19. Business logic lives in Page Objects, never in specs.

```typescript
// pages/base.page.ts
import { Page, Locator } from '@playwright/test';

export abstract class BasePage {
  constructor(protected readonly page: Page) {}

  abstract readonly url: string;

  async navigate(): Promise<void> {
    await this.page.goto(this.url, { waitUntil: 'networkidle' });
    await this.checkForInterstitial();
  }

  protected async checkForInterstitial(): Promise<void> {
    const title = (await this.page.title()).toLowerCase();
    const body = await this.page.locator('body').textContent().catch(() => '') ?? '';
    const bodyLower = body.toLowerCase();
    const interstitials = [
      'just a moment', 'checking your browser', 'under maintenance',
      'queue full', 'under heavy load', 'captcha', 'access denied',
    ];
    for (const pattern of interstitials) {
      if (title.includes(pattern) || bodyLower.includes(pattern)) {
        throw new Error(`[ENVIRONMENT] Interstitial detected: "${pattern}" on ${this.page.url()}`);
      }
    }
  }

  async waitForReady(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }
}
```

```typescript
// pages/login.page.ts
import { BasePage } from './base.page';

export class LoginPage extends BasePage {
  readonly url = '/login';

  readonly emailInput = this.page.getByLabel('Email');
  readonly passwordInput = this.page.getByLabel('Password');
  readonly submitButton = this.page.getByRole('button', { name: 'Login' });

  async login(email: string, password: string): Promise<void> {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
    await this.page.waitForLoadState('networkidle');
  }
}
```

---

# 8. Retry Ladder Implementation

Implements `01` §21 and `08` §33. **Every rung requires Decision Engine
approval** (`08` §33) — this ladder describes the mechanics, not the authority.

**Attempt 1 — plain retry.** Confirm the element is visible, enabled, stable;
the page has settled; network is idle.

**Attempt 2 — adaptive interaction.** Wait for visibility and attachment,
scroll into view, hover before click, keyboard interaction, re-query stale
references. Force-click only where the Policy Engine permits it.

**Attempt 3 — self-healing.** Hand off to `08`. Candidate locators follow the
framework locator priority (`01` §20, `06` §26):

1. `getByTestId()`
2. `getByRole()`
3. `getByLabel()`
4. `getByPlaceholder()`
5. `getByText()`
6. `aria-label`
7. `name`
8. `id`
9. Stable CSS
10. Relative XPath — last resort

Absolute XPath is prohibited. Never generate `nth-child`, dynamic CSS classes,
auto-generated IDs, or deeply nested XPath.

A healed locator is a **candidate**, not a replacement. Promotion requires
repeated success plus Decision Engine approval (`01` §20, `08` §31).

**Final attempt** — the Decision Engine chooses stop, escalate, or continue.

Never retry assertion failures, logic failures, permission failures,
application exceptions, or security defects (`01` §21, `05` §30, `08` §13).

## Hard Limits

Per `08` §32. Exceeding any limit is a stop condition, not a reason to loop:

| Limit | Max |
|---|:---:|
| Healing attempts per test | 3 |
| Locator replacements | 2 |
| Wait adaptations | 1 |
| Browser restarts | 1 |

Healing also stops when confidence falls below 60 (`08` §20), when policy
rejects it, when validation fails, or when business intent would change
(`08` §32).

## Superseded Behaviour — Do Not Reintroduce

An earlier revision of this framework specified that Attempt 3 should
**"update the Page Object and retry the test once using the new locator."**

That is **prohibited**. It conflicted with `08` §31–§33 on three counts:

| Old behaviour | Why it is prohibited | Governing rule |
|---|---|---|
| Wrote the healed locator straight into the Page Object | A healed locator is an unproven candidate. Writing it in makes one lucky match permanent and silently rewrites a generated, traceability-stamped artifact mid-run. | `01` §20, `06` §40, `08` §31 |
| Healed and retried without approval | Retry and healing are Decision Engine decisions. An engine that heals itself is a second decision authority. | `02` §5.1, `08` §33 |
| Single unbounded heal-and-retry | Healing needs ranked candidates, confidence, policy validation, and post-apply verification — and it must stop at the §32 limits so defects stay visible. | `08` §19–§21, §29, §32 |

The correct flow: propose ranked candidates → `02` approves → apply in-session →
verify → audit → record the candidate in `10` for promotion **only** after
repeated success. The Page Object changes on a subsequent generation pass
(`06` §31 incremental regeneration), never as a runtime side effect.

---

# 9. Security Tooling — ZAP Tiers

Security testing is opt-in by design (`01` §3.5, `05` §22). Penetration testing
and production exploitation are explicit non-goals (`01` §2).

## Setup

The API key is a credential. It SHALL come from the environment, never a
literal in a config file, script, or report (`01` §30).

```bash
# once — provision
which zaproxy || docker image ls zaproxy/zap-stable
docker pull zaproxy/zap-stable

# per session — key from env, generated if absent
export ZAP_API_KEY="${ZAP_API_KEY:-$(openssl rand -hex 16)}"

docker run -d --name zap --dns 1.1.1.1 --dns 8.8.8.8 \
  -p 8080:8080 -p 8090:8090 zaproxy/zap-stable \
  zap.sh -daemon -host 0.0.0.0 -port 8080 \
  -config api.addrs.addr.name=.* \
  -config api.addrs.addr.regex=true \
  -config api.key="$ZAP_API_KEY"

# readiness — poll, never assume. Observed ~30s on first start.
until curl -sf "http://localhost:8080/JSON/core/view/version/?apikey=$ZAP_API_KEY" >/dev/null; do sleep 2; done
```

## Verify The Proxy End-To-End Before Trusting Tier 2

A running container with a responding API does **not** mean ZAP can reach the
target. Explicit DNS servers are mandatory: if the host resolver is a NAT-local
or loopback address — `10.0.2.3` under VirtualBox/QEMU, `127.0.0.53` under
systemd-resolved — the container inherits an address unreachable from Docker's
bridge network.

The failure is silent in the worst way. ZAP still accepts the proxied request
and registers the host, so `core/view/sites` looks correct, but every upstream
fetch returns **502** and the passive scanner sees nothing. A Tier 2 run then
reports zero findings and looks like a clean target.

Never report T2 as active without this check:

```bash
Z="http://localhost:8080"; K="apikey=$ZAP_API_KEY"

docker exec zap getent hosts example.com                        # DNS resolves?
curl -sk -o /dev/null -w '%{http_code}\n' -x "$Z" "$BASE_URL"   # must be 2xx/3xx, never 502
until [ "$(curl -sf "$Z/JSON/pscan/view/recordsToScan/?$K" | jq -r .recordsToScan)" = "0" ]; do sleep 2; done
curl -sf "$Z/JSON/core/view/numberOfAlerts/?$K"                 # must be > 0
```

A 502 through the proxy, or zero alerts after the queue drains, means T2 is
**not** working. Fall back to T1 and say so in the report (§16) rather than
publishing an unearned clean result.

Every later API call appends `?apikey=$ZAP_API_KEY`. The key SHALL be masked in
logs and SHALL NOT appear in any generated artifact or report.

If neither ZAP nor Docker is available, security testing falls back to T1 only,
and the report SHALL state that T2/T3 did not run and why (§16).

## Tier 1 — Passive Observation (always on)

Read-only checks in the suite itself. Zero risk.

- HTTPS enforcement
- Cookie flags: `Secure`, `HttpOnly`, `SameSite` on session cookies
- Headers: `X-Frame-Options`, `X-Content-Type-Options`,
  `Strict-Transport-Security`, `Referrer-Policy`, `Permissions-Policy`
- CSP presence and basic directive validation
- Mixed content on HTTPS pages
- `autocomplete="off"` on sensitive fields
- Information disclosure: server version headers, stack traces, debug endpoints
- TLS protocol version and certificate validity

## Tier 2 — ZAP Passive Proxy (on when ZAP is available)

Playwright traffic routes through ZAP. ZAP analyses requests the suite already
makes — no extra requests, no injection, zero risk.

```typescript
use: {
  proxy: { server: 'http://localhost:8080' },
  ignoreHTTPSErrors: true, // ZAP's intercepting certificate
}
```

Catches: session IDs in URLs, cookie flag gaps, missing anti-CSRF tokens on
state-changing forms, error-response disclosure, private IPs and internal
hostnames, framework version leakage, cache-control on authenticated pages,
cross-domain script inclusion, content-type mismatch, basic auth over HTTP.

## Tier 3 — ZAP Active Scan (off by default)

Requires `SECURITY_SCAN=active`.

- Scoped **only** to API endpoints observed during Discovery — never ZAP's own
  spider
- Injection vectors: SQLi, XSS, command injection, path traversal, SSRF
- Access control: forced browsing, IDOR, privilege escalation paths
- Business logic: mass assignment, rate-limit bypass, parameter tampering
- Triggered after execution completes

**Active scanning mutates state.** It submits forms, creates records, modifies
data. Run it only against a dedicated test environment, and only with explicit
written authorization. Never against production or shared staging.

Post-execution collection:

```bash
Z="http://localhost:8080"; K="apikey=$ZAP_API_KEY"

curl -sf "$Z/JSON/core/view/alerts/?$K"

# active scan only — poll to completion, never assume
until [ "$(curl -sf "$Z/JSON/ascan/view/status/?$K" | jq -r .status)" = "100" ]; do sleep 5; done
```

## Teardown

```bash
Z="http://localhost:8080"; K="apikey=$ZAP_API_KEY"

curl -sf "$Z/JSON/core/action/saveSession/?$K&name=execution-$EXECUTION_ID"
curl -sf "$Z/OTHER/core/other/htmlreport/?$K" > reports/security/zap-report.html
docker rm -f zap
```

Teardown SHALL run even when execution failed — an orphaned ZAP holds ports
8080/8090 and silently proxies the next run.

All ZAP artifacts live under `reports/security/`.

---

# 10. OWASP Top 10 Mapping (2021)

Every security finding is tagged with its category.

| ID | Category | T1 | T2 | T3 |
|---|---|:---:|:---:|:---:|
| A01 | Broken Access Control | — | ✅ | ✅ |
| A02 | Cryptographic Failures | ✅ | ✅ | ✅ |
| A03 | Injection | — | — | ✅ |
| A04 | Insecure Design | — | ✅ | ✅ |
| A05 | Security Misconfiguration | ✅ | ✅ | ✅ |
| A06 | Vulnerable Components | — | ✅ | — |
| A07 | Authentication Failures | — | ✅ | ✅ |
| A08 | Data Integrity Failures | — | ✅ | ✅ |
| A09 | Logging & Monitoring Gaps | ✅ | ✅ | — |
| A10 | SSRF | — | — | ✅ |

---

# 11. Performance Budgets

| Metric | Threshold |
|---|---|
| LCP | ≤ 2.5s |
| CLS | ≤ 0.1 |
| INP | ≤ 200ms |
| TTFB | ≤ 800ms |
| Performance Score | ≥ 50 |
| Accessibility Score | ≥ 70 |
| SEO Score | ≥ 50 |
| Best Practices | ≥ 50 |

Lighthouse SHALL run as plain `.mjs`/`.js` under Node, never through a
TypeScript transform — see §15.

---

# 12. Failure Artifact Checklist

Implements the Artifact Manager (`01` §22) against the artifact plan (`05` §31).

Screenshot · full-page screenshot · trace · video · console logs · network logs ·
HAR · DOM snapshot · HTML source · CSS snapshot · cookies · localStorage ·
sessionStorage · request · response · browser version · viewport · OS · timestamp

Every artifact links to Execution ID, Test ID, Workflow ID, and Correlation ID.
Sensitive values are masked before persistence (`07` §41).

---

# 13. Report Formats

Rendered from the normalized report model (`09` §15, §28) — never hand-written
twice.

- HTML — primary human-readable
- Markdown — tickets, Confluence, Slack
- JSON — machine-parseable, feeds dashboards
- JUnit XML — CI/CD native
- ZAP HTML — security findings
- Executive dashboard (`09` §25)

> **Optional**: Allure via `npm i -D allure-playwright` plus
> `['allure-playwright']` in the reporter array. Not default — it needs a Java
> runtime and its trending features duplicate `10`.

Content: passed, failed, skipped, duration, browser matrix, screenshots, videos,
traces, root-cause analysis, retry history, healing events, promoted locators,
coverage summary, OWASP-mapped security findings.

---

# 14. CI/CD Integration

```yaml
name: QA Automation
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: |
          npm ci
          npx playwright install --with-deps

      - name: Run QA Suite
        run: npx playwright test
        env:
          BASE_URL: ${{ secrets.TARGET_URL }}
          TEST_USER: ${{ secrets.TEST_USER }}
          TEST_PASS: ${{ secrets.TEST_PASS }}
          CI: true

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: |
            reports/
            test-results/
          retention-days: 30
```

## Exit Code Contract

- `0` — all tests passed
- `1` — at least one test failed

Never swallow exit codes. CI SHALL fail on test failures.

---

# 15. Hard-Won Rules

Every rule below was learned by getting it wrong on a real engagement. Each cost
debugging time or produced a wrong report. Apply them before writing code, not
after a failure.

## Verify locators before generating tests

Never write a spec against an assumed selector. Crawl first, then run a
throwaway probe that asserts every intended locator resolves **and returns the
expected count**, then generate. A probe costs minutes; a suite built on wrong
selectors costs hours.

Do not assume two pages share a structure because they look alike or share a
component. Verify per page.

> Observed: a cart table with id `#cart_info_table` on `/view_cart` had **no id
> at all** on `/checkout`, where it sat inside a `#cart_info` wrapper with no
> `<tbody>`. A page object reusing the id silently matched zero rows, and the
> assertion failed as "expected 2, received 0" — reading like an application bug.

## Count every locator, not just its existence

A locator that resolves is not a locator that resolves *uniquely*. Check counts
during discovery and scope accordingly.

- Repeated controls are common: hover overlays, sticky duplicates, mobile and
  desktop variants rendered together. One site exposed **68** `a.add-to-cart`
  nodes for **34** products.
- `getByPlaceholder` and `getByText` are **case-insensitive substring** matches
  by default. `getByPlaceholder('Email')` also matches "Your email address".
  Pass `{ exact: true }`, and scope to a container when the same exact text
  appears in two forms on one page.
- This matters doubly in self-healing candidate chains: a fallback matching
  three nodes heals into a strict-mode violation. Every candidate must be
  unambiguous on its own.

## Never swallow a `page.evaluate` rejection

`.catch(() => [])` around an extraction converts a hard error into a plausible
empty result. Route extractions through a helper that logs and records the
failure, and surface a count of extraction errors at the end of discovery.

> Observed: a discovery crawler reported 0 components and 0 forms across all 17
> routes while printing a successful-looking summary line for every page.

## No named functions inside `page.evaluate`

Tooling that applies esbuild's `keepNames` transform — `tsx`, `ts-node` with
esbuild, most bundlers — rewrites a function assigned to a named binding into
`__name(fn, "fn")`. That helper does not exist in the browser context, so the
callback throws `__name is not defined`.

Keep evaluate bodies free of named inner functions; pass data in as an argument
instead of declaring a helper inside the callback.

The same transform corrupts **third-party** injected bundles. Tools that inject
their own code into the page — Lighthouse is the common case — must run as plain
`.mjs`/`.js` under node, never through a TypeScript transform.

## Standalone scripts do not read the Playwright config

`use.testIdAttribute` applies only inside the test runner. A discovery or probe
script must call `selectors.setTestIdAttribute(...)` itself, or use explicit
attribute selectors. Detect the project's real test-id attribute during
discovery — `data-qa`, `data-test`, `data-cy` are all common — and never assume
`data-testid`.

## Never pass `--reporter` on the CLI

It **replaces** the entire reporter array from the config. html, json, junit,
allure and any custom reporter silently stop writing, leaving only the artifacts
directory. Use the config's reporters and include `list` there.

## Never run two `playwright test` invocations concurrently

Playwright clears `outputDir` when a run starts, so a second invocation destroys
the first run's traces, videos and screenshots mid-flight. Run multiple projects
in one invocation (`--project=a --project=b`) or give each run a distinct
`outputDir`.

## Aggregate reporter results per test, never per attempt

`onTestEnd` fires once per **attempt**, and `test.outcome()` is not settled until
every retry has run — during a flaky test's first attempt it still reads
`unexpected`. Counting inside the hook tallies attempts, not tests.

Collect into a map keyed by `test.id` and aggregate in `onEnd`. Always reconcile
the custom reporter's totals against the runner's own summary line before
publishing them.

> Observed: a reporter announced **33 failures** when **4** tests had failed — an
> 8x overstatement in the headline number of the deliverable.

## Detect the target's own failure pages

Applications behind request queues, WAFs and rate limiters serve interstitials
that are **not** the application: "under heavy load", "queue full", "checking
your browser", CAPTCHA challenges, maintenance pages. Every locator then times
out against markup that never contained it, and the failures read exactly like
application defects.

Build a health check that:

1. Recognises the interstitial and throws a **named, self-describing** error.
2. Runs on every navigation **and** on the final DOM of any failed test — the
   interstitial can arrive after a form POST, where no navigation helper ran.
3. Is classified by the reporter as an environment issue, ordered **ahead of
   every other rule**, so it can never be scored as a regression.

Then reduce concurrency. Against a shared or public target, default to
`workers: 1` and raise it only for a private deployment.

> Observed: 4 workers produced **59 spurious failures** in one run, all initially
> indistinguishable from real defects.

## Never build assertions on truncated command output

Parse the whole payload and aggregate it — count entries, count distinct values,
diff sets — before deciding what the contract is.

> Observed: reading only the first 200 bytes of a brands endpoint suggested one
> duplicate brand. The full payload had **34 entries for 8 brands**, keyed by
> product id — a far larger defect than the truncated view implied, and it
> produced two wrong assertions that had to be corrected after they failed.

## Assert observed behaviour; report the deviation

When the application genuinely misbehaves, a permanently red suite is worthless.
Assert what the application **actually does**, state the target behaviour in the
assertion message, and raise the gap as a finding. Write the assertion so it
**inverts cleanly** once the defect is fixed.

For accessibility and similar rule engines, keep an explicit baseline of known
violations plus a test that each baselined violation **still exists** — so a fix
surfaces as a failure telling you to delete the entry. Verify each baselined
violation **on the page it occurs on**; a single-page check reports defects on
other pages as fixed.

Gate on new violations only. Never suppress by lowering the rule set.

## Prefer the authoritative tool over a hand-rolled equivalent

If a standards engine already covers a check, do not also hand-roll it. Two
mechanisms on one target will disagree, and the hand-rolled one is usually the
weaker.

> Observed: a hand-rolled accessible-name check accepted `placeholder` as a
> label, which axe correctly rejects. Keeping both meant one finding reported
> twice by two mechanisms that disagreed on the standard.

## Visual baselines: never `fullPage` on a height-unstable page

Asynchronous content — recommendation carousels, ad slots, lazy images — changes
document height between runs, and a height mismatch fails **before any pixel is
compared**, so masking cannot save it.

For such pages capture a fixed-height viewport region, or assert on specific
component containers. Reserve `fullPage` for pages whose height is genuinely
stable, and mask volatile regions there.

## De-duplicate before delivering

Writing many spec files independently produces accidental repetition. Audit
before reporting, and classify each case:

| Case | Action |
| --- | --- |
| Identical assertion, same technique | Delete one. Keep the superset. |
| Same target, different technique, second adds no signal or is weaker | Delete the weaker. Document the removal in place so it is not reintroduced. |
| Same target, genuinely different scope or surface | Keep. State why in a comment. |
| Same assertion across browser projects | Keep — that is the point of the matrix. |
| Smoke overlapping functional | Keep — smoke is a cheap pre-gate. State that it is deliberate. |

Then report the audit: what was removed, what was kept, and the justification
for each keep. A reader must be able to tell deliberate overlap from accidental
repetition.

**Pure-function tests belong in a browser-free project.** A unit test placed in
a browser spec is re-executed once per browser project for no added signal. Give
them their own project and exclude that path from the browser projects.

## Report duplication is worse than test duplication

Never maintain the same findings or coverage content in two hand-written
formats; they drift at the first edit. Pick one hand-written document as the
source of truth and **generate** everything else from run output.

## Distinguish "cases run" from "behaviours verified"

A headline test count includes cross-browser repeats and deliberate smoke
overlap. Label it "cases run". Never present it as a count of distinct verified
behaviours.

## Re-running is not coverage

Re-executing a suite to confirm a fix is verification, not additional coverage,
and must never be reported as such. Report totals from a **single complete
run**, never accumulated across runs.

## Budget for the target, not the suite

A rate-limited target forces serial execution, which sets the real wall-clock
cost. Measure one full pass early and set `globalTimeout` from that measurement
plus headroom. State the duration in the report, and if a run is truncated, say
exactly how many tests did not execute rather than presenting partial results as
complete.

## Test case ids must be PREFIX-unique

`--grep` is a regex, not an exact match. An id that is a prefix of another id
silently runs both tests.

> Observed: `-g "TC-0039"` ran **TC-0039 and TC-0039b**. A reviewer asking for one
> test got two, from two different concerns. Never suffix an id with a letter to
> squeeze in a late addition — allocate a fresh number.

Assert prefix-uniqueness in validation, not just uniqueness:

```
for a in ids: for b in ids: if a != b and b.startswith(a): FAIL
```

Anchor the pattern when you cannot change the ids: `-g "TC-0039 "` (trailing
space) or `-g "\bTC-0039\b"`.

## Never assign the same id twice from two authors

Ids written once in the plan and again by hand in the specs WILL drift.

> Observed: 44 of 79 tagged tests carried an id that the plan assigned to a
> different test, and one id landed on two unrelated tests. The ids were
> decorative, which defeats their only purpose.

Derive spec titles from the plan catalogue, or generate both from one source. If
drift already exists, build a traceability matrix from the SPEC FILES — the
executable truth — and match back to the plan **by name**, not by id.

## `.count()` never auto-waits

`expect(await locator.count())` is a snapshot taken immediately. `toHaveCount`
retries. On any async-rendered list the snapshot reads 0 and the failure looks
exactly like an application defect.

> Observed: four tests failed counting table rows immediately after navigation.
> Rows arrived via XHR at a measured 351ms and 902ms. All four were suite defects.

Add a generation rule and a validation check: `await expect(l).toHaveCount(n)` is
permitted; `expect(await l.count())` is not.

## `click()` implicitly waits for navigation, bounded by `actionTimeout`

A click that schedules navigation waits for it — and that wait uses
`actionTimeout`, **not** any longer budget you wrote afterwards.

> Observed: a login page object set `actionTimeout: 20_000` globally and then a
> 45s `Promise.race` for the outcome. The click always died at 20s with
> "waiting for scheduled navigations to finish", so the 45s budget was
> unreachable. The generous wait was decorative.

Give the click its own explicit timeout matching the intended tolerance:
`click({ timeout: 60_000 })`.

## A shell env script must be sourced, and must say so

`./env.sh` runs in a subshell; every `export` and `unset` is discarded. The user
gets no error and no environment.

> Observed: an operator ran `./env.sh`, saw no output, and then hit
> `Available projects: ""` from a different directory — two failures compounding
> one silent no-op.

Guard it, resolve paths from the script's own location so it works from any
directory, and print a confirmation:

```bash
if [ "${BASH_SOURCE[0]:-$0}" = "$0" ] && [ -z "${ZSH_EVAL_CONTEXT:-}" ]; then
  echo "ERROR: source this file, do not execute it" >&2; exit 1
fi
QA_PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)"
```

---

# 16. Reporting Integrity

Non-negotiable when producing the final report. Implements `09` §5.1 and
`01` §31.

- **Reconcile every number** against the runner's own output before publishing.
- **Separate defects in the target from defects in the generated suite.** Both
  belong in the report, labelled, never conflated.
- **Separate environment failures from application failures.** A rate-limited or
  unreachable target is not a defect.
- **Name the test that pins each finding**, so remediation flips a specific
  assertion.
- **State what did not run and why** — unsupported browser engines, truncated
  runs, skipped suites, capabilities absent from the application.
- **Include evidence**: the command, the response, the measured numbers — not a
  summary of them.
- **Report your own mistakes** in the tooling and the suite. A report that hides
  them is not trustworthy about the rest.

---

# 17. Manual Observation Mode

Watching a suite drive the application is a legitimate engineering need —
reviewing a flow with a stakeholder, diagnosing something a trace does not
explain, demonstrating a defect. The default fixture design actively fights it.

## The problem

A correct fixture creates a **fresh browser context per test**, because tests must
be independent and parallel-safe (`01` §33). In headed mode every context is a new
OS window, so the run flickers: window opens, closes, next opens.

> Observed: a 2-test headed run opened **three** windows — one for the
> worker-scoped login fixture, then one per test, each closing before the next
> appeared. The operator could not follow what the suite was doing, and the
> default action speed made it unwatchable regardless.

Verify the fix by MEASURING, not by assuming. Log
`context.browser()?.contexts().length` on every reuse; it must stay at 1. An
assertion that "it should now be one window" is not evidence — the first attempt
at this fix looked correct in code and was still broken for the operator.

## Drive it from HEADEDNESS, not from an env var

The first version of this pattern gated single-window behind `MANUAL=1`. That was
wrong, and it failed in the field: the operator ran `--headed`, got the old
three-window flicker, and reported the fix as not working. They were right — an
observability fix the operator has to know a magic variable to reach is not a fix.

**Headedness IS the signal.** Nobody runs headed except to watch. Resolve it per
worker from the project's effective `use.headless`, which already reflects the
`--headed` CLI override:

```typescript
function isSingleWindow(headless: boolean | undefined): boolean {
  return process.env.MANUAL === '1' || headless === false;
}

sharedAuthContext: [async ({ browser }, use, workerInfo) => {
  if (!isSingleWindow(workerInfo.project.use.headless)) { await use(null); return; }
  /* …create ONE context, log in visibly, keep it open… */
}, { scope: 'worker' }],
```

Keep `MANUAL=1` as a convenience switch that additionally forces headed, sets a
human pace and disables retries — but never as the only way in.

## Both fixture families need sharing

Authenticated and unauthenticated tests cannot share a context: a session-free
context is a hard requirement of the clean tests. But each family can share ONE
window with itself. Clear cookies on reuse so clean tests still start session-free.

Window accounting in single-window mode: **at most 2** for a whole run (one
authenticated, one clean), and exactly **1** when the run uses a single family.

## The pattern

Make it implicit and complete, so the operator sets nothing:

```typescript
const MANUAL = process.env.MANUAL === '1';
const SLOWMO = Number(process.env.SLOWMO ?? (MANUAL ? 350 : 0));

use: { headless: !MANUAL, launchOptions: { slowMo: SLOWMO } },
retries: MANUAL ? 0 : (process.env.CI ? 2 : 1),
```

`retries` MUST be forced to 0. Otherwise the operator watches a silent rerun
without realising the first attempt failed.

One worker-scoped context, logged in **visibly**, reused by every test:

```typescript
manualContext: [async ({ browser }, use) => {
  if (!MANUAL) { await use(null); return; }
  const context = await browser.newContext({ /* NO storageState */ });
  await performLogin(await context.newPage());   // the operator watches this
  await use(context);
  await context.close();
}, { scope: 'worker' }],

authenticatedPage: async ({ browser, sessionState, manualContext }, use) => {
  if (manualContext) {
    // Reuse the existing PAGE, not just the context — a new page is a new tab
    // and the window still blinks between tests.
    await use(manualContext.pages()[0] ?? await manualContext.newPage());
    return;                                       // deliberately not closed
  }
  const context = await browser.newContext({ storageState: JSON.parse(sessionState) });
  const page = await context.newPage();
  await use(page);
  await context.close();
},
```

## Invert the session dependency

`manualContext` SHALL NOT depend on the login-state fixture. Playwright resolves
destructured fixtures eagerly, so depending on it builds the throwaway login
context first — reintroducing the very window this mode exists to remove.

Invert it: the shared context logs itself in, and the state fixture harvests from
it.

```typescript
sessionState: [async ({ browser, manualContext }, use) => {
  if (manualContext) { await use(JSON.stringify(await manualContext.storageState())); return; }
  /* …normal throwaway-context login… */
}, { scope: 'worker' }],
```

No cycle: `manualContext` does not depend on `sessionState`.

## State the trade-off in the code

Reusing one context means the tests are **no longer independent**. State bleeds
between them. This is a real loss and must be documented where a reader will hit
it, not buried in a README.

- Use it to WATCH a handful of tests.
- A passing `MANUAL` run is **not** evidence the suite passes in isolation.
- Reported results SHALL come from a normal run.
- Tests that destroy the shared session (logout) or require a session-free
  context will misbehave — do not run the full suite this way.

---

# 18. Target Health Pre-Flight

Implements the `07` §40 "Environment Verified" gate, and closes a
misclassification the reporter cannot otherwise avoid.

## The problem

Failure classification keys on markers: interstitial text, gateway status codes.
A target that is merely **slow**, or that returns bare 500s and drops TLS
connections, emits none of them — so its timeouts classify as TIMING and get
reported as candidate **defects**.

> Observed on one shared demo instance, minutes apart:
> `GET /auth/login` → HTTP 500 in 0.5s; → HTTP 200 in **26.7s**; → TLS connection
> dropped. A test failing against that is an environment artifact, but the report
> called it a candidate defect.

## The pattern

Probe the target in `globalSetup`, write a verdict, and let the reporter consult
it. Compare against a latency reference measured during Discovery, not a guess.

```typescript
// HEALTHY | DEGRADED (>=5x reference) | UNHEALTHY (any 5xx or connection failure)
const health = await probeTargetHealth(BASE_URL);   // 3 probes
fs.writeFileSync('reports/target-health.json', JSON.stringify(health));
```

Then, in the reporter, reclassify **narrowly**:

```typescript
if (targetUnhealthy && /waiting for scheduled navigations|timeout .*exceeded|navigation/.test(text))
  return 'ENVIRONMENT';
```

Narrowly matters. An ASSERTION failure stays an assertion failure on a sick
target — a wrong expectation is still wrong.

## Rules

- Do NOT abort by default. A deliberate run against a degraded target is
  sometimes exactly what is wanted. Gate it behind `ABORT_IF_UNHEALTHY=1`.
- Print the verdict where the operator will read it, before any test output.
- Carry the verdict into the report, so a reader knows whether the numbers mean
  anything.
- A run against an UNHEALTHY target SHALL NOT be used for reported totals.

---

# 19. Evidence Quality Measurement

Implements the requirement already specified by `02_Decision_Engine.md` §18
(*"Every evidence item receives its own confidence score"*) and §21, whose
Confidence Formula weights `Evidence Quality × 0.35`.

## Status Of This Section

This section adds **no model**. `02` §21's Confidence Formula is the framework's
single confidence model and is used **unchanged**.

`02` specifies **what** Evidence Quality is — a per-item score, aggregated into
the 0.35 term of its formula. `02` does not state **how** to compute it. That
mechanical procedure is supplied here, and here only.

| Concern | Owner |
| ------- | ----- |
| Evidence Quality — definition, weight, role in confidence | **02** §18, §21 |
| Framework Confidence — the model and its output | **02** §21 |
| Measurement procedure — the mechanical HOW | This section |

`02` remains the sole owner. Nothing in this section may override, reweight, or
substitute for `02` §21.

Evidence Quality is **not** an engine-local score. It is an input term of `02`'s
own model, so the Ownership Matrix rule requiring engine-local scores be *"named
distinctly from confidence"* does not apply and no second score is created.

---

## Why This Section Exists

`02` §18 has mandated per-item evidence confidence since Architecture v3.0.
Across three executions — `qa-automationexercise`, `qa-orangehrm-20260803`,
`qa-orangehrm-20260805` — it was measured **zero** times, and no Framework
Confidence value was emitted in any run.

A complete specification with no tactics entry produced no implementation three
times. This section closes that gap.

---

## Per-Item Evidence Confidence

Every Evidence Object defined by `02` §18 carries the fields
`id · type · source · confidence · timestamp · payload`.

The `confidence` field SHALL be populated by three mechanical checks over
attributes `02` §18 already mandates. No attribute is invented.

### Check 1 — Completeness

Each of the six required fields is present and non-empty.

```
completeness = (populated required fields) / 6        → 0.0 … 1.0
```

### Check 2 — Directness

Derived from the evidence category `02` §18 already defines:

| `02` §18 category | Directness | Rationale |
| ----------------- | ---------: | --------- |
| **Dynamic** — current DOM, network latency, CPU, retry count, environment health | 1.00 | Measured this execution |
| **Static** — routes, configuration, capabilities, browser matrix | 0.90 | Measured, but not re-observed under load |
| **Historical** — flaky tests, past failures, locator evolution, healing success, execution trends | 0.75 | Inferred from prior executions; correct at the time observed |

### Check 3 — Corroboration

Count of **distinct `source` values** asserting the same fact:

| Distinct sources | Multiplier |
| ---------------: | ---------: |
| 0 | 1.00 |
| 1 | 1.00 |
| 2 | 1.05 |
| ≥ 3 | 1.10 |

**Zero distinct sources yields 1.00, the same as one source.**

Absence of corroboration is already represented through the completeness factor
(Check 1), which counts an absent `source` field as unpopulated. Applying a
second reduction here would penalise the same absence twice.

The multiplier therefore rewards corroboration above a single source; it never
penalises its absence.

### Item Score

```
itemConfidence = round( 100 × completeness × directness × corroboration )
clamped to 0 … 100
```

The clamp is required: corroboration may lift a complete, dynamic item above 100
before clamping.

---

## Evidence Quality — The §21 Term

`02` §21 lists **Evidence Quality** and **Evidence Quantity** as *separate*
factors. Evidence Quality SHALL therefore be **quantity-neutral** — a mean, never
a sum. A large pile of weak evidence must not present as high quality.

```
evidenceQuality = arithmetic mean of itemConfidence over the evidence SET
```

Aggregation is over the set, **not** a sequence. It SHALL be order-independent:
identical evidence gathered in a different order SHALL produce an identical
value. This matches the determinism discipline already required by `14` §16.

The result is expressed on `02` §21's existing 0–100 scale. **No new scale is
introduced.**

---

## Empty Or Insufficient Evidence

Where the evidence set is empty, `evidenceQuality` is **undefined** — never
zero, and never defaulted.

Per `02` §5.5 Safe Failure, the decision SHALL be refused with
`INSUFFICIENT_EVIDENCE` rather than computed with a substituted term.

A defaulted 0.35 term would silently fabricate confidence. An undefined term
makes the gap visible, which is the behaviour `01` §31 requires.

---

## Feeding Framework Confidence

The measured value is supplied to `02` §21 as the `Evidence Quality` term of its
existing formula:

```
Confidence = Evidence Quality        × 0.35
           + Historical Reliability  × 0.25
           + Rule Agreement          × 0.20
           + Environment Stability   × 0.20
```

This formula SHALL NOT be altered, reweighted, or reimplemented here. The
remaining three terms are supplied by `02` from its own sources.

`02` emits the resulting Framework Confidence. It is the only value that may be
reported as "confidence" (Ownership Matrix, *Confidence Ownership*).

---

## Persistence — No New Artifact

| Value | Recorded in | Defined by |
| ----- | ----------- | ---------- |
| `itemConfidence` | the `confidence` field of the Evidence Object | `02` §18 |
| `evidenceQuality` | the decision's evidence record | `02` §36 |
| Framework Confidence | the decision audit record | `02` §39 |

All three destinations already exist. **No dataset, file, or schema is created.**

---

## Determinism Requirements

The measurement SHALL:

- Contain no randomness
- Contain no wall-clock dependence — `timestamp` is compared for ordering and
  recency classification, never used as a magnitude
- Aggregate order-independently
- Produce an identical value for an identical evidence set
- Round once, at the item level, using a single fixed rule

Two executions with identical evidence SHALL produce identical `evidenceQuality`
and identical Framework Confidence.

---

## Verification

A future execution satisfies this section when:

- Every Evidence Object carries a populated, non-default `confidence`
- `evidenceQuality` appears in the decision's evidence record
- Framework Confidence is emitted, and reproduces `02` §21's formula from its
  four stated terms
- Two runs over identical evidence produce identical values
- An empty evidence set yields `INSUFFICIENT_EVIDENCE`, not a zero

> **Demonstrated in RW0** (`EXEC-20260810-0001`, 2026-08-10, authorized OrangeHRM
> target). Thirty-three Evidence Objects across six decision records each carried a
> measured `confidence`; `evidenceQuality` was emitted on all six; the 0–100 clamp
> fired on a three-source dynamic item; and every value recomputed identically from
> its persisted inputs, including under reversed evidence order. The empty-evidence
> decision path did not arise naturally and was not manufactured.

---

# 20. Framework Confidence — Remaining §21 Term Measurement

## Status Of This Section

This section adds **no model**. `02` §21's Confidence Formula remains the
framework's single confidence model, used **unchanged**:

```
Confidence = Evidence Quality        × 0.35
           + Historical Reliability  × 0.25
           + Rule Agreement          × 0.20
           + Environment Stability   × 0.20
```

The four weights are `02`'s and are **not** restated as adjustable here. They sum
to 1.00. Nothing in this section may reweight, replace, duplicate, or relocate
them, and `02` remains the sole Framework Confidence authority.

§19 supplied the mechanical procedure for **one** of the four terms. `02` names
the other three as factors and, exactly as it did for Evidence Quality, does not
state **how** to compute them. This section supplies those three procedures, and
here only.

| Concern | Owner |
| ------- | ----- |
| The confidence model, its weights, its output | **02** §21 |
| Term definitions and their role in confidence | **02** §18, §21 |
| Evidence Quality measurement procedure | **§19** (this document) |
| Remaining three measurement procedures | **§20** (this section) |

None of the three is an engine-local score. Each is an **input term of `02`'s own
model**, so the Ownership Matrix rule requiring engine-local scores be *"named
distinctly from confidence"* does not apply, and no second score is created.

---

## Why This Section Exists

RW0 (`EXEC-20260810-0001`) demonstrated the full runtime chain from an authorized
target through Evidence Objects and per-item confidence to `evidenceQuality` and
`02` §21's unchanged weights. **Framework Confidence was still not emitted**,
because three of the four required terms had no approved measurement procedure.
The runtime refused to produce a value rather than substitute one.

> That refusal was correct behaviour, not an implementation failure. `02` §5.5 and
> `01` §31 require an absent term to stay visible. A defaulted term would have
> fabricated confidence — the precise failure §19 was written to prevent.

This section closes the remaining gap so the model becomes computable without any
value ever being guessed.

---

## Common Rules For All Three Terms

These rules are identical to §19's and are restated, not redefined.

**Scale.** Each term is expressed on `02` §21's existing 0–100 scale. **No new
scale is introduced.**

**Rounding.** Each term rounds **once**, at the term level, half-up, then clamps
to 0–100. There is no second rounding anywhere.

**Determinism.** No randomness. No hidden state. No undocumented heuristic.
Aggregation is order-independent. Each term is independently recomputable from its
declared persisted inputs.

**Wall-clock rule.** A `timestamp` MAY be used for **ordering** and **recency
classification**; it SHALL NEVER be used as a **magnitude**. A measured elapsed
**duration** is an observation, not a clock reading, and may be used as a
magnitude. This is §19's rule, unchanged.

**Unavailability.** Each term has an explicit `UNAVAILABLE` state with a named
reason. A term that is unavailable SHALL NOT be substituted with `0`, `50`, `100`,
a configured default, a value inferred from an unrelated artifact, or silence.

**Composite behaviour.** Framework Confidence is computed only when **all four**
terms are available. Where any term is unavailable, `02` refuses the value and
names every unavailable term, consistent with `02` §5.5. This is the behaviour RW0
already exercised; it is unchanged.

**Persistence.** All three term values are recorded in the **decision audit
record** (`02` §39), which already mandates `Confidence` and `Rules evaluated`.
**No dataset, file, or schema is created by this section.**

**Producer.** `02` §21 computes all four terms. Other documents supply *inputs*
through their existing read contracts. No engine produces a confidence-like score
of its own, and no measurement is relocated outside the confidence owner.

---

## 20.1 Historical Reliability — The 0.25 Term

### Definition

The rate at which **prior executions of the same application** completed their
lifecycle reliably.

### What It Deliberately Does NOT Measure

It does **not** measure how many tests passed.

A test failure is a finding about the **target**, not about framework
reliability. Scoring pass rate here would lower Framework Confidence precisely
when the framework succeeds at finding real defects, which inverts the meaning of
confidence. Reliability is therefore measured against the **execution lifecycle**,
not against test outcomes.

### Input Evidence

| Input | Source | Owner |
| ----- | ------ | ----- |
| Prior execution records | `executions/` in the Learning Repository, `10` §13 | **10** |
| Partition key | Execution Scope Identity | **01** §30.1 |
| Snapshot semantics | Start-of-execution pinned snapshot, `10` §7.5 | **10** |
| Terminal lifecycle state vocabulary | `01` §17 state machine | **01** |

Canonical dataset: **Learning Database (`10`)**. Consumed through `10`'s existing
read contract; this section defines no new record type.

### Eligible Records

A record is eligible when **all** of the following hold:

- it belongs to the **current Execution Scope Identity** (`10` §7.2);
- it is present in the **pinned snapshot** taken at execution start (`10` §7.5);
- it is **not** the current execution;
- its terminal lifecycle state is `COMPLETED` **or** `FAILED`.

Excluded, with reasons:

| Excluded | Reason |
| -------- | ------ |
| `CANCELLED` executions | Operator cancellation carries no reliability signal. Counting it as unreliable would penalise a deliberate stop. |
| Foreign-scope records | `10` §7.2 — foreign-scope reads are *evidence*, never native history. They SHALL NOT contribute to this term. |
| Records outside the pinned snapshot | `10` §7.5 — an execution SHALL NEVER read history it is concurrently writing. |

### Reliable Outcome

| Terminal state | Classification |
| -------------- | -------------- |
| `COMPLETED` | reliable |
| `FAILED` | unreliable |

### Recency Window And Decay

Decay is **rank-based, never time-weighted**. Time-weighted decay would use a
timestamp as a magnitude, which the Common Rules forbid.

```
order  = eligible records sorted by Start Time DESC,
         ties broken by Execution ID DESC          → a total order
window = the first W records of that order
```

`W` is the configured recency window, resolved through the configuration hierarchy
owned by `01` §29. Older executions lose influence by **falling out of the
window**, not by being multiplied by a decaying factor.

The tie-break makes the order **total**, so the window is identical regardless of
the order in which records were enumerated.

### Calculation

```
reliable              = |{ r in window : terminalState(r) == COMPLETED }|
historicalReliability = round( 100 × reliable / |window| )
                        clamped to 0 … 100
```

### Minimum Sample

`minimumExecutions`, resolved through `01` §29. This reuses the configuration
semantics `14` §30 already established for *"insufficient history (below
configured minimum executions)"*; it introduces no new threshold and no new
configuration authority.

### Empty Or Insufficient Input

| Condition | Result |
| --------- | ------ |
| No store for this scope (cold start, `10` §7.3) | `UNAVAILABLE: NO_HISTORY` |
| `|eligible|` = 0 | `UNAVAILABLE: NO_HISTORY` |
| `0 < |eligible| < minimumExecutions` | `UNAVAILABLE: INSUFFICIENT_HISTORY` |

A cold start is **not an error** (`10` §7.3) and SHALL NOT halt execution. It
makes this term unavailable, which refuses Framework Confidence for that decision
while the decision itself still proceeds on rules, policy and risk.

### Range And Boundaries

| Case | Value |
| ---- | ----- |
| Every windowed record `COMPLETED` | `100` |
| No windowed record `COMPLETED` | `0` |
| `|window|` = `minimumExecutions` exactly | measurable |
| `|window|` = `minimumExecutions − 1` | `UNAVAILABLE: INSUFFICIENT_HISTORY` |

### Determinism

Deterministic and order-independent: inputs come from an immutable pinned
snapshot, a total order fixes the window, and the aggregation is a ratio over a
set. `Start Time` is used only for ordering — never as a magnitude.

### Persistence

Value → decision audit record (`02` §39). Inputs remain in `10`'s Learning
Database. Owner: **02** §21.

---

## 20.2 Rule Agreement — The 0.20 Term

### Definition

The proportion of **applicable** rules evaluated for this decision whose `Action`
is consistent with the **selected** candidate.

### Input Evidence

| Input | Source | Owner |
| ----- | ------ | ----- |
| Rules evaluated | `02` §39 audit record — already a mandated field | **02** |
| Rule fields (`Rule ID`, `Action`, `Version`) | `02` §19 Rule Format | **02** |
| Selected candidate | `02` §39 audit record, *Winning candidate* | **02** |

Canonical dataset: **none new**. Every input is already required to be present in
the same decision audit record that will hold the result. `02` §39 mandates
*"Rules evaluated"*, so the observation exists today and no engine needs to begin
recording anything new.

### Applicability

A rule is **applicable** when its `Condition` was evaluated for this decision and
yielded a determinate outcome. Excluded from both numerator and denominator:

| Excluded | Reason |
| -------- | ------ |
| Rule not evaluated | It expressed no position on this decision. |
| `Condition` indeterminate | An indeterminate condition is not agreement. |
| `Action` orthogonal to every candidate | The rule neither supports nor contradicts the selection. |

Orthogonal exclusions SHALL be disclosed in the audit record, so a small
denominator is visible rather than hidden.

### Agreement And Disagreement

| Relationship between rule `Action` and the selected candidate | Classification |
| ------------------------------------------------------------- | -------------- |
| The selected candidate **satisfies** the `Action` | agrees |
| The selected candidate **contradicts** the `Action` | disagrees |
| Neither | orthogonal — excluded |

### Conflicting Rules

A conflict between two rules is **not resolved here**. Both are counted: one
agrees, one disagrees, and the value falls accordingly — which is the correct
signal, since a conflicted rule set is genuinely weaker evidence. Conflict
*resolution* remains `02`'s own capability and is never performed by this
measurement.

### `Confidence Modifier` Is Deliberately Unused

`02` §19's Rule Format includes a `Confidence Modifier`. This measurement
**SHALL NOT** read it. Doing so would give a rule a second, direct path into the
confidence value, producing an unauditable blend of a measured term and a
rule-authored adjustment. Rule agreement is measured from `Action` alone.

### Calculation

```
denominator   = |agreeing| + |disagreeing|
ruleAgreement = round( 100 × |agreeing| / denominator )
                clamped to 0 … 100
```

### Empty Or Insufficient Input

| Condition | Result |
| --------- | ------ |
| `denominator` = 0 (no applicable rules) | `UNAVAILABLE: NO_APPLICABLE_RULES` |
| *Rules evaluated* absent from the audit record | `UNAVAILABLE: RULE_OUTCOMES_NOT_RECORDED` |

**An empty rule set is not unanimous agreement.** "No rule objected" is the
absence of evidence, not perfect evidence, and SHALL NOT yield `100`.

### Range And Boundaries

| Case | Value |
| ---- | ----- |
| All applicable rules agree | `100` |
| All applicable rules disagree | `0` |
| One applicable rule, agreeing | `100` — legitimate, and the small denominator is disclosed |
| No applicable rules | `UNAVAILABLE: NO_APPLICABLE_RULES` |

### Determinism

Deterministic and order-independent: a count over a set, with no dependence on
evaluation order. `Rule Version` is recorded per rule, so a recomputation resolves
the same rule semantics that were evaluated.

### Persistence

Value → decision audit record (`02` §39), alongside the rule outcomes it was
computed from. Owner: **02** §21.

---

## 20.3 Environment Stability — The 0.20 Term

### Definition

The degree to which the target environment behaved **nominally** during the
pre-execution verification window.

### Input Evidence

| Input | Source | Owner |
| ----- | ------ | ----- |
| Environment verification (Health, Availability) | `07` §17, §31 Environment Manager | **07** |
| Pre-flight probe series | §18 Target Health Pre-Flight | this document |
| Discovery latency reference | measured during Phase 1, consumed by §18 | **03** |
| Persistence of execution metrics | `07` §37, §39 | **07** |

Canonical dataset: **Execution State (`07`)**, with the reported view carried into
**Reports (`09`)** as §18 already requires. No dataset is created.

Required observations, all already produced by §18:

```
totalProbes    errorProbes    medianMs    referenceMs
```

`errorProbes` counts probes returning a `5xx` status **or** failing to connect —
§18's existing `UNHEALTHY` criteria, unchanged.

### Window Semantics

The **pre-execution verification window only**: the probe series taken in
`globalSetup`, before any test runs.

Continuous during-execution health (`07` §40) is deliberately **excluded**.
Including it would let the outcome being scored feed back into the term that helps
score it, which would make Framework Confidence depend on its own subject.

### Stability And Instability Criteria

Both anchors below are **pre-existing approved §18 values**, restated:

| Criterion | Boundary | Source |
| --------- | -------- | ------ |
| Nominal | `medianMs` at the measured Discovery reference | §18 |
| Degraded | `medianMs` ≥ 5 × `referenceMs` | §18 |
| Unhealthy | any `5xx` or connection failure | §18 |

### Calculation

```
availability  = (totalProbes − errorProbes) / totalProbes
latencyRatio  = medianMs / referenceMs
latencyFactor = clamp( (5 − latencyRatio) / (5 − 1), 0, 1 )

environmentStability = round( 100 × availability × latencyFactor )
                       clamped to 0 … 100
```

`latencyFactor` interpolates linearly between §18's two existing anchors: `1.00`
at the measured reference latency, `0.00` at §18's `DEGRADED` threshold of 5×. A
target faster than its reference clamps to `1.00` — faster than nominal is not
better than nominal.

> **The linear shape between those two anchors is the one genuinely new numeric
> choice in this section.** It is declared here rather than buried: the anchors are
> pre-approved, the interpolation between them is this decision's.

### Transient Failures

A single failing probe reduces `availability` proportionally. It is neither
amplified into an automatic `UNAVAILABLE` nor ignored. A transient failure lowers
the term; it does not erase it.

### Empty Or Insufficient Input

| Condition | Result |
| --------- | ------ |
| `totalProbes` = 0 | `UNAVAILABLE: NO_ENVIRONMENT_OBSERVATION` |
| `referenceMs` absent or 0 | `UNAVAILABLE: NO_LATENCY_REFERENCE` |

§18 already forbids substituting a guessed reference; a missing reference
therefore makes this term unavailable rather than estimated.

### Range And Boundaries

| Case | Value |
| ---- | ----- |
| All probes non-error, `medianMs` ≤ `referenceMs` | `100` |
| All probes error | `0` |
| `medianMs` ≥ 5 × `referenceMs` | `0` |
| `medianMs` = 3 × `referenceMs`, no errors | `round(100 × 1.00 × 0.50)` = `50` |
| No probes | `UNAVAILABLE: NO_ENVIRONMENT_OBSERVATION` |

### Determinism

Deterministic and order-independent: a fixed probe count, a median over a set, and
arithmetic on persisted measured values. Durations are measured **intervals**, not
clock readings, so the wall-clock rule is satisfied.

### Persistence

Value → decision audit record (`02` §39). Source observations remain in `07`'s
Execution State, reported through `09`. Owner: **02** §21.

---

## Stated Implementation Dependencies

These are **declared, not resolved**, in keeping with this document's
evidence-first rule. None blocks the specification; each must hold for the
measurement to be exactly recomputable.

| # | Dependency | Affects |
| - | ---------- | ------- |
| D-1 | The §18 probe series (`totalProbes`, `errorProbes`, `medianMs`, `referenceMs`) must be present in `07`'s persisted Execution State metrics (`07` §37, §39), not only in the §18 report view, for independent recomputation from a canonical dataset. | 20.3 |
| D-2 | `10` §13 records *Overall Status* and illustrates it as `PASSED`. This section deliberately classifies against `01` §17's terminal lifecycle states (`COMPLETED`/`FAILED`/`CANCELLED`) because `01` is the higher authority and its vocabulary is enumerated. Implementations must record the terminal lifecycle state, not only an informal pass/fail label. | 20.1 |
| D-3 | `W` (recency window) and `minimumExecutions` must be present in the configuration hierarchy (`01` §29). No default is asserted here, because asserting one would make an unmeasured choice look approved. | 20.1 |

---

## Verification

A future execution satisfies this section when:

- Each of the three terms is either **measured** by the procedure above or marked
  `UNAVAILABLE` with its named reason — never defaulted;
- Framework Confidence is emitted when all four terms are available, and refused
  naming every unavailable term otherwise;
- `02` §21's weights appear unchanged as `0.35 / 0.25 / 0.20 / 0.20`;
- Each term recomputes identically from its persisted inputs, including under a
  reversed enumeration order;
- A cold-start scope yields `UNAVAILABLE: NO_HISTORY` rather than `0`;
- An empty applicable-rule set yields `UNAVAILABLE: NO_APPLICABLE_RULES` rather
  than `100`.

> **Not yet demonstrated.** This section is an instruction. Runtime satisfaction
> requires an actual execution and has not occurred. RW0 validated §19 only.

---

# 21. Deep Discovery & API Capture Tactics (W7-C)

Implements the **W7-C data foundation**: discover the application deeply once,
capture its observed network/data surfaces safely, and produce evidence that maps
into the **frozen W7-B AIC v1.0.0 contract** so downstream skills need not
re-crawl. This section is **tactics (HOW)**; the AIC contract and its ownership by
`04` are authoritative and unchanged. W7-C adds no engine, dataset, lifecycle
state, identity authority, confidence model, or Evidence Quality model.

## Status Of This Section

Implemented and verified at **implementation tier** in a disposable, non-framework
surface (`qa-w7c-impl-20260811/`, 55/55 tests) against a **fully-mocked local
fixture** — no real target, no offensive testing. Runtime-tier satisfaction awaits
the separately-authorized **W7-H** wave. QA remains defensive: discover / observe /
capture / safe-validate only.

## Scope Boundary (defensive)

W7-C produces intelligence and functional evidence. It SHALL NOT exploit, and
SHALL NEVER emit the reserved AIC states `OFFENSIVELY_VALIDATED` /
`VULNERABILITY_CONFIRMED` (W7-B §A5). Those belong to the offensive/validator
authorities that later *consume* the AIC.

## Discovery Sources & Provenance

Discover, within scope, from these sources; every record carries
`provenance.discoverySource` (W7-B §A6): `crawl · js · sitemap · robots ·
network · verification`.

- **robots.txt** — fetch, record `RETRIEVED|NOT_FOUND|UNAVAILABLE`, parse
  directives + sitemap refs. A `Disallow` is **intelligence, recorded
  `DISCOVERED`** — never a bypass grant, never offensively tested (W7-B §B11).
- **sitemap** — parse `<loc>` routes, `discoverySource=sitemap`, `DISCOVERED`.
- **JavaScript routes** — extract path/API/GraphQL-like strings from JS already
  loaded by the app; exclude asset paths; `discoverySource=js`, `DISCOVERED` — a
  string in JS is **not** executable proof (W7-B §B12).
- **network** — every XHR/fetch witnessed during normal navigation → `OBSERVED`.
- **crawl** — pages/links/forms via BFS within caps.

A hidden endpoint = discovered from `js|sitemap|robots` and **not** page-linked;
record it with provenance, `DISCOVERED`, and never auto-exercise or auto-bypass.

## State Semantics (W7-B §A5 — never collapse)

`DISCOVERED → OBSERVED → EXERCISED → VALIDATED`. A witnessed call is `OBSERVED`;
a test-driven call is `EXERCISED`. `exercised` is **derived** from state, never a
stored boolean. `DISCOVERED`/`OBSERVED` ⇒ not exercised ⇒ not tested ⇒ not
vulnerable.

## URL Normalization, Route Templates, Scope & Caps

Consume `03` §18 normalization (no second normalizer): lowercase scheme/host,
default-port removal, tracking-param (`utm_*`, `fbclid`, `gclid`, …) removal,
fragment removal, sorted keys, trailing-slash strip on the **pathname**. Collapse
dynamic segments to a route template (`/users/{id}`) so many pages share one route
id (W7-B §C1). Enforce scope against `SCOPE_FILE` (host list) or same-origin
default **before any navigation**; out-of-scope targets are recorded, never
contacted. Caps (`maxPages`, `maxDepth`, RPS) come from `01` §29 — **never
hard-code a universal ceiling**; a missing cap is a disclosed config gap, never
silently infinite. Every skipped/capped/inaccessible surface is disclosed.

## API Capture — masking is the gate (W7-B §B15, `01` §30, `07` §41)

For every observed call, build masked request + response evidence and an
`api-call` record correlating **page · journey · test · execution · scope**
(consumed identities: `01` §30, `01` §30.1, `06` §22.1 — never invented). Support
REST/JSON, form-urlencoded, multipart, GraphQL, plain text, XML, HTML, binary.

**Masking runs BEFORE any persistence, through one authority** (no second masking
system):
- Sensitive **names** (Authorization, bearer/api-key/session/cookie/csrf/
  password/credential/`token`/access·refresh·id-token/PII incl. card, ssn,
  email…) → `masked`.
- Sensitive **value shapes** (Bearer…, JWT, `AKIA…`, `sk_live_…`, high-entropy) →
  `masked` even under a bland name.
- JSON/form bodies → `redacted` (structure/keys kept, sensitive values masked,
  recursively).
- **Binary** bodies → `binary` repr (contentType + size + sha256, **never raw**).
- **Unknown sensitivity defaults to `masked`.** Reprs: `plain · masked ·
  redacted · omitted · hashed · binary` (W7-B §B15).

## Schema Observation

Observe request/response schema from **real** payloads only (object fields+types,
array element type, GraphQL indicators). Uncertainty → explicit W7-B absence
(`NOT_OBSERVED`/`UNAVAILABLE`), **never a guessed schema**.

## Explicit Absence (W7-B §C7)

Never use bare `null/0/false/[]` for meaning. Use
`EMPTY · UNAVAILABLE · NOT_PRODUCED · BLOCKED · NOT_EXERCISED · NOT_OBSERVED`.
Fields owned by later waves (journeys result → W7-F; attack-surface projection →
W7-E; full graph serialization → W7-D) are emitted as `NOT_PRODUCED`, never
fabricated.

## Determinism (W7-B §A10)

Deterministic IDs (`sha256[0:16]` of canonical inputs), array ordering by id,
timestamps/durations non-semantic. Identical observations → equivalent AIC.

## Maps Into The AIC — W7-C Populates

`target · pages · routes · apis · api-calls · forms · parameters ·
auth-surfaces · robots · javascript-routes · relationships · evidence/`. It does
**not** own `04`'s serialization (W7-D) or produce journeys (W7-F) or the
attack-surface projection (W7-E) — those are `NOT_PRODUCED` here.

## Verification

Satisfied at implementation tier when: robots/sitemap/JS/hidden discovery carry
provenance and `DISCOVERED`; network capture masks every sensitive category
before persistence and never stores raw binary; calls correlate to
page/journey/test/execution; state ladder stays distinct and reserved states are
un-emittable by QA; absence is explicit; output is deterministic; and no raw
secret appears anywhere in the serialized AIC. Runtime tier is **W7-H**.

> **Implementation-tier demonstrated (W7-C).** Runtime satisfaction requires the
> separately-authorized W7-H wave and has not occurred.

---

# End of IMPLEMENTATION_PLAYBOOK.md
