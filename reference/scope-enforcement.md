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
| `CI` | ❌ | Fewer workers, more retries |

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
