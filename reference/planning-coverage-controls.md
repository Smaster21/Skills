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

> Prefer these tripwires to a judgement-based reviewer. They are deterministic, so
> identical input produces identical flags (`PLAYBOOK` §19), they need no second
> model, and they cannot invent a concern. A reviewer that reasons freely over 892
> decisions introduces exactly the authored-judgement problem `Rule 4` exists to
> keep out of this skill.

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

