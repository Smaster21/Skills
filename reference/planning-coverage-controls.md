# Planning Coverage Controls — sampling, review tripwires, tags

> Executor reference. **Phase 4 PLANNING / 4A PLANNING_REVIEW.** Companion to
> [`coverage-ledger.md`](coverage-ledger.md), which defines the ledger these three
> controls are computed from. Load both together for Phase 4.
>
> All three answer the same question from different angles: *the ledger says what
> was excluded — but was excluding it the right call, and can a reader act on the
> result?*

## Repeated shell controls — sample across routes, never test one and exclude the rest

An application shell repeats the same controls on every route. Testing all of them is
waste: one run held **464 navigation-affordance instances**, and asserting the same
eight links on 58 routes buys almost nothing.

Testing **one route** and excluding the other 456, however, silently asserts that the
shell behaves identically everywhere — a single-route measurement promoted to a global
invariant, which is exactly FM-2. And it is measurably false often enough to matter:
in that same run the shell varied across routes —

```
banner       58 of 65 routes      navigation    2 of 65 routes
contentinfo  58 of 65 routes      main          2 of 65 routes
```

Seven routes carried no banner at all. Navigation simply was not tested on any route
where the shell differed.

**Sample deliberately, from measured structure:**

1. the entry route;
2. one route per **route template**;
3. **every** route whose measured landmark set differs from the modal shell — these
   are the routes a single-route baseline is guaranteed to mis-describe.

Then ledger the remainder as a sample, not as a dismissal:

```json
{ "population": "navigationAffordanceInstances",
  "discovered": 464, "tested": 240,
  "excluded": [{ "count": 224, "reason": "POLICY_EXCLUDED",
    "basis": "sampled 30 of 65 routes, covering every route template and every distinct measured landmark set",
    "consequence": "Navigation on the 35 unsampled routes is INFERRED from template equivalence, not measured. A shell defect confined to one unsampled route would not be detected." }],
  "unaccounted": 0 }
```

The `basis` field is what makes this a sample rather than a guess, and the
`consequence` is what stops a reader mistaking it for full coverage. **A cap chosen
for cost is an exclusion; a sample chosen from structure is a method — and only the
second may be reported as coverage of the population.**

## Repeated route instances — owned by Phase 1

Route-instance sampling and frontier seeding are **discovery** behaviour, not planning
behaviour, and both are mode-gated. They live with the crawl that performs them:
[`discovery-profiling.md`](discovery-profiling.md) → *Route instances* and *Security Prep
Deep Crawl*. Planning consumes their output through the ledger, and **must not** re-derive
a sampling decision the crawl already recorded.

## Plan review tripwires — arithmetic, not opinion (Phase 4A)

Phase 4A `PLANNING_REVIEW` is advisory and often skipped, which leaves a plan's
exclusion decisions entirely unchallenged — in one run, **892 exclusions with no
second opinion**. A review does not need to re-reason about every decision. It needs
a small set of **deterministic tripwires computed from this ledger**, which catch the
same things a reviewer would and cost nothing:

| Tripwire | Fires when | What it is asking |
|---|---|---|
| `HIGH_EXCLUSION_RATIO` | `excluded / discovered > 0.8` in any population | is a whole population being written off? |
| `POPULATION_UNTESTED` | `discovered > 0` and `tested == 0` | did an entire class of artifact get no test at all? |
| `HIGH_UNUSED_RATIO` | `verifiedButUnused / verified > 0.7` | is the planner, not discovery, the bottleneck? |
| `BULK_SINGLE_REASON` | one exclusion reason covers `> 100` items | is one blanket justification doing too much work? |
| `SINGLE_ROUTE_GENERALISED` | a measurement from one route is applied to `> 10` routes | FM-2 — is a local observation being promoted to an invariant? |
| `NO_WORKFLOW_EXERCISED` | workflows discovered `> 0`, `EXERCISED == 0` | were only pages tested, never flows? |
| `INTENT_DROP_CLUSTER` | `NO_INTENT_MATCH > 0` on a single page | did one page's controls fail to bind as a group? |

Each fires as a **flag with its numbers**, written to `planning/plan-review.json` and
rendered in the report. Flags are **advisory** — `SKILL.md` states 4A is advisory, so
a tripwire never blocks a run. Its job is to move the plan from *unreviewed* to
*reviewed, with N flags a reader can weigh*.

### The two layers, and which one is authoritative

Phase 4A is **two layers with different authority**, and the split is already fixed by
the architecture — not invented here:

| Layer | Owner | Authority | Output |
|---|---|---|---|
| **Deterministic plan validation** | **`05` §32** | the objective **safety** layer | `PASS` · `WARNING` · `FAIL` |
| **AI planning review** | **`15`** | **advisory only** | `PASS` · `WARNING` · `FAIL` |
| Approve / reject / withhold | **`02`** | the only decider | — |

`docs/15` §2 states the boundary itself: the AI engine SHALL NOT *"Approve, reject, or
block a Test Plan (owned by 02)"* and SHALL NOT *"Perform Plan Validation (owned by 05
§32)"*. Its `FAIL` is a **Recommend Rejection**, and *"does not itself prevent
generation."*

> **A deterministic finding is measured evidence and is therefore append-only.** The AI
> layer may explain a finding, argue it is expected, or rank it — it may **never**
> remove, downgrade, re-score, or rewrite one. A consumer that drops a finding because
> the AI called it acceptable is non-conformant. This is `Rule 4` applied to the review
> itself: an interpretation never becomes a measurement.

Both layers write into **one** artifact, `planning/plan-review.json` — deterministic
findings under `deterministic.findings`, the advisory review under `aiReview`. Separate
keys, so neither can overwrite the other.

### How the AI layer is invoked — no AI client, by design

The skill contains **no model client and no SDK**, and must not acquire one: the
executing agent *is* the reviewer. Phase 4A is therefore an instruction, not a library
call:

1. Deterministic validation runs first and always, writing `deterministic` with
   `executed: true`. It never depends on the AI layer being available.
2. `aiReview.state` is initialised `NOT_REQUESTED`. It is **never** pre-filled and never
   defaulted to a pass.
3. The executor — reading this file at Phase 4A — reviews the deterministic findings
   together with `coverage/coverage-summary.json`, `planning/test-plan.json`, the
   declared exclusions and the write-gate configuration, then writes `aiReview` back
   into the same artifact.

The review answers exactly six questions, and its output MUST keep the three registers
apart:

```
MEASURED FACT     — a value copied from an artifact, with its path
AI INTERPRETATION — a judgement about that value, labelled as such
RECOMMENDATION    — a suggested action for a QA engineer
```

> An interpretation presented as a measured fact is the same defect as an authored
> assertion (Gate A). Every `MEASURED FACT` cites the artifact and path it came from;
> anything without a citation is an interpretation and is labelled one.

Questions: (1) is the plan reasonable · (2) is each deterministic finding **expected**
or **suspicious** · (3) what coverage gaps matter · (4) are exclusions justified by
configuration or policy · (5) does discovery, planning and intended execution agree ·
(6) what should a QA engineer look at first.

Per-finding, the review adds an `assessment` of `EXPECTED` or `SUSPICIOUS` plus its
reasoning — **alongside** the finding, never replacing its `status`.

### Absence is a state, never a pass

| `aiReview.state` | Means |
|---|---|
| `NOT_REQUESTED` | Phase 4A's advisory half did not run |
| `UNAVAILABLE` | requested, but no reviewer was available — with the reason |
| `COMPLETED` | a review was actually performed and is present |

`UNAVAILABLE` and `NOT_REQUESTED` are **never** rendered as approval, and a missing
review never upgrades the deterministic verdict. Fabricating a review — or defaulting
it to `PASS` — is the `Rule 4` violation this whole section exists to prevent.

### Why deterministic first

These tripwires are preferred to a judgement-based reviewer as the *safety* layer: they
are deterministic, so identical input yields identical flags (`PLAYBOOK` §19), they need
no second model, and they cannot invent a concern. A reviewer reasoning freely over
hundreds of exclusions introduces exactly the authored-judgement problem `Rule 4` keeps
out of this skill. The AI layer adds what arithmetic cannot: whether a measured number
is *expected here*.

## Test tags — the suite must be runnable in useful subsets

A suite that can only be run whole is run rarely. Generation therefore emits a
**tag** per test, derived from fields the plan already holds — never hand-assigned:

| Tag | Derived from | Purpose |
|---|---|---|
| `@<category>` | the case's planned category | run one area (`@api`, `@navigation`, …) |
| `@critical` | cases on the authentication, primary-navigation and workflow paths | the pre-commit subset |
| `@workflow` | `isFormWorkflow` / multi-step cases | exercise flows only |
| `@write` | `mutates: true` | include or exclude every mutating test in one switch |
| `@slow` | **measured** duration above the suite's p90 | opt out of the long tail |

Two rules keep tags honest:

1. **A tag is derived, never authored.** It restates a field the plan already
   carries. A hand-written tag is a second, unverifiable classification.
2. **`@slow` is emitted only from a measurement, and only when it changes
   anything.** Deriving it from a guess — "detail pages feel slow" — produces a tag
   that excludes tests for no gain. Where durations are near-uniform, the honest
   outcome is **no `@slow` tag at all**: a run whose slowest 5 tests account for 6%
   of wall-clock has no long tail to trim, and pretending otherwise invites someone
   to skip tests believing they saved time.

Report the tag inventory with the per-tag test count **and measured duration**, so a
reader can choose a subset on evidence rather than on the tag's name.

A headline figure such as "121 passed" is **incomplete** unless accompanied by
its ledger. Pass counts describe the suite; the ledger describes the application.

