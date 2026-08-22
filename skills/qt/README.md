# Quick Task (qt)

Efficient end-to-end workflow for most implementation, refactor, and bug-fix tasks: lock the task, read context, build and stress-test a plan, execute precisely, verify thoroughly, and commit only with explicit approval.

## When to use

- Most small to medium coding tasks: new features, refactors, bug fixes.
- Any task that fits a concise analyze → plan → execute → verify loop.
- As the discipline to attach when delegating work to sub-agents.

## How to use

Invoke in Claude Code:

```
/qt <your task>
```

The skill drives the session through 8 steps (lock → read → plan → stress-test → execute → verify → confirm → close) and never commits without your explicit confirmation.

## Sibling skills

- [qd](../qd) — evidence-first debugging.
- [deep-plan](../deep-plan) + [deep-execute](../deep-execute) — large, multi-phase tasks.
