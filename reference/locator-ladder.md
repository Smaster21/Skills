# Locator Strategy Ladder — and the gate that admits nothing unverified

> Executor reference. **Phase 3 Locator Verification (`03` §29 → `06` §26).**
> Extends [`site-agnostic-discovery.md`](site-agnostic-discovery.md), whose prime
> directive governs this file: no application-specific selector is ever hand-written.
> Every rung below is either a web standard or a measurement of this target.

## Locator strategy ladder

Build candidates in this order, most durable first:

1. `data-testid` / `data-test` / `data-cy`
2. ARIA role + accessible name
3. `label` association
4. `name` / `id` attribute
5. **Label-scoped**: locate the field group by its label text, then the control
   within it — the only strategy that survives repeated placeholders
6. **Container-scoped**: locate the enclosing region by its role, then the control
   within it — the strategy that survives a control repeated once per row, card or
   dialog (see below)
7. `placeholder`
8. Text content (buttons and links only)

**Structural paths (`nth-of-type` chains) are an identity key, never a locator.**

### Container-scoped — the generic "one of many identical controls"

Label-scoping solves a repeated control inside **a form**. It does nothing for one
repeated per **item** — an `Edit` link in every row, an `Add` in every card, the same
button in every modal. Every copy shares role and accessible name, so rungs 1–5 each
resolve to *N* elements and the control is pruned `UN_TARGETABLE`.

Derive the container from web standards, never from a class name:

> Walk up to the nearest ancestor carrying a **container role** — `dialog`,
> `alertdialog`, `row`, `listitem`, `article`, `region`, `form`, `group`, `table` —
> or the native element mapping to one (`<dialog>`, `<tr>`, `<li>`, `<article>`,
> `<section>`, `<form>`, `<table>`). Scope to that container, then locate within it.

Distinguish the container by **its own measured content** — the row's key cell, the
card's heading, the dialog's accessible name — never by index. A prune caused by *N*
identical copies is an instruction to try this rung: the copies stop being ambiguous
once the thing that differs between them is named.

Where the container itself cannot be distinguished by measured content, the control
stays `UN_TARGETABLE` and is ledgered. **Never fall back to `.nth(0)`** — that
silently binds to whichever copy happens to be first, which is a false pass waiting
to happen (Rule 13).

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

