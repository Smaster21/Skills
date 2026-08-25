# Environment Contract and Scope Enforcement

> Executor reference. The `./output/qa/` artifact tree is in
> `output-and-scope.md`; this file carries the environment variables a run reads
> and the scope check that precedes any network action.

## Environment contract — non-interactive

Reads context from environment variables only; no prompt ever blocks a run.

| Variable | Required | Meaning |
|---|:---:|---|
| `OUTPUT_DIR` | ✅ | Root output dir; this skill writes under `$OUTPUT_DIR/qa/` |
| `BASE_URL` | ✅* | Full URL to the app entry point |
| `TARGET` / `TARGET_DOMAIN` | ✅* | Accepted as `BASE_URL` fallback (coordinator spelling) |
| `SCOPE_FILE` | ⛒ | If present, every target validated against it before navigation |
| `ROE_FILE` | ⛒ | If present, honor time-based restrictions / blackout / expiry |
| `TEST_USER` / `TEST_PASS` | ❌ | Present → **Authenticated Exploration**; absent → **Unauthenticated Exploration** (narrows scope, never stops). Scope/authorization is mandatory in both modes |
| `ALLOW_SAFE_WRITES` | ❌ | `1` permits non-`GET` requests **proven** to change no observable state. Absent ⇒ `GET` only |
| `ALLOW_WRITE_TESTS` | ❌ | `1` permits state-mutating tests on **reversible** surfaces only — synthetic data, run-scoped markers, mandatory cleanup, disclosed mutation. Absent or `0` ⇒ no write test is planned |
| `ALLOW_IRREVERSIBLE_SURFACES` | ❌ | Comma-separated **surface ids** explicitly opted in, each of which has no inverse affordance. Never a wildcard. Absent ⇒ every irreversible surface is ledgered, not run |
| `MAX_ROUTE_INSTANCES` | ❌ | Per-template instance ceiling for the exhaustive crawl. Absent ⇒ the built-in default. Reaching it is a **declared exclusion**, never a silent stop |
| `CI` | ❌ | Fewer workers, more retries |

\* At least one of `BASE_URL` / `TARGET` / `TARGET_DOMAIN` must resolve to a URL.

The three write controls form **one escalating ladder**, each implying the one above
it, and none is a second competing gate (`write-operations-and-test-data.md`):

```
(default)                      → READ only
ALLOW_SAFE_WRITES=1            → + non-GET that changed nothing, proven per request
ALLOW_WRITE_TESTS=1            → + create/update on surfaces with an inverse
ALLOW_IRREVERSIBLE_SURFACES=…  → + only the named surfaces, one id at a time
```

Application-delete **feature** tests are enabled by no value of any of these.

Credentials arrive via env only and are **masked before any persistence**
(`01` §30, `07` §41). No secret reaches a log, report, screenshot, trace, or
artifact.

### The target is read from the environment — never defaulted, anywhere

`BASE_URL` resolution happens **once**, at Phase 0, and a missing value **aborts**:

```bash
BASE_URL="${BASE_URL:-$TARGET}"
[ -z "$BASE_URL" ] && { echo "[ABORT] no BASE_URL/TARGET"; exit 2; }
```

That rule binds **every artifact the run generates**, not just the shell:

> **No generated file may embed a default, fallback, or previously-measured target.**
> A runner config, fixture, page object, helper or spec MUST read the target from the
> environment and **fail when it is absent**. A construct of the form
> `process.env.BASE_URL ?? "<some measured origin>"` is a defect the moment it is
> written, even though it works on the target it was measured against.

Why this is not a style preference:

- The measured origin is correct **only for the run that measured it.** Baking it in
  makes the artifact silently correct-looking on a target it was never validated for.
- A later execution with `BASE_URL` unset then produces a **complete, green report
  about the wrong application** — and nothing in the run signals it, because every
  assertion passes against the site the fallback points at.
- Where runs are dispatched per job — one target per job, many targets over time — a
  stale fallback is a **cross-target contamination** path: job B silently exercises
  job A's application.

Correct form, in any generated file:

```
target = <read from environment>
if target is absent → abort with a named error; never continue with a literal
```

The same prohibition applies to `SCOPE_FILE`, `ROE_FILE` and credentials: an absent
input is a **named state that stops or narrows the run** (`Rule 6`), never a value
substituted from a previous measurement. This is `Rule 4` — *evidence is observed,
never authored* — applied to configuration: a target that was not supplied for **this**
run has not been authorized for this run.

## Crawl behaviour — exhaustive by default, and there is no other mode

This skill has **one** crawl behaviour: visit **every discovered route instance**, up to
the declared `MAX_ROUTE_INSTANCES` ceiling. There is no sampling mode, no mode switch,
and no configuration that reduces instance coverage.

| Rule | Meaning |
|---|---|
| Every discovered instance is crawled | up to the declared ceiling, and no further reason to stop exists |
| Template dedupe is **loop protection only** | its sole job is to stop a crawl that would never terminate; "this template has been seen" is **never** permission to skip an instance |
| Convergence early-stop is **not allowed** | a repeated structural signature is evidence about *structure*; the crawl is collecting *content*, so it never ends the crawl |
| `INSTANCE_SKIPPED_BY_SAMPLING` is **retired** | it is not a valid outcome and MUST NOT be emitted (`coverage-ledger.md`) |

An instance goes uncrawled for **exactly five** reasons, each recorded with its
machine-readable state and nothing else:

| Reason | Ledger state |
|---|---|
| the declared ceiling was reached | `INSTANCE_EXCLUDED_BY_CAP` |
| outside the authorized origin | `OUT_OF_ORIGIN` |
| unsafe to navigate by the structural filters | `UNSAFE_TO_NAVIGATE` |
| an explicit policy decision excludes it | `INSTANCE_EXCLUDED_BY_POLICY` |
| the URL was already visited | not an exclusion — the same URL is **one** route, counted once |

### Deeper discovery is not broader authority

A deeper crawl visits more routes and records more affordances so that this run — and
any skill that later reads its output — has a fuller map. It is **not** permission to act.

> **Crawl depth grants no authority whatsoever.** It does not relax the read-only
> boundary, does not imply `ALLOW_SAFE_WRITES`, `ALLOW_WRITE_TESTS` or any irreversible
> surface id, and never accepts a wildcard. The target requires exactly the same
> authorization, scope and RoE it always did. **Discovering an endpoint is not a reason
> to call it.**

| DO | Still DO NOT |
|---|---|
| visit every discovered route instance within caps | submit a form, unless the write ladder allows that surface |
| record forms, parameters, APIs, links, buttons | activate a state-changing affordance without authorization |
| record a state-changing affordance **as a fact of structure** | call a discovered endpoint merely because it was discovered |
| follow disclosures that profiling proved reveal routes | create, update or delete anything the ladder has not opened |

A state-changing affordance is **catalogued, not exercised**: recording that
`GET /basket/add/{id}` exists is discovery; issuing it is a write, and the write ladder
alone decides that.

### Depth does not make this a security skill

Output remains QA-only (`Rule 21`): no vulnerability finding, no CVSS, no
exploitability, no severity, no security coverage, and no judgement that a target is or
is not secure. A separate security skill may read `qa/discovery/` and `qa/network/`;
producing a fuller map does not make this skill one.

## Scope enforcement — before any network action (non-negotiable)

```bash
# Validate the target (and its resolved IP) against SCOPE_FILE if provided.
if [ -f "$SCOPE_FILE" ]; then
  host=$(echo "${BASE_URL:-$TARGET}" | sed -E 's#^[a-z]+://##; s#/.*$##; s#:.*$##')
  if ! grep -qE "$(echo "$host" | sed 's/\./\\./g')" "$SCOPE_FILE"; then
    ip=$(dig +short "$host" | head -1)
    if [ -z "$ip" ] || ! grep -qE "$(echo "$ip" | sed 's/\./\\./g')" "$SCOPE_FILE"; then
      echo "[ABORT] $host (or its IP) not in SCOPE_FILE" >&2
      exit 1
    fi
  fi
fi
```

If `SCOPE_FILE` is absent, fall back to the skill's own rule: proceed **only**
against an explicitly authorized target, and STOP with `BLOCKED` if the target
or its authorization is missing or ambiguous. Never discover, invent, or
substitute a target.

## Cleanup and determinism

- Close browsers/contexts; leave no lingering process, temp file, or open
  connection on exit.
- Deterministic output for identical input: no unseeded randomness, no
  wall-clock value used as a magnitude (`01` §3.1, `PLAYBOOK` §19 determinism).
