# Known Failure Modes — mistakes already made, and the rules that prevent them

> Executor reference. **Mandatory before building any phase.** Every entry below is a real defect from a
> real execution, with its measured cost. None was caught by the type gate, the
> locator gate, or a passing test suite — which is precisely why each needs a
> rule rather than good intentions.
>
> Across six executions of one plan, **92 test failures occurred and not one was
> a defect in the target application.** Every single one was a defect in the
> suite or the pipeline that built it. Treat that as the base rate.

---

## FM-1 · Mixed auth states in one spec file — 21 failures

A spec file contained both anonymous and authenticated tests. A file-scoped
`test.use({ storageState: <empty> })` applies to the **whole file**, so it
stripped the session from every authenticated test in it. Twenty API tests and
one smoke test failed with `401` / redirect-to-login.

The trap: the endpoints were fine. `401` produces the same assertion text a
genuine authorization bug would, so it read as an application defect.

> **RULE.** One authentication state per spec file. A fixture-level or
> file-level state override applies to the entire file — never place an
> anonymous test and an authenticated test in the same file. Emit separate
> files (for example `x.spec.ts` and `x-anon.spec.ts`).

---

## FM-2 · A probe measured on one route, asserted on all — 13 failures

A shell anchor was probed on the landing route, resolved to exactly one element,
and was promoted to a global "page loaded" invariant. Measured **per route**, the
same anchor legitimately resolved to **0, 1, or 2** elements. Thirteen tests
failed against correct application behaviour.

> **RULE.** A baseline measured on one route is evidence about **that route
> only**. Persist assertion baselines per route and assert the measured value for
> the route under test. Never generalise a single-route observation into a global
> invariant.

---

## FM-3 · Expectations built from observed requests, not observed responses — 3 failures

Discovery recorded which API calls the application makes, but never what came
back. The planner therefore asserted `200` everywhere. Two endpoints require
query parameters and correctly answer `422` without them; one is an optional
sub-resource the application itself receives `404` for.

Worse, the observed **query strings were captured in the knowledge graph and then
discarded** by a planner that keyed on `method + path` alone.

> **RULE.** An expectation must come from a measured **response**, never from the
> existence of a **request**. Take it from W7-C's **already-captured** response
> evidence, which records status / content-type / body shape against the request
> actually issued. Never assume `200`.
>
> **Do not bulk-replay the endpoint inventory to obtain responses** — that is
> duplicate API collection and violates the W7 contract. Where a response is
> `NOT_OBSERVED` and an expectation genuinely needs one, run a **targeted**
> verification probe recorded as `discoverySource: verification`, correlated like
> any other call (`w7-api-evidence-contract.md`).

---

## FM-4 · Session lost mid-run, unexplained — 50 failures

A shared storage state was minted once and reused. Partway through a run the
session was invalidated and every subsequent test using it failed with one
signature: landed on the login page. The split was perfectly by session strategy
— anonymous 3/3 passed, own-fresh-login 6/6 passed, shared-state **0/38**.

Root cause was **never established**. Single-session-per-user was refuted (three
concurrent sessions stayed alive); the logout test was refuted by ordering (five
shared-state tests passed after it). Measured only as: target-side, intermittent,
recoverable by a fresh login.

> **RULE.** Never assume a session survives a whole run. Detect the
> landed-on-login condition, re-authenticate in the fixture, and retry the
> navigation once. **Every recovery MUST be recorded and reported** — a recovered
> pass is not a clean pass. A run reporting zero recoveries has not demonstrated
> the mechanism works; say so rather than implying resilience.

---

## FM-5 · Failure classification from text alone — 22 misclassifications

The classifier read failure **text**. An API `401` and a real authorization bug
produce the same `expect(...)` string, so 22 suite defects were filed as
`TARGET_DEFECT_CANDIDATE`. Publishing that would have reported 22 fabricated
application defects to a client.

> **RULE.** A failure may be classified as a **target defect** only when a probe
> measured against the live target supports it. Where a follow-up probe
> establishes the true cause, that measurement **overrides** the text heuristic,
> and the classification record MUST name the artifact proving it. Absent
> evidence, classify `UNCLASSIFIED` — never default to blaming the target.

---

## FM-6 · Waiting on the wrong signal — 2 failures

A search test waited for "any response from the list endpoint". That endpoint
also fires on page load, so the wait resolved on the **initial** request and the
assertion ran before the filtered results arrived. The banner it looked for was
proven present by direct measurement — the test simply looked too early.

> **RULE.** Wait for the response **caused by the action under test**, identified
> by its distinguishing parameters — not merely for a matching path, and not for
> a bare `networkidle`.

---

## FM-7 · Evidence overwritten and identifiers collided

Two discovery passes wrote to the same filenames; the first pass's report was
destroyed. Separately, two passes each numbered candidates from `0001`, so
identifiers collided and broke the compile gate.

> **RULE.** Artifact filenames are scoped by the pass and run that produced them.
> Identifiers are namespaced by their producing pass. A previous artifact is
> immutable evidence — a new pass writes a new name, never over an old one.

---

## FM-8 · Framework Confidence permanently unavailable

Across **six** executions, Framework Confidence was refused every time:
`UNAVAILABLE: NO_HISTORY_CONFIGURATION`. Three of four terms were measured; only
Historical Reliability was missing, because no Learning Repository store existed
for the scope and the recency window `W` / `minimumExecutions` were never
configured.

The refusal is correct behaviour (`PLAYBOOK` §20 forbids substituting a default),
but a term that can *never* become available is a configuration defect, not a
principled unavailability.

> **RULE.** Phase 0 MUST resolve a Learning Repository store for the Execution
> Scope Identity and record `W` and `minimumExecutions` in the configuration
> hierarchy. Each execution MUST persist its **terminal lifecycle state**
> (`01` §17) and its scope identity, so later runs have eligible history. If the
> store cannot be provisioned, report that as a **configuration gap**, not as a
> neutral unavailable measurement.

---

## FM-9 · Destructive-control detection assumed English

The safety pattern blocking destructive clicks matched English words. On a target
localised in another language, a delete or save control would **not** match, and
read-only discovery could mutate data.

> **RULE.** Never rely on a text lexicon alone for safety. Prefer structural
> signals — the control's owning form `method`, `type="submit"`, ARIA role, and
> whether activation triggers a non-`GET` request. Treat text matching as an
> additional filter, never the primary guard. When intent cannot be established
> structurally, **do not activate the control**.

---

## FM-10 · Portability claimed without a second target

The pipeline was built and validated against one application. A claim that it
works "for any site" was not tested until it was measured — at which point it
held **126 hand-written selectors** for that one application.

> **RULE.** A site-agnostic claim is `NOT DEMONSTRATED` until the same skill has
> run end to end against a **second, structurally different** target. Report the
> validation tier honestly: specification, implementation, or runtime — and never
> present the first as the third.

---

## FM-11 · A planned interaction dropped by title-based dispatch — 1 failure, misattributed

The generator chose a case's interaction by pattern-matching its **generated title**. Of two sibling cases on one feature, the differently-phrased one matched nothing, fell through to the plain-route emitter, and its planned `interaction` was **silently discarded**: the emitted test opened the route and asserted a value measured *after* the submit it never made. Every gate passed it — the code compiled, Gate A's provenance was intact while the **body** ignored it, Gate B scored only the perturbed run (which the test failed unperturbed too), and Gate C then blamed the target, its probe re-measuring the route without replaying the interaction — one step before a suite defect was published as someone's application defect.

> **RULE.** Generation dispatches on **structural fields of the plan**, never a title or any generated prose — an explicit `interaction.kind` from a **closed vocabulary of mechanical shapes**, never a business concept. A case whose interaction reaches no emitter **fails loudly** rather than being emitted without it.
>
> **RULE.** Every gate MUST compare **two** things; each above passed on one side only. Per `suite-gates-extended.md`: **A2/A3** plan vs emitted source and assertion vs state reached; **B2** perturbed vs normal run; **C** replay before re-measuring, else `UNCLASSIFIED`.

---

## Reading this list

Ten of these eleven produced **passing type checks and a green-looking pipeline** while being wrong; three produced *passing tests* asserting the wrong thing, and FM-11 a **failing** test blamed on the target. A suite cannot validate itself: only a measurement against the live target distinguishes "my expectation was wrong" from "the application is broken", and the default assumption must always be the former. FM-11's corollary: **a gate inspecting one side of a comparison passes work that contradicts itself.**
