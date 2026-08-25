# Discovery — Profiling and the Exhaustive Crawl

> Executor reference. Steps 3–4 of the run. The end-to-end command sequence is in
> `quickstart.md`; the technique behind these steps is in
> `site-agnostic-discovery.md`.

## 3. Profile the target (Phase 0B) — learn, never assume

**Before crawling.** Sample the entry page plus several in-navigation routes and
measure the target's own conventions. Full technique:
[`site-agnostic-discovery.md`](site-agnostic-discovery.md).

```
qa/knowledge/application-model.json (+ discovery/discovery-summary.json)
```

Record: landmarks stable across every sampled route (the shell), which
accessible-name source actually wins, the label-coverage ratio, whether choice
controls are native `<select>` or custom widgets (**confirm by behaviour** —
activate one and count option-like elements before/after), whether activating a
disclosure reveals navigation, and the login form shape derived from the password
field's form.

Expected: a profile containing **no hand-written selector for this application**.
A label-coverage ratio below 1.0 names controls that are un-targetable — they go
to the ledger, they are never dropped silently.

## 4. Discover (Phase 1) — depth-first, exhaustive, read-only

Crawl **depth-first**: finish a page's entire subtree before its next sibling.
Expand navigation disclosures when profiling proved they reveal routes. **No URL
cap, no `slice(n)`.**

**Every discovered instance of a repeated route template is crawled**, up to the
declared `MAX_ROUTE_INSTANCES` ceiling — never sampled, never collapsed to one page.
Each instance terminates in one of the four route states: `TEMPLATE_TESTED` ·
`INSTANCE_TESTED` · `INSTANCE_EXCLUDED_BY_CAP` · `INSTANCE_EXCLUDED_BY_POLICY`. Seed the
frontier with same-origin path literals found in the page's own scripts
(`discoverySource: js`), filtered structurally before navigation so the read-only
boundary holds. Ledger contract: **`coverage-ledger.md`**.

```
qa/discovery/discovery-report.json     pages (depth, parent, discovered-via), network, skipped+reason
qa/discovery/component-inventory.json  UNVERIFIED candidates, incl. custom widgets
qa/evidence/screenshots/*.png
```

Read-only boundary: navigate and open menus only. **Never** submit a form; never
activate a control whose accessible name matches destructive/write intent
(delete, remove, purge, save, submit, add, reset, approve, upload, export…).

Expected: every page carries `depth`, `parent`, and the affordance it was found
through. Menu entries with no resolvable destination are recorded `UNREACHABLE`.

### Route instances — every one of them, within the declared ceiling

A catalogue template (`/item/{id}`) yields many instances. Crawling **one** and calling
the template covered is a silent cap that reports a template as an application, so this
skill does not do it: **every discovered instance is crawled**, up to the declared
`MAX_ROUTE_INSTANCES` ceiling. There is no sampling behaviour and no mode that enables
one (`scope-enforcement.md` → *Crawl behaviour*).

```
crawl the instance when   crawled < MAX_ROUTE_INSTANCES
at the ceiling            INSTANCE_EXCLUDED_BY_CAP   (declared exclusion, per instance)
early-stop on repeat      NOT ALLOWED — a repeated structural signature is evidence
                          about STRUCTURE; the crawl is collecting CONTENT
```

An instance goes uncrawled for **exactly five** reasons, each recorded with its
machine-readable state: the **ceiling** was reached, the **origin boundary** applies,
the route is **unsafe to navigate** by the structural filters below, an **explicit
policy** excludes it, or the **URL was already visited** — and an already-visited URL is
the same route counted once, not an exclusion. Nothing else may stop an instance.

**Template dedupe is loop protection, never a coverage decision.** Its only job is to
stop a crawl that would otherwise never terminate. "This template has been seen" is
never permission to skip the rest of a catalogue.

**Structural signature — reported, never a skip reason.** Per crawled page, over its
*structure only*: HTTP status, per-route landmark counts, the sorted set of **distinct**
control identity keys, form count with methods. Title and outbound links are excluded as
content. The signature tells a reader whether a template's instances are structurally
identical, which is a useful measured fact; it **must not** gate crawling, and a repeat
never ends the crawl.

`MAX_ROUTE_INSTANCES` is the infinite-crawl guard and a **declared exclusion with a
reason**, never a silent truncation. Its value is configuration and is reported with the
instance counts it produced.

### Routes are reached from more than links

Link-following alone under-discovers. A route referenced only from script (a handler
that assigns `location`, a fetch target, a client-side route table) is invisible to a
crawler that reads `href` attributes, even when it serves a normal page. Seed the
frontier with same-origin path literals extracted from the page's own scripts,
recorded as `discoverySource: js` (`Rule 23`), and validate each by navigation before
admitting it.

Seeding is subject to the **read-only boundary**, and the boundary is structural:

| Rejected before navigation | Why |
|---|---|
| a literal ending in `/` | a concatenation prefix, not a route — its completed form is an action |
| a final path segment that is purely numeric | an id assembled at runtime; instances arrive from links |
| a static-asset extension | not a route |
| a response that is not 2xx `text/html`, or redirects off-path | not a page (same shape check as `robots.txt`) |

Those four filters are the guard. **A candidate that cannot be shown harmless
structurally is not navigated** — an unknown is never assumed safe, and discovery
still creates, edits and deletes nothing.

### Static sources — a `200` is not proof the file exists

`robots.txt` and `sitemap.xml` seed the crawl (`discoverySource: robots` /
`sitemap`). Both MUST be validated by **content type and shape**, not by status
code alone:

| Check | Requirement |
|---|---|
| `robots.txt` | `content-type` is a `text/plain` family **and** the body parses as directives (`User-agent:` / `Allow:` / `Disallow:` / `Sitemap:`) |
| `sitemap.xml` | `content-type` is an XML family **and** the body parses as XML containing `<urlset>` or `<sitemapindex>` |
| Either | The response was **not** reached through a cross-path redirect to an unrelated document |

A response failing its shape check is recorded `NOT_PRESENT` with the observed
status, content type, and final URL. It is **never** recorded as a retrieved file,
and no route is seeded from it.

> A target that answers unmapped paths with a redirect to its home page returns
> `200 text/html` for `/robots.txt`. Trusting the status recorded "robots.txt:
> status 200, 1263 lines" for a site that has **no robots.txt at all** — those
> lines were the home page. Worse, the report then read as though crawler rules
> had been honoured, when there were none to honour. An absent file is a fact
> worth reporting; a fabricated one is not.

Report the outcome for each source explicitly — `PRESENT` (with the directives or
URL count actually parsed) or `NOT_PRESENT` (with the reason). Where robots
directives *are* present, record the disallowed paths and honour them, and state
the count that was excluded. **Never claim rules were respected when none were
served** — that is a vacuous claim and it reads as evidence of good behaviour.

### Re-discovery (Phase 0A) — re-measure what moved, never trust what didn't

A full exhaustive crawl is the correct default and the only valid **first** pass.
Repeating it against an unchanged target is waste, and waste discourages frequent
runs — precisely when a regression suite earns its keep.

Enable incrementality with a **per-page fingerprint** recorded during discovery,
composed only of measured facts:

```
fingerprint = sha256( httpStatus · normalized route · document title ·
                      per-route landmark counts · sorted control identity keys ·
                      form count + each form's method · sorted outbound route set )
```

On a later run, re-fetch each known page and recompute. Then:

| Outcome | Action |
|---|---|
| Fingerprint identical | Carry the page's model forward, marked `CARRIED_FORWARD` with the prior run id |
| Fingerprint differs | **Re-measure that page in full**, and re-verify every locator on it |
| New outbound route appears | Crawl its subtree exhaustively, as a first pass |
| Page now unreachable | Record `UNREACHABLE`; do not carry forward its model |

Three constraints keep this honest:

1. **Carried-forward is a provenance state, not a measurement.** A carried page's
   items keep their original `measuredAt` and are reported as carried, never as
   measured this run. A reader must be able to tell which numbers are fresh.
2. **Locator verification is never carried forward across a changed page.** The
   fingerprint includes control identity keys precisely so that a DOM change forces
   re-verification rather than reusing a stale gate result.
3. **Assertion baselines are per route (FM-2).** A carried baseline is valid only
   for the exact route whose fingerprint matched.

Where no prior run exists for the Execution Scope Identity, or the profile's shell
facts changed, Phase 0A **falls back to full discovery** — and says so. Skipping to
incremental on a changed shell would carry forward baselines the target no longer
honours.
