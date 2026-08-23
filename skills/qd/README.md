# Quick Debug (qd)

Fast, evidence-first debugging: capture the bug, diagnose the root cause with evidence, apply the smallest safe fix, and verify the original issue is gone.

Similar to [qt skill](../qt) but for debugging.

## When to use

- Bugs, regressions, failing tests, and runtime errors.
- Unexpected API or UI behavior.
- Any "why does X misbehave?" question where guesswork must be avoided.

Just replace most your regular day to day debugs and fixes with from `fix this bug` to `/qd fix this bug` and see how much more efficiently and accurately it will resolve your issues.

## How to use

Invoke in Claude Code:

```
/qd fix the alembic migrations failing.
```

Include expected vs actual behavior, error text, and reproduction steps. The skill drives a diagnose → plan → fix → verify loop and never commits without your explicit confirmation.

## Sibling skills

- [qt](../qt) — general implementation tasks.
- [deep-plan](../deep-plan) + [deep-execute](../deep-execute) — large, multi-phase tasks.
