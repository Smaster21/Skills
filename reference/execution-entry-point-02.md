<!-- PRESERVED VERBATIM (part 2 of 7). Split ONLY to satisfy the 200-line
     reference cap; split boundaries never fall inside a fenced block, and no
     sentence was altered, reordered, or removed. Preservation note: part 1. -->
## Optional Engines — Normative When Present

Absence changes **cost or assurance, never correctness** (`01` §16).

| # | Document | Owns | On absence |
|---|---|---|---|
| 08 | [08_Self_Healing_Engine.md](../docs/08_Self_Healing_Engine.md) | Self-healing · Locator evolution · Healing Candidate Score *(engine-local)* | No healing; failures reported |
| 10 | [10_AI_Learning_Repository.md](../docs/10_AI_Learning_Repository.md) | Learning Database | No historical evidence |
| 11 | [11_Unified_Test_Intelligence_Engine.md](../docs/11_Unified_Test_Intelligence_Engine.md) | Unified Intelligence Projection *(derived)* · **Framework Cache Service** | Consume `04` directly |
| 12 | [12_Adaptive_Execution_Optimizer.md](../docs/12_Adaptive_Execution_Optimizer.md) | Execution Optimization Proposal *(advisory)* · Batch planning · Runtime prediction | `07` executes plan order |
| 13 | [13_Incremental_Discovery_Engine.md](../docs/13_Incremental_Discovery_Engine.md) | Discovery Delta · Fingerprinting | Full discovery |
| 14 | [14_Confidence_Coverage_Optimizer.md](../docs/14_Confidence_Coverage_Optimizer.md) | Coverage Recommendations · Stop-condition recommendation | Full plan executes |
| 15 | [15_AI_Planning_Review_Engine.md](../docs/15_AI_Planning_Review_Engine.md) | Planning Review *(advisory)* | Plan unreviewed, flagged |

## Supporting Documents — Non-Normative Operational Playbooks

Neither is an engine. Neither adds a state, a phase, or any ownership. The Engine
Registry remains **01–15**.

| Document | Layer | Answers |
|---|---|---|
| [16_QA_Execution_Playbook.md](../docs/16_QA_Execution_Playbook.md) | Operational QA doctrine | **WHAT** good QA behaviour looks like — environment intelligence, workflow-first validation, evidence standards, runtime adaptation, reporting integrity |
| [IMPLEMENTATION_PLAYBOOK.md](../docs/IMPLEMENTATION_PLAYBOOK.md) | Playwright implementation | **HOW** to build it — scaffold, config, discovery tactics, retry mechanics, ZAP tiers, hard-won rules, manual mode (§17), health pre-flight (§18) |

Consult both for judgement and tactics. Neither for authority.

---

# 2. Precedence

```
01_Master_Architecture              ← highest architectural authority
        ↓                             architecture, principles, lifecycle
Architecture_Ownership_Matrix       ← governance contract
        ↓                             who owns what
02 … 15                             ← engine specifications
        ↓
16_QA_Execution_Playbook            ← non-normative: operational doctrine (WHAT)
IMPLEMENTATION_PLAYBOOK             ← non-normative: implementation (HOW)
        ↓
SKILL.md                            ← orchestration only (WHEN)
```

- `01` SHALL take precedence over every other document.
- The **Matrix** governs *ownership allocation only*. It does not define architecture.
- Where an engine document conflicts with `01`, **`01` wins**.
- Where an engine document conflicts with the Matrix on *who owns what*, **the Matrix wins** until `01` is amended.
- Where the Matrix conflicts with an architectural principle in `01`, **`01` wins** and the Matrix is defective.
- Where two engine documents conflict, **the lower number wins**.
- Neither playbook overrides a numbered document. If one appears to, that playbook is defective.
- **This file overrides nothing.**
- No document authorises skipping a lifecycle stage or a mandatory quality gate.

---

# 3. Ownership Resolution

**This file never resolves ownership.** When ownership is unclear:

1. Read `Architecture_Ownership_Matrix.md` — 37 capabilities, 14 canonical datasets.
2. If the Matrix is silent or ambiguous, read `01` §37 and §42.
3. If both are silent, **stop and report**. Do not infer an owner.

## Two Authorities Only

```
02 Decision Engine   →  decides WHAT happens
07 Execution Engine  →  decides HOW it runs at runtime
```

Every other engine produces knowledge, plans, artifacts, or recommendations.
None decides. None executes.

## Routing Table — Every Operation Has One Destination

| Operation | Routes to |
|---|---|
| Any decision · risk · Framework Confidence · policy · conflict | **02** |
| Any knowledge operation · graph read, version, diff | **04** |
| Any planning operation · canonical coverage · dependencies · retry policy | **05** |
| Any generation operation · regeneration scope · duplicate detection | **06** |
| Any runtime execution · scheduling · browsers · workers · timeouts | **07** |
| Any healing operation · locator history | **08** |
| Any reporting operation · analytics | **09** |
| Any learning operation · historical evidence | **10** |
| Cache storage, eviction, TTL *(region semantics stay with the region owner)* | **11** |
| Optimization advice · batch plan · runtime prediction | **12** |
| Change detection · fingerprinting | **13** |
| Coverage optimization · stop recommendation | **14** |
| Plan review | **15** |
| Operational judgement — *how should a QA engineer behave here?* | **16** |
| Implementation tactics — *how do I write this in Playwright?* | **PLAYBOOK** |

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

---

<!-- nav -->
*← [part 1](execution-entry-point-01.md)  ·  part 2 of 7  ·  [part 3](execution-entry-point-03.md) →*  ·  [reference index](INDEX.md)
