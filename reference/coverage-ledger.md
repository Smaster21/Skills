# The Coverage Ledger — nothing disappears quietly

> Executor reference. **Mandatory artifact and gate.** Every other fix in this
> skill repairs a past mistake. This one prevents the next one.

## Why this exists

A real run reported **121 tests passed** while, unreported:

- 37 of 58 discovered API endpoints were dropped by a `slice(0, 20)` nobody
  declared;
- 116 of 169 verified elements were never referenced by any test;
- every control lacking `id`, `name`, and `placeholder` was discarded during
  extraction without a word;
- one test **passed** while driving an element other than the one it named.

None of that was visible in the report. The run looked complete because the only
numbers shown were about tests that ran, never about work that vanished. A suite
that cannot say what it skipped cannot be trusted about what it covered.

## The rule

> **Every artifact discovered on the target MUST terminate in exactly one of two
> states: `TESTED`, or `EXCLUDED` with a machine-readable reason.**
> A third state — silently absent — is a defect of the run, not a detail.

## Ledger scope

Account for every one of these populations:

| Population | Source |
|---|---|
| Routes / pages | `03` discovery |
| Form controls (incl. un-nameable ones) | `03` discovery |
| Choice widgets (native and custom) | `03` profiling + discovery |
| Actionable controls (buttons, submits, menu items) | `03` discovery |
| Forms | `03` discovery |
| Tables | `03` discovery |
| API endpoints, per method | `03` network observation |
| Assertion targets | `03` §29 triggered-state probes |

For each population record: `discovered`, `tested`, `excluded`, and an
`unaccounted` count that **must be zero**.

## Exclusion reasons — closed vocabulary

An exclusion carries one of these. Free text alone is not acceptable, because
free text cannot be counted or audited.

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

`POLICY_EXCLUDED` additionally requires a `consequence` field saying plainly what
is therefore unverified (for example: *"create/update business rules are
UNTESTED and must not be read as passing"*).

## Caps are forbidden

There is **no** URL cap, no per-category cap, no `slice(n)`, no "first N".

If some limit is genuinely unavoidable, it is not a cap — it is an exclusion, and
it is declared per item with a reason. A truncation that produces no ledger entry
is the specific failure this document exists to stop.

## The gate

Reporting (`09`) **SHALL**:

1. Refuse to present a run as complete while `unaccounted > 0` in any population.
2. Publish `tested` / `discovered` per population, not just test pass counts.
3. Surface `NO_INTENT_MATCH` and `UN_TARGETABLE` prominently — those are the
   states that previously hid real coverage loss.
4. Report a **verified-but-unused** count (elements that passed the locator gate
   and no test references). A large value means the planner, not discovery, is
   the bottleneck.

A headline figure such as "121 passed" is **incomplete** unless accompanied by
its ledger. Pass counts describe the suite; the ledger describes the application.

## Artifact

Write `qa/coverage/coverage-summary.json`, and render its summary in every report
format alongside the pass/fail totals.

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

Read against that, "121 passed" is no longer a claim about the application — it
is a claim about the suite, and the ledger says how much of the application the
suite actually reached.
