# Multi-Step Workflows — discovery, inference, execution

> Executor reference. **Mandatory for Phases 1, 4, 5 and 7 whenever a target has
> multi-step business flows.** Implements `docs/05` §19 *Workflow Planning
> Engine*, which already specifies this and states: *"Business workflows are
> planned before individual pages. Testing isolated pages without workflow
> context is discouraged."*

## What a workflow is here

A named sequence of steps that carries state forward:

```
pageA --[affordance 1]--> pageB --[affordance 2]--> pageC --[affordance 3]--> pageD
```

Concrete shapes this covers, on any kind of application: a purchase path, a
leave request and its approval, a patient admission, a support ticket lifecycle,
an onboarding wizard, a multi-page filing. The engine reads the graph, not the
domain — it needs no notion of what the application is *for*.

Testing those pages individually proves they render — not that the flow works,
which is usually what the business cares about.

## What this is NOT — the ownership boundary

| Thing | Owner | Status |
|---|---|---|
| AIC `journeys` — the canonical, cross-skill journey serialization | `04`, schema by a later wave (W7-F) | stays `NOT_PRODUCED` here |
| **QA workflows** — test-oriented flows | **`05` §19** | produced here |

A QA workflow is a planning/generation artifact with a `WF-` identifier. It is
never written into the AIC `journeys` collection and creates no second journey
model; where the AIC later carries journeys, QA workflows reference them.

## The four steps

### 1 · Discovery records affordances and transitions (read-only)

Discovery already extracts actionable controls. For workflow purposes it also
records, per page:

- each actionable control's accessible name, its owning form, and whether
  activating it would issue a non-`GET` request (structural write-intent, per
  `site-agnostic-discovery.md`);
- **observed transitions** — where a navigation actually landed after following a
  link, recorded as `pageA --[link text]--> pageB` with `discoverySource: crawl`.

Discovery stays read-only: it does not press action buttons. It records that
an affordance exists; it does not activate it.

The exact field shapes both halves must emit and read are in
`output-and-scope.md` → **Workflow input collections**. Emit those names verbatim;
any other shape leaves discovery and planning unable to connect.

### 2 · Planning infers candidate flows, labelled `INFERRED`

`05` §19 assembles candidates from what discovery saw — control names, form
targets, page relationships, and the observed API surface. Each candidate is
recorded as:

```json
{
  "workflow_id": "WF-001",
  "name": "<derived from the affordance names in order>",
  "source": "inferred",
  "confidence": 0.62,
  "steps": [
    { "index": 1, "page": "PAGE-014", "action": "<affordance>", "writeIntent": false },
    { "index": 2, "page": "PAGE-021", "action": "<affordance>", "writeIntent": true,
      "stepClass": "state_creating" }
  ],
  "evidence_refs": ["EVID-…"]
}
```

`source: "inferred"` is load-bearing. A candidate flow is a **hypothesis derived
from page structure**, not a witnessed behaviour, until execution proves it.
Reporting it as observed would be the same defect as reporting a `DISCOVERED`
endpoint as called.

Where the structure supports no candidate, the workflow list is empty and a
coverage-ledger limitation records why. A flow is never invented to fill a gap.

### 3 · Generation emits one multi-step test per flow

One test per workflow, each step asserting that state actually **moved**:

Each step asserts that **something observable changed** — measured on this target
during discovery, not assumed. Any of these qualifies:

- the URL or route advanced;
- a counter, badge or total changed value;
- a row, record or list entry appeared or disappeared;
- a region became present, absent, enabled or disabled;
- an `aria-live` region or status message announced a change.

```
step 1  activate <affordance>  → assert one measured observable changed
step 2  activate <affordance>  → assert the change from step 1 persists
step 3  activate <affordance>  → GATED (see below)
```

A step that only asserts "the button was clickable" proves nothing. Each step's
assertion cites the measurement it came from (`suite-self-validation.md` Gate A),
and every step is individually falsifiable (Gate B).

### 4 · Execution promotes the flow

Running the test is what turns a hypothesis into evidence, using the existing
state ladder — no new vocabulary:

| Outcome | Result |
|---|---|
| every ungated step passed | `INFERRED → EXERCISED` |
| stopped partway | `PARTIALLY_EXERCISED`, with the step index reached |
| a step failed | flow stays `INFERRED`; the failing step is diagnosed normally |

## Step gating — how far a flow is walked

Steps are classified by reversibility, and each class has its own gate:

Classification keys on **structure**, never on vocabulary. A word list is
language- and domain-bound; HTTP and HTML semantics are not.

| Class | Structural test | Gate |
|---|---|---|
| **Reversible** | activation issues only `GET`; no form submission | runs by default |
| **State-creating** | submits a form whose method is not `GET`, or issues a non-`GET` request | runs when `ALLOW_WRITE_TESTS=1`, with synthetic data and cleanup |
| **Irreversible** | non-`GET` **and** any of: no inverse affordance exists on the resulting page · a confirmation dialog gates it · operator config names it | each requires its own explicit opt-in; **off by default** |

The "no inverse affordance" test is the portable one: a create is reversible when
the resulting page offers a delete or undo; a step with no way back is
irreversible whatever it is called, in any language.

Where a control's method cannot be established, it is classed **state-creating**
and gated. An unknown is never assumed safe.

No text lexicon is used for classification. An English word list misses a
localised target and encodes one business domain into a framework meant for any
(`known-failure-modes.md` FM-9).

### Reporting a partial walk honestly

A flow that stops at a gate is a **useful result**, reported as such:

> **WF-001 (3 steps): 2 of 3 verified.** Stopped at step 3 — classed
> irreversible (non-`GET`, no inverse affordance on the resulting page), not
> enabled for this run. Steps 1–2 confirmed the observable state advanced and
> persisted.

That is materially more informative than either "flow untested" or a claim that
the purchase path works. The coverage ledger records the gated step as
`POLICY_EXCLUDED` with its consequence stated.

## Coverage

Workflow coverage is `flows EXERCISED ÷ flows discovered`. A partially-walked
flow counts toward neither — it is reported in its own row with the step reached,
so a high coverage number can never hide a flow that stopped at step 1 of 6.
Where no flow was discovered, coverage is `UNAVAILABLE` with the reason, never
`0%` and never `100%`.

## What this changes per phase

| Phase | Obligation |
|---|---|
| 1 DISCOVERING | record affordances (with structural write-intent) and observed transitions |
| 4 PLANNING | assemble candidate flows as `INFERRED`; plan flows before individual pages (`05` §19) |
| 5 GENERATING | one multi-step test per flow, per-step state assertions, gated steps skipped with a recorded reason |
| 7 EXECUTING | promote `INFERRED → EXERCISED`, or record the step reached |
| 10 REPORTING | per-flow status, step reached, and the gate that stopped it |
