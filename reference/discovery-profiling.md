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
