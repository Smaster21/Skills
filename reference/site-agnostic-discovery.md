# Site-Agnostic Discovery, Locators, and Interaction

> Executor reference. **Mandatory technique.** This file exists because a run
> against one application produced a pipeline that only worked against *that*
> application: 126 hand-written framework selectors, every label unresolved, zero
> dropdowns modelled, and a test that passed while driving the wrong element.
> Nothing here names a framework, a CSS class, or a URL shape.

## Prime directive

> **No application-specific selector, class name, URL prefix, or text literal may
> be hand-written anywhere in the pipeline.** Everything the suite needs about a
> target is **measured from that target** and persisted as evidence.

A selector like `.oxd-input-group`, `.MuiSelect-root`, `.ant-form-item`, or
`/web/index.php` is a defect the moment it is typed, even when it works. It
encodes one application into a framework meant for any.

The two legitimate sources of a selector are:

1. **Web standards** — ARIA roles, landmark elements, native form semantics, the
   accessible-name algorithm. These are the same everywhere.
2. **A measurement of this target**, recorded in `qa/discovery/discovery-summary.json`.

---

## Phase 0B — Site Profiling (run before DISCOVERING)

Learn the target's conventions instead of assuming them. Sample the entry page
plus several in-navigation routes and record:

| Measured | Why it matters |
|---|---|
| **Landmarks stable across every sampled route** | This is the application shell. It is the only safe basis for a "page loaded" assertion. |
| **Which accessible-name source actually wins** (`aria-label`, `label[for]`, wrapping label, field-group text, `placeholder`, `title`) | Tells you whether labels are reliable on this target, and reveals *un-nameable* controls. |
| **Label coverage ratio** — named controls ÷ visible controls | A ratio below 1.0 means some controls cannot be targeted. They are **reported as un-targetable, never silently dropped**. |
| **Choice-control style** — native `<select>` vs custom widget | Decides whether the suite may use `selectOption()` or must click-then-pick. |
| **Whether activating a disclosure reveals navigation** | Decides whether the crawl must expand menus to see the application. |
| **Login form shape** | Derived from the password field, its form, its sibling text field, and its submit control. Never hard-coded. |

Write `qa/discovery/discovery-summary.json`. Every later phase consumes it.

### Never infer the shell from one page

A shell anchor measured on a single route and then applied globally is a known
false-negative generator: the same anchor legitimately resolves to 0, 1, or 2
elements on different routes of the same application. **Measure per route, assert
per route.**

---

## Deriving a label without framework knowledge

Apply the accessible-name algorithm in precedence order. Stop at the first hit:

1. `aria-labelledby` → concatenated text of referenced elements
2. `aria-label`
3. `<label for="...">`
4. Wrapping `<label>` (with nested controls stripped)
5. **Field group** — see below
6. `placeholder`
7. `title`
8. For buttons/links only: text content, then `value`

### Field group — the generic form-row

Frameworks each invent a wrapper class for "label + control". Derive it instead:

> Walk up from the control. The field group is the **nearest ancestor containing
> exactly one labelable control and carrying its own text**. Stop early if an
> ancestor contains more than one control.

This yields the same element the framework's own wrapper class would, on every
framework, without naming any of them.

> **Never** use a selector list like `closest('.some-group, .form-group, div')`.
> `closest()` returns the nearest ancestor matching **any** member, so a generic
> `div` wins and the label resolves empty. This exact mistake nulled every label
> in a real run, which in turn silently discarded every field that had no `id`,
> `name`, or `placeholder`.

---

## Finding controls the markup does not declare

Many applications build dropdowns, date pickers, and menus from plain elements
with no ARIA. Markup inspection alone will report **zero** choice controls on
such a target. Use two passes:

**Pass 1 — structural.** A container that carries label-like text, contains **no**
native form control, but does contain a **focusable** element, is a custom widget.

**Pass 2 — behavioural (authoritative).** Activate the candidate and observe:

```
optionsVisibleBefore  →  activate  →  optionsVisibleAfter
```

Count elements with option semantics (`[role=option]`, `[role=menuitem]`,
`[role=treeitem]`, `li`, `option`). If the count rises, the control **is** a
choice widget — confirmed by behaviour rather than by markup. Record the evidence.

Apply the same principle to navigation. An element is a candidate affordance if it
is focusable **or** has `cursor: pointer`; activate it and diff the link set.
Elements that are pointer-interactive but not keyboard-reachable are a genuine
accessibility defect of the target — **record that fact**, and still use them.

Probe on **every sampled route**, not just the entry page: a module's own tabs
usually exist only on that module's pages.

---

## Locator strategy ladder

Build candidates in this order, most durable first:

1. `data-testid` / `data-test` / `data-cy`
2. ARIA role + accessible name
3. `label` association
4. `name` / `id` attribute
5. **Label-scoped**: locate the field group by its label text, then the control
   within it — the only strategy that survives repeated placeholders
6. `placeholder`
7. Text content (buttons and links only)

**Structural paths (`nth-of-type` chains) are an identity key, never a locator.**

### The gate is unchanged, and absolute

Probe every candidate live, **in the auth state its test will run in**. Keep only
those resolving to exactly one element. A pruned candidate is recorded with its
failing count.

### Intent, not availability — the false-pass rule

> When a planned case names a target ("the Employee Name filter"), it must bind to
> a verified locator whose **accessible name matches that intent**. If none does,
> **drop the case and record it in the coverage ledger**. Never fall back to
> "the first verified control on the route".

This rule exists because that fallback produced a **passing test that asserted
nothing about the field it named** — it drove an unrelated search box while
reporting coverage of a filter it never touched. A false pass is worse than a
failure: a failure gets investigated, a false pass ships. Only its sibling test's
failure exposed it.

---

## Traversal — depth-first and exhaustive

- **Depth-first.** Finish a page's entire subtree before moving to its sibling.
- **Expand disclosures** when profiling showed they reveal navigation. Restricted
  to navigation affordances; see the safety boundary below.
- **No caps.** No URL cap, no per-category cap, no `slice(n)`. If a limit is
  genuinely unavoidable it is a **declared exclusion with a reason** in the
  coverage ledger, never a silent truncation. See `coverage-ledger.md`.
- Record for every page: depth, parent, and the affordance it was discovered
  through, so the traversal is auditable.
- Menu entries with no resolvable destination are recorded as **unreachable**, not
  guessed at.

### Safety boundary for interaction

Discovery may **navigate and open menus**. It may never submit a form, and never
activate a write-intent control.

Establish write intent **structurally first** — a text lexicon is a secondary
filter, never the primary guard:

1. the control submits a form whose `method` is not `GET`;
2. it is `type="submit"`, or a default-type `<button>` inside a form;
3. activating it would issue a non-`GET` request;
4. its role/attributes mark it destructive (`aria-*`, confirmation dialog).

Only after those, apply a text lexicon as an **additional** filter.

> A lexicon of English words is not a safety mechanism. On a target localised in
> another language "Löschen", "削除" or "Eliminar" sail straight through an
> English pattern, and read-only discovery mutates data. **Where intent cannot be
> established structurally, do not activate the control** (`known-failure-modes.md`
> FM-9).

Discovery still creates, edits, and deletes **nothing**.

---

## What this changes downstream

| Phase | Consumes |
|---|---|
| `03` Discovery | the profile; expands menus only if profiling proved it necessary |
| `03` §29 Verification | the strategy ladder; label-scoped probing for ambiguous controls |
| `05` Planning | intent-based binding; workflow cases; every unbound intent → ledger |
| `06` Generation | click-then-pick for custom widgets, `selectOption()` only for native `<select>` |
| `09` Reporting | the ledger, published as coverage truth |
