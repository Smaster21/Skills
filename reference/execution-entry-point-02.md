<!-- PRESERVED VERBATIM (part 2 of 6). Split ONLY to satisfy the 200-line
     reference cap; no sentence was altered, reordered, or removed. The original
     preservation note is retained at the top of part 1. -->
## Framework Confidence vs Engine-Local Scores

**02 owns the single Framework Confidence model.** An engine MAY compute an
internal domain score — Healing Candidate Score (`08`), Test Value Score (`14`),
Review Score (`15`). Such a score is **evidence submitted to 02**. It SHALL NEVER
be reported as confidence, override Framework Confidence, or justify a decision.

---

# 3A. QA Reasoning Layer

## What This Is

An **internal reasoning capability of this orchestrator**. It is the deliberate
thinking a Senior QA Engineer does *before* raising a request — so that what
reaches an owner is a considered proposal with evidence, not a reflex.

It runs **silently**, before major requests are routed.

## What This Is Not

| Not | Because |
|---|---|
| An engine | The Engine Registry remains **01–15** (`01` §38). No number is assigned. |
| A document | It lives here. Nothing is added to the registry. |
| A lifecycle phase | Phase dispatch (§5) is unchanged — still 16 phases. |
| A state | The state machine (§4) is unchanged — still 21 states. |
| An owner | It owns no capability and produces no canonical dataset. |
| A canonical artifact | Its output is transient input, absorbed into the receiving owner's evidence and audit record (§below). |

It changes **no** ownership, **no** authority, and **no** routing. Every request
still travels to the same owner it did before, through the §3 routing table.

---

## When It Runs

Before each of these requests is routed — never instead of routing it:

| Before | Package goes to | Which then routes to |
|---|---|---|
| Discovery decisions | **02** | 03 / 13 |
| Planning decisions | **02** | 05 |
| Test prioritization | **02** | 05, with 14's input |
| Test generation requests | **02** | 05 → 15 → 06 |
| Runtime re-planning requests | **02** | 05 → 15 → 06 → 07 |
| Healing requests | **02** | 08 |
| Reporting conclusions | **09** | — |
| Learning recommendations | **02** | 10 |

Note that no arrow terminates at the reasoning layer. It is always upstream of a
decision, never a substitute for one.

---

## What It Reasons About

**Objective**

• Business objective · • User objective · • Functional objective
• Risk reduction *(risk values consumed from `02`)*

**Evidence**

• Evidence required · • Existing evidence · • Missing evidence
• Assumptions, and whether each is verifiable

**Hypotheses**

• Failure hypotheses — what would break, and how it would present
• Alternative validation paths — cheaper or stronger ways to obtain the same proof

**Economics**

• Cost versus value · • Expected confidence improvement
• Expected coverage improvement

---

## What It SHALL NEVER Do

It SHALL NEVER own decisions, planning, execution, healing, reporting, or
learning.

It SHALL NEVER calculate **Framework Confidence** or **Framework Risk**.

It SHALL NEVER bypass the Decision Engine.

Every value it reasons with is **consumed from its owner**:

| Value | Consumed from | Contract |
|---|---|---|
| Framework Risk | **02** | §20 Risk Engine |
| Framework Confidence | **02** | §21 Confidence Engine |
| Canonical coverage | **05** | §18 Coverage Analysis |
| Coverage gaps, test value | **14** | Coverage Recommendations |
| Execution cost, runtime prediction | **12** | §26 Runtime Predictor |
| Application model | **04** | Graph query contract |
| Historical evidence | **10** / **09** | Learning and analytics contracts |
| Duplicate detection | **06** | §32 |

Where a required value is unavailable, the layer SHALL state the gap. It SHALL
NOT substitute an estimate of its own for an owner's value.

---

## Output — The Reasoning Package

Every recommendation SHALL contain exactly these five fields:

| Field | Content |
|---|---|
| **Reasoning** | Why this request, in terms of the objective it serves |
| **Supporting evidence** | Concrete observations, cited by source and owner |
| **Uncertainty** | What remains unknown, and what would resolve it |
| **Expected confidence improvement** | Estimated, as evidence for `02` — never a Framework Confidence value |
| **Expected execution cost** | Consumed from `12` where available; otherwise stated as unavailable |

A package with an empty evidence field SHALL NOT be submitted.

### Not A Canonical Artifact

The package is **transient**. It is submitted to the receiving owner and absorbed
into that owner's existing evidence and audit record — `02` §36 Evidence Model
and §39 Decision Audit for decisions, `09` for reporting conclusions.

It is **not** persisted as a dataset of its own, appears in no Canonical Data
Ownership row, and creates no new artifact. The Matrix is unchanged.

### Estimates Are Evidence, Not Values

The two estimated fields follow the engine-local score precedent already
established for `08`, `14`, and `15`:

- They are **evidence submitted to an owner**
- They SHALL NEVER be reported as Framework Confidence or Framework Risk
- They SHALL NEVER override an owner's value
- Where an estimate and an owner's value disagree, **the owner's value prevails**

---

## Visibility

The layer is **invisible by default**. It produces no user-facing output.

Its reasoning becomes visible only when verbose or debug output is enabled,
resolved through the configuration hierarchy owned by the Configuration Layer
(`01` §29). This orchestrator defines no configuration option of its own.

When visible, reasoning SHALL be shown as the package's five fields — never as
narration, and never as a substitute for the owner's recorded decision.

---

## Determinism

Identical inputs SHALL produce an identical package (`01` §3.1).

The layer SHALL NOT introduce randomness, wall-clock dependence, or
completion-order dependence. Where it reasons during execution, it does so at the
same deterministic checkpoints `14` uses (§5, Phase 7) — never per test
completion.

## Failure Mode

Where evidence is insufficient to reason, the layer SHALL **refuse to
recommend** — consistent with `02` §5.5 Safe Failure.

It SHALL then route the underlying request with the evidence gap stated, so the
owner decides on a known-incomplete picture rather than a fabricated one.

Refusing to recommend SHALL NEVER block a request from reaching its owner.

---
