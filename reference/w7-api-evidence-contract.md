# W7-A / W7-B / W7-C API Evidence — Preservation Contract

> Executor reference. **Binding.** The output/report layer is a **presentation
> and summary layer** over the existing W7 API evidence. It is additive. It never
> redesigns, downgrades, replaces, simplifies, or reinterprets W7-A/B/C.

## What already exists — do not rebuild it

| Wave | Owns | Status |
|---|---|---|
| **W7-A** | Ratified the Application Intelligence Contract (AIC), Board decision `BD-W7-1` | ratified |
| **W7-B** | The **AIC v1.0.0 schema** — frozen | **byte-identical, unchangeable here** |
| **W7-C** | Deep discovery + API capture tactics that populate the AIC | implementation tier |

The AIC is owned by **`04`** and is *"the versioned, immutable, cross-skill
interoperability serialization of the canonical application intelligence"*. It is
**not** a second knowledge graph and must not acquire a competitor.

W7-C populates: `target · pages · routes · apis · api-calls · forms · parameters ·
auth-surfaces · robots · javascript-routes · relationships · evidence/`.

## The canonical/derived split

```
W7-C capture  →  AIC v1.0.0 (canonical, owned by 04)      ← the single source of truth
                        │
                        └── qa/network/*  qa/report/*      ← DERIVED presentation only
```

> `qa/network/` is a **derived view**. It carries no API identity, no capture
> mechanism, and no authority of its own. Deleting it must lose nothing that the
> AIC does not still hold.

**Never** create a second API identity system. Where an AIC record already has a
stable identifier, **reuse it verbatim**. Human-readable aliases (`API-001`) are
permitted **only** as a 1:1 display mapping that carries the AIC id alongside;
they never replace it and never become a join key on their own.

## State ladder — never collapse

```
DISCOVERED → OBSERVED → EXERCISED → VALIDATED
```

| State | Meaning | The report must say |
|---|---|---|
| `DISCOVERED` | found in `js`, `sitemap`, `robots`, or static source — **a string is not proof** | "discovered, not called" |
| `OBSERVED` | a real browser/network exchange was witnessed | "observed" — request/response evidence exists |
| `EXERCISED` | driven by a test | "exercised by TC-…" |
| `VALIDATED` | reserved | **QA never emits this** |

`exercised` is **derived from state**, never a stored boolean. `DISCOVERED` /
`OBSERVED` ⇒ not exercised ⇒ not tested.

### Post-execution API reconciliation

After Phase 7, the derived `qa/network/api-inventory.json` view MUST be
reconciled with `tests/catalogue.json` and `execution/execution-summary.json`.
For each catalogue entry with `area: "api"` and both `apiRef` and `aicId`:

1. find the matching API record by AIC id;
2. add the test id to `exercisedByTests`;
3. promote the report-visible state to `EXERCISED` when that test reached a
   verdict and drove the request;
4. update `apisExercised` and all report/summary counts from that reconciled
   view.

The underlying observation history remains intact: an exercised endpoint still
keeps its `OBSERVED` request/response evidence. The current state shown to
readers is the highest state reached in this run. It is non-conformant for
`summary.json` or `final-report.md` to report five exercised APIs while
`network/api-inventory.json` says zero exercised, or for an endpoint table to say
"observed and exercised" while its state column remains only `OBSERVED`.

A hidden endpoint (from `js|sitemap|robots`, not page-linked) is recorded with
provenance as `DISCOVERED` and is **never auto-exercised or auto-bypassed**. A
`robots.txt` `Disallow` is intelligence — never a bypass grant.

## Provenance — one vocabulary, W7-B's

`provenance.discoverySource` ∈ `crawl · js · sitemap · robots · network ·
verification`.

The report's *observed / discovered / inferred* language is a **presentation of
these existing fields**, not a parallel scheme:

| Report says | Backed by |
|---|---|
| observed | state `OBSERVED` (`discoverySource: network`) |
| discovered | state `DISCOVERED` (`js` / `sitemap` / `robots` / `crawl`) |
| inferred | a derived association, explicitly labelled — **never rendered as observed** |

## No re-crawl, no duplicate capture

> **The report generator consumes already-captured W7-C evidence. It does not
> crawl, does not re-request, and does not re-capture.**

Where an endpoint's response is `NOT_OBSERVED` and an expectation genuinely
requires one, a **targeted verification probe** may run — and it is recorded under
the already-approved `discoverySource: verification`, correlated like any other
call. It is never a bulk replay of the endpoint inventory, and never a second
capture pipeline.

> **Correction on record.** An earlier revision of this skill instructed the
> planner to *"replay each observed endpoint"* to measure responses. That is
> duplicate API collection and is **withdrawn**. Response expectations come from
> W7-C's captured response evidence; `verification` probes are the narrow,
> recorded exception.

## Deterministic dedup and no-re-crawl (W8, folding W7-A §11)

Every page, route, API, form, parameter and action carries a deterministic
identity (`sha256[0:16]` of canonical inputs). **A surface that already has an
identity SHALL NOT be explored again by default.**

A revisit is permitted **only** with a recorded reason:

| Reason | Example |
|---|---|
| distinct state | the same route before and after a record is created |
| distinct authentication context | the same route unauthenticated and authenticated |
| workflow transition | the route is a step in a different multi-step flow |
| validation | a locator or assertion target probed in its **triggered** state |
| explicitly authorized re-discovery | the target materially changed, or data outside the contract is needed (W7-B §C11) |

The reason is persisted with the revisit record. **A revisit without a recorded
reason is a defect.** Re-discovery never silently replaces prior history — it is
recorded as new provenance/evidence.

> This is the contract that makes locator/assertion probing legitimate
> (`validation`) while making a bulk endpoint replay a defect (no reason applies).

## Masking — one authority, already implemented

Masking runs **before any persistence, through one authority** (W7-B §B15,
`01` §30, `07` §41). Reprs: `plain · masked · redacted · omitted · hashed ·
binary`. Sensitive names *and* sensitive value shapes are masked; JSON/form bodies
are `redacted` structurally; binary is `binary` (contentType + size + sha256,
never raw); **unknown sensitivity defaults to `masked`**.

> **Do not implement a second masking system.** The report layer consumes
> already-masked evidence and must never weaken, bypass, or re-derive it. A report
> that needs an unmasked value does not get one.

## Explicit absence — never bare nulls

Use W7-B §C7 vocabulary: `EMPTY · UNAVAILABLE · NOT_PRODUCED · BLOCKED ·
NOT_EXERCISED · NOT_OBSERVED`. Fields owned by later waves stay `NOT_PRODUCED` —
journeys (W7-F), attack-surface projection (W7-E), full graph serialization
(W7-D). Never fabricate them, and never substitute `null`, `0`, `false`, or `[]`
for meaning.

`api_capture_status` in the report is one of `complete · partial · unavailable ·
failed`, and when not `complete` it states **why**.

## Caps — configured, never hard-coded, always disclosed

Caps (`maxPages`, `maxDepth`, RPS) come from the configuration hierarchy
(`01` §29). **Never hard-code a universal ceiling.** A missing cap is a
**disclosed configuration gap**, never silently infinite. Every skipped, capped or
inaccessible surface is disclosed in the coverage ledger.

This is the correct reading of "no caps": no *arbitrary, invented* limit such as
`slice(0, 20)`. A configured, disclosed cap is legitimate.

## Correlation chain — do not break it

```
API → request/response evidence → page → journey/workflow → test case
    → test execution → evidence
```

Correlation and provenance fields already exist in the `api-call` record. Reuse
them. Identities are consumed from `01` §30, `01` §30.1, `06` §22.1 — **never
invented**.

## Determinism

AIC IDs are `sha256[0:16]` of canonical inputs; arrays order by id; timestamps and
durations are non-semantic. Identical observations produce an equivalent AIC. Any
derived view must preserve this — a presentation layer may not reorder or re-key
canonical data.

## Site Explorer boundary

qa-automation populates only the QA/application-intelligence fields it owns. It
SHALL NOT produce security-applicability decisions from the AIC, interpret it as a
security recommendation, classify security applicability, recommend attack
targets, or assign vulnerability probability. The attack-surface projection
remains `NOT_PRODUCED` here (W7-E) and is produced and interpreted solely by
downstream security authorities.

## Regression checklist — verify before declaring this phase complete

- [ ] W7-A ratification untouched
- [ ] **W7-B AIC v1.0.0 schema byte-identical** — no field renamed, removed, reinterpreted or restructured
- [ ] W7-C deep discovery intact: crawl · js · sitemap · robots · hidden-endpoint discovery
- [ ] full request capture intact; full response capture intact
- [ ] masking intact and not duplicated; no raw secret in any artifact
- [ ] provenance + `discoverySource` intact
- [ ] state ladder distinct; `VALIDATED` / `OFFENSIVELY_VALIDATED` / `VULNERABILITY_CONFIRMED` un-emittable by QA
- [ ] API ↔ page ↔ workflow ↔ test ↔ execution correlation intact
- [ ] no second API identity system introduced
- [ ] **no re-crawl and no duplicate capture introduced for reporting**
- [ ] explicit-absence vocabulary used; no bare nulls carrying meaning
- [ ] deterministic dedup honoured; every revisit carries a recorded reason
- [ ] `ALLOW_WRITE_TESTS` remains the sole write gate (no competing control)
- [ ] existing W7-C tests still pass
