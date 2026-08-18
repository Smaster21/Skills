# Output Discipline, Environment Contract, and Scope Enforcement

> Executor reference. How this skill maps onto the coordination system's
> `$OUTPUT_DIR` discipline, which environment variables it reads, and the
> mandatory scope check. This skill is the **Site Explorer** backend — it writes
> a QA phase directory, never `findings/finding-NNN/`.

## Phase directory — write into `$OUTPUT_DIR/qa/`, never the root

The coordinator owns `$OUTPUT_DIR`'s root-level files. This skill writes only
inside its phase directory `$OUTPUT_DIR/qa/`.

```
$OUTPUT_DIR/
└── qa/                        ← this skill's phase directory
    ├── discovery/             discovery-report.json, component-inventory.json,
    │                          verified-inventory.json, assertion-target-probes.json
    ├── knowledge/             knowledge-graph.json, graph-diff.json
    ├── planning/              test-plan.json, runtime-schedule.json
    ├── pages/ fixtures/ tests/  generated Playwright suite
    ├── decision-history/      decision-000N.json  (evidence + Evidence Quality + Framework Confidence)
    ├── execution-history/     execution-*.json, deviation-record.json
    ├── reports/               results.json, results.xml, html/, diagnostics.json,
    │                          analytics.json, target-health.json,
    │                          exploration-disclosure.json, rw-report.md
    ├── aic/                   AIC serialization (04, PLAYBOOK §21):
    │                          pages, routes, apis, api-calls, forms, parameters,
    │                          auth-surfaces, robots, javascript-routes,
    │                          relationships, manifest
    │   └── evidence/          requests/, responses/ (masked), screenshots/
    └── raw/                   traces, screenshots, videos (retain-on-failure)
```

`aic/attack-surface` is **permanently `NOT_PRODUCED`**. This skill emits no
attack-surface projection, no `candidateAttackClass`, and no `suggestedSkill`
(Ownership Matrix, W8 / C1).

Discipline (per `coordination/reference/output-discipline.md`):

- Machine-readable artifacts are JSON (`discovery/`, `knowledge/`, `planning/`).
- Human-readable analysis is markdown (`reports/*.md`).
- Never mix raw tool output with analysis — raw traces/screenshots live in `raw/`.
- Preserve all raw output for reproducibility.
- A previous execution's `qa/` tree is immutable — a new run writes a new
  boundary; it never overwrites or appends to an earlier one.

### Why not `findings/finding-NNN/`

That layout is for **security skills** producing findings with a PoC and CVSS.
This skill produces **exploration evidence and QA results** — recorded in
`qa/aic/`, verified in `qa/decision-history/`, and reported in `qa/reports/`. A
functional failure is triaged as a **target defect**, a **suite defect**, or an
**environment artifact** — never as a finding, and never assigned a severity.

### The architectural boundary

```
qa-automation (Site Explorer)              selected security skill
──────────────────────────────             ───────────────────────────────────
Discovery                                  User selects the security skill
  → Application understanding                → RedOps mounts it
  → QA test planning                         → It reads Site Explorer evidence
  → QA test generation                       → It determines its own targets
  → Playwright execution                     → It determines its own methodology
  → QA results                               → Security execution
  → Evidence / reports                       → Security evidence → validation
                                             → Finding

"What exists in the application,           "Given the explicitly selected
 and what evidence do we have?"              methodology, what security tests
                                             should be performed, and how?"
```

This skill never crosses that line. It emits no security applicability tag, no
routing suggestion, no probability, no recommendation, and no finding.

## Environment contract — non-interactive

Reads context from environment variables only; no prompt ever blocks a run.

| Variable | Required | Meaning |
|---|:---:|---|
| `OUTPUT_DIR` | ✅ | Root output dir; this skill writes under `$OUTPUT_DIR/qa/` |
| `BASE_URL` | ✅* | Full URL to the app entry point |
| `TARGET` / `TARGET_DOMAIN` | ✅* | Accepted as `BASE_URL` fallback (coordinator spelling) |
| `SCOPE_FILE` | ⛒ | If present, every target validated against it before navigation |
| `ROE_FILE` | ⛒ | If present, honor time-based restrictions / blackout / expiry |
| `TEST_USER` / `TEST_PASS` | ❌ | Present → **Authenticated Exploration**; absent → **Unauthenticated Exploration**. Absence narrows scope, never stops a run |
| `ALLOW_WRITE_TESTS` | ❌ | `1` permits state-mutating QA tests: synthetic data, mandatory cleanup, scope + RoE, disclosed mutation (W7-A BD-W7-3). **Never** enables application-delete tests |
| `CI` | ❌ | Fewer workers, more retries |

**Authentication mode is not authorization.** `SCOPE_FILE` / `ROE_FILE`
validation below is mandatory in **both** exploration modes. Removed in W8:
`SECURITY_SCAN`, `ZAP_API_KEY` — this skill performs no security scanning.

\* At least one of `BASE_URL` / `TARGET` / `TARGET_DOMAIN` must resolve to a URL.

Credentials arrive via env only and are **masked before any persistence**
(`01` §30, `07` §41). No secret reaches a log, report, screenshot, trace, or
artifact.

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
- Deterministic surface identity (`sha256[0:16]` of canonical inputs); a surface
  is revisited only with a recorded reason (`PLAYBOOK` §21).
- Masking runs **before** any persistence, through one authority. No raw secret
  reaches a log, report, screenshot, trace, or artifact (`01` §30, `07` §41).
