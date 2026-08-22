# Deep Execute (deep-execute)

Execution half of the `plan` + `execute` pair. Implements or resumes a `READY` Plan-Contract 1 artifact through verified completion: repository-drift preflight, durable execution journal, safe sub-agent orchestration, tests, independent review, and an approval-gated commit.

## When to use

- After [deep-plan](../deep-plan) produces a `READY` plan at `.planning/navoid-plans/<slug>/plan.md`.
- Any time a compatible `READY` plan needs to be implemented or resumed.

## How to use

Start a fresh session and invoke:

```
/deep-execute .planning/navoid-plans/<slug>/plan.md
```

A plan directory works too; with no path, the skill auto-selects when exactly one compatible `READY` plan exists. It executes phase by phase, keeps an `execution.md` journal, and never commits without your explicit approval.

## Sibling skills

- [deep-plan](../deep-plan) — produces the plan this skill executes.
- [qt](../qt) — everyday tasks. [qd](../qd) — debugging.
