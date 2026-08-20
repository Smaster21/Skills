# Write Operations and Test Data

> Executor reference. **Mandatory before planning any write (Phase 4).** `SKILL.md` declares
> `CRUD-except-Delete` and `business-rule` as coverage areas, but no document said
> **how** to perform a write safely. Six executions produced zero write tests as a
> result — a declared area silently delivered as nothing.

## The governing control already exists — `ALLOW_WRITE_TESTS`

> **Correction on record.** An earlier revision of this file introduced a
> `TEST_ENV_CLASS` variable to gate writes. That was a **second competing control**
> for a decision the framework already governs, and it is **withdrawn**.

Writes are governed by **`ALLOW_WRITE_TESTS`** (`01` §29 configuration hierarchy,
ratified by **W7-A BD-W7-3**):

> `ALLOW_WRITE_TESTS=1` permits state-mutating QA tests. Requires **synthetic
> data**, **mandatory cleanup**, **scope + RoE**, and **disclosed mutation**.
> Never enables application-delete tests.

Absent or `0` ⇒ **no write test is planned or generated.** The affected cases
become coverage-ledger entries, never silent omissions.

Note what this resolves: cleanup is of the run's own **synthetic data**, which is
why "mandatory cleanup" and "never enables application-delete tests" coexist
without contradiction. An earlier revision of this file claimed the two were
irreconcilable and that every create was permanent residue — that claim was made
against an incomplete working tree and is **withdrawn**.

### When writes are refused

The affected cases are **not** silently dropped. They become coverage-ledger
entries:

```
reason:      POLICY_EXCLUDED
consequence: "Create/update business rules are UNTESTED. Validation, persistence
              and uniqueness constraints remain unverified; treat them as
              untested, not as passing."
```

That consequence line is mandatory. "Excluded" without it lets a reader assume the
area was fine.

## Step 2 — identifying data the suite created

A write test must be able to recognise its own records, on a target that already
contains other data.

- Every created record carries a **run-scoped marker** in a free-text field —
  derived from the Execution ID, the worker index, and the case ID.
- The marker is **deterministic**: identical inputs produce identical values
  (`PLAYBOOK` §19 determinism). No randomness, no wall-clock value used as a
  magnitude.
- Determinism plus uniqueness is achieved by *composition*, not entropy:
  `<executionId>-<workerIndex>-<caseId>` is unique across workers and runs while
  remaining reproducible.

> **Never modify or delete a record the suite did not create.** A write test
> operates only on data carrying its own run marker. Editing a pre-existing record
> is indistinguishable from damaging real data.

## Step 3 — the write workflow

A write case is a workflow with a verified outcome, not a form submission:

```
populate every field from the verified inventory
  → submit
  → assert the success signal observed in the assertion-target probes
  → re-query the record and assert it PERSISTED with the values sent
```

The re-query matters. A success toast proves the UI reacted; only reading the
record back proves the write happened. Assert the persisted values, and cite the
probe that established what a success signal looks like on this target
(`suite-self-validation.md` Gate A).

### Update

An update case operates **only** on a record the same run created. It asserts the
changed field persisted *and* that untouched fields were not altered — silent
collateral modification is a real defect class and is invisible unless asserted.

### Business rules

Derive these from the target, never from assumption:

- required-field enforcement, from validation messages captured in their
  **triggered** state;
- uniqueness constraints, by submitting a value the run already created and
  asserting the target's own rejection;
- field limits (`maxlength`, format), from the measured attributes of the control.

Each assertion cites the probe it came from. A business rule that was never
observed is not tested — it is guessed.

## Step 4 — residue accounting

Whatever the outcome, a run that mutated state reports what it left (disclosed mutation, W7-A BD-W7-3):

```
Records created   12   (marker EXEC-20260818-0001-*)
Records updated    4   (all self-created)
Records cleaned   12   (synthetic data removed by the run's own cleanup)
Residue            0   (anything not cleaned is listed explicitly)
```

Cleanup is mandatory for synthetic data the run created; anything that could not
be cleaned is **stated**, never discovered later by someone else.

## Parallelism

Write cases are worker-scoped: the marker includes the worker index so parallel
workers cannot collide. Where a target enforces global uniqueness on a field the
suite must populate, write cases for that field run **serially** — recorded as a
scheduling decision, not left to chance.

## What this changes

| Phase | Obligation |
|---|---|
| 0 Config | resolve `ALLOW_WRITE_TESTS` (W7-A BD-W7-3); refuse all writes unless it is `1`, and verify scope + RoE permit mutation |
| 3 Verification | probe success signals and validation messages in triggered state |
| 4 Planning | plan writes only when permitted; otherwise ledger `POLICY_EXCLUDED` **with consequence** |
| 5 Generation | emit populate → submit → assert-signal → **re-query and assert persisted** |
| 9/10 Reporting | publish disclosed-mutation + residue accounting (W7-A BD-W7-3) |
