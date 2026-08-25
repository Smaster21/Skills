# The Coverage Ledger — nothing disappears quietly

> Executor reference. **Mandatory artifact and gate — Phases 4 and 10.** Every other fix in this
> skill repairs a past mistake. This one prevents the next one.

## Why this exists

A real run reported **121 tests passed** while, unreported:

- 37 of 58 discovered API endpoints dropped by a `slice(0, 20)` nobody declared;
- 116 of 169 verified elements never referenced by any test;
- every control lacking `id`, `name` and `placeholder` discarded silently;
- one test **passed** while driving an element other than the one it named.

None of that was visible in the report: the only numbers shown were about tests that
ran, never about work that vanished. A suite that cannot say what it skipped cannot be
trusted about what it covered.

## The rule

> **Every artifact discovered on the target MUST terminate in exactly one of two
> states: `TESTED`, or `EXCLUDED` with a machine-readable reason.** A third state —
> silently absent — is a defect of the run, not a detail.

## Ledger scope

Account for every population:

| Population | Source |
|---|---|
| Routes / pages | `03` discovery |
| Form controls (incl. un-nameable ones) | `03` discovery |
| Choice widgets (native and custom) | `03` profiling + discovery |
| Actionable controls (buttons, submits, menu items) | `03` discovery |
| Route instances, per template | `03` discovery (four states, below) |
| Forms | `03` discovery |
| Tables | `03` discovery |
| API endpoints, per method | `03` network observation |
| Assertion targets | `03` §29 triggered-state probes |

Per population record `discovered`, `tested`, `excluded`, and an `unaccounted` count
that **must be zero**.

## Exclusion reasons — closed vocabulary

An exclusion carries one of these. Free text alone cannot be counted or audited.

| Reason | Meaning |
|---|---|
| `WRITE_OPERATION` | Would create, modify, or delete data (`Rule 3`, or plan policy) |
| `DESTRUCTIVE_CONTROL` | Accessible name matches the destructive-intent pattern |
| `UN_TARGETABLE` | No strategy resolved to exactly one element; pruned at the `03` §29 gate |
| `NO_INTENT_MATCH` | A planned case found no verified locator matching its named intent |
| `UNREACHABLE` | Discovered as a menu entry with no resolvable destination |
| `OUT_OF_ORIGIN` | Outside the authorized target origin |
| `NOT_AUTHORIZED` | Outside scope, RoE, or the browser matrix |
| `ENVIRONMENT_BLOCKED` | Target refused or failed to serve it during the run |
| `POLICY_EXCLUDED` | Deliberate plan decision — **must** carry a stated consequence |
| `UNSAFE_TO_NAVIGATE` | A discovered route the structural filters rejected before navigation — **must** carry the filter that rejected it |
| `INSTANCE_EXCLUDED_BY_POLICY` | An instance excluded by an explicit plan decision — **must** carry the reason and its consequence |
| `INSTANCE_EXCLUDED_BY_CAP` | An instance beyond the declared crawl ceiling (`MAX_ROUTE_INSTANCES` / `maxInstancesPerTemplate`) — **must** carry the bound reached |

`POLICY_EXCLUDED` additionally requires a `consequence` field saying plainly what
is therefore unverified (for example: *"create/update business rules are
UNTESTED and must not be read as passing"*).

## Route templates: every instance, or a named reason

A catalogue route repeats one template across many ids. Collapsing every instance into
one crawled page is **not** dedupe — it is a silent cap wearing dedupe's clothes. A run
that crawls one detail page and drops the other 33 has tested a template, not an
application; reporting only `TEMPLATE_TESTED` makes "25 pages" read like 64 routes.

Every route instance terminates in exactly one of **four** states, reported separately:

| State | Meaning |
|---|---|
| `TEMPLATE_TESTED` | the route template has at least one crawled representative |
| `INSTANCE_TESTED` | this specific instance was crawled and measured |
| `INSTANCE_EXCLUDED_BY_CAP` | the declared crawl ceiling was reached; accounted with the bound |
| `INSTANCE_EXCLUDED_BY_POLICY` | an explicit plan decision excluded it; accounted with its reason |

> **`INSTANCE_SKIPPED_BY_SAMPLING` is retired and MUST NOT be emitted.** One crawl
> behaviour exists — exhaustive within the declared ceiling — so "the sample was
> satisfied" is not a reason an instance may go uncrawled. The permitted reasons are the
> ceiling, `OUT_OF_ORIGIN`, `UNSAFE_TO_NAVIGATE`, an explicit policy, or an
> already-visited URL — and the last is not an exclusion, because the same URL is one
> route counted once (`scope-enforcement.md` → *Crawl behaviour*).

### The denominator is DISTINCT routes, never encounters

A route linked from thirty pages is **one** route; counting encounters inflates `discovered` and deflates every percentage computed from it.

> A real run reported `routes discovered: 199, tested: 25`. The target had **64**
> distinct routes; 174 of the 199 were repeat *encounters* of 39 URLs. Both the 199 and
> the 12.6% derived from it were wrong, in opposite directions.

`discovered` counts distinct normalized URLs; encounter counts may be reported
alongside, labelled as encounters.

### Instance coverage is reported per template

```
Routes (distinct)  discovered 64  tested 60  cap-excluded 4  unaccounted 0
  per template        instances  tested  note
  /item/{id}                 33      33  every instance crawled
  /group/{id}                 7       7  every instance crawled
  /vendor/{name}             12       8  ceiling MAX_ROUTE_INSTANCES=8 reached (4 cap-excluded)
```

A report **SHALL NOT** describe template coverage as instance coverage, and **SHALL
NOT** claim full instance coverage unless every discovered instance was crawled. Where a
ceiling excluded instances, it names the ceiling and the count. The crawl is exhaustive
by design, so any shortfall is a **measured** limit, never a sampling choice — and the
deeper map carries no additional authority: output stays QA-only (`Rule 21`).

## Caps are forbidden

There is **no** URL cap, no per-category cap, no `slice(n)`, no "first N". If a limit
is genuinely unavoidable it is not a cap — it is an exclusion, declared per item with a
reason. A truncation that produces no ledger entry is the failure this document exists
to stop.

That is what `INSTANCE_EXCLUDED_BY_CAP` is, and why it does not contradict the rule
above: the ceiling is visible configuration, it emits one ledger entry **per instance**
carrying the bound, and the report states that bound. The prohibition is on a **silent**
truncation, never a **declared** one.

## The gate

Reporting (`09`) **SHALL**:

1. Refuse to present a run as complete while `unaccounted > 0` in any population.
2. Publish `tested` / `discovered` per population, not just pass counts.
3. Surface `NO_INTENT_MATCH` and `UN_TARGETABLE` prominently — those hid real loss.
4. Report **verified-but-unused** (passed the locator gate, referenced by no test); a
   large value means the planner, not discovery, is the bottleneck.
5. Report **unexercised framework capability** — handling this run possesses but never
   ran, because the target contained nothing to run it against.

### Unexercised capability — untested code is unproven code

A population the target does not contain (no iframes, no custom choice widgets, no
tables) yields `discovered: 0`. That is neither coverage nor an application gap — it
means **this run proved nothing about that handler.**

Record it as its own line, distinct from an exclusion:

```json
{ "capability": "customChoiceWidgets", "discovered": 0,
  "state": "NOT_EXERCISED",
  "consequence": "The click-then-pick widget path did not run on this target. Its correctness is UNPROVEN, not verified." }
```

`NOT_EXERCISED` is a W7-B absence term: neither pass nor fail. It is the difference
between *"this works"* and *"this was never tried"*, and the honest basis for FM-10's
validation tier — a capability that never executed cannot support a portability claim.

## Sampling, review tripwires, and tags

Computed from this ledger, specified in
[`planning-coverage-controls.md`](planning-coverage-controls.md): **repeated shell
controls** (sample across routes with a stated `basis`, never test one and exclude the
rest — FM-2), **plan review tripwires** (Phase 4A arithmetic), and **test tags**
(derived, so the suite runs in useful subsets). Route-instance sampling and the crawl
modes are Phase 1 behaviour: [`discovery-profiling.md`](discovery-profiling.md).

## Artifact

Write `qa/coverage/coverage-summary.json`; render its summary in every report format alongside the pass/fail totals.

```
{
  "population": "apiEndpoints",
  "discovered": 59,
  "tested": 57,
  "excluded": [
    { "id": "POST /events/push", "reason": "WRITE_OPERATION" },
    { "id": "GET /core/i18n/messages", "reason": "POLICY_EXCLUDED",
      "consequence": "Public localisation bundle; covered by a dedicated anonymous-access test instead." }
  ],
  "unaccounted": 0
}
```

## Worked example — the shape a report must reach

```
Routes      discovered  70   tested  70   excluded  0    unaccounted 0
Controls    discovered 412   tested 340   excluded 72   unaccounted 0
                                           └ UN_TARGETABLE 61, DESTRUCTIVE_CONTROL 11
Actions     discovered 121   tested  95   excluded 26   unaccounted 0
API (GET)   discovered  58   tested  58   excluded  0    unaccounted 0
API (POST)  discovered   1   tested   0   excluded  1    unaccounted 0
                                           └ WRITE_OPERATION 1
Verified but unused: 0
```

Read against that, "121 passed" is a claim about the suite, not the application — and
the ledger says how much of the application the suite actually reached.
