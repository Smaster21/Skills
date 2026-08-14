# INDEX.md row — paste into the tool's skill router

When you copy this skill into `/home/kali/communitytools/skills/qa-automation/`,
add the row below to `/home/kali/communitytools/skills/INDEX.md`.

This is a **defensive QA / verification** skill, not an attack skill. It does not
belong under *Web Application* (those are exploit classes). Add it under a
**Specialized** or a new **Quality & Verification** grouping. Two options:

## Option A — under "## Specialized" (has a "Use with" column pattern nearby)

```markdown
| [`qa-automation`](qa-automation/SKILL.md) | Autonomous web-app QA — discover, plan, generate & run Playwright tests | Web app in scope needs functional / regression / E2E verification |
```

## Option B — a new grouping (if you prefer to separate defensive verification)

```markdown
## Quality & Verification

| Skill | Purpose | Trigger | Use with |
|-------|---------|---------|----------|
| [`qa-automation`](qa-automation/SKILL.md) | Autonomous web-app QA — discover, plan, generate & run Playwright tests | Web app in scope needs functional / regression / E2E verification | reconnaissance, techstack-identification |
```

## Notes for manual integration

- **Folder name must equal the `name:` field** → directory must be exactly
  `qa-automation` (it already is; `name: qa-automation` in `SKILL.md`).
- The `description` in `SKILL.md` frontmatter is ≤80 chars and matches the
  Purpose column above — keep them in sync if you edit either.
- Pairs naturally after `reconnaissance` / `techstack-identification` (they map
  the surface; this verifies it) and upstream of `regression-sweep` (which can
  re-run this suite's results on a schedule).
- Nothing else in the tool needs editing: this skill adds no engine, dataset,
  lifecycle state, phase, identifier authority, or confidence model to the
  coordination system.
