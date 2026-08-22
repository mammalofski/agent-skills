# Quick Debug (qd)

Fast, evidence-first debugging: capture the bug, diagnose the root cause with evidence, apply the smallest safe fix, and verify the original issue is gone.

## When to use

- Bugs, regressions, failing tests, and runtime errors.
- Unexpected API or UI behavior.
- Any "why does X misbehave?" question where guesswork must be avoided.

## How to use

Invoke in Claude Code:

```
/qd <bug description>
```

Include expected vs actual behavior, error text, and reproduction steps. The skill drives a diagnose → plan → fix → verify loop and never commits without your explicit confirmation.

## Sibling skills

- [qt](../qt) — general implementation tasks.
- [deep-plan](../deep-plan) + [deep-execute](../deep-execute) — large, multi-phase tasks.
