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
