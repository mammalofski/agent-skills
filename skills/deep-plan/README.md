# Deep Plan (deep-plan)

Planning half of the `plan` + `execute` pair. Produces a versioned, implementation-ready plan (`Plan-Contract: 1`) as a durable handoff at `.planning/navoid-plans/<slug>/plan.md` for a separate execution session.

## When to use

- Large or complex tasks that need thorough product and engineering Q&A.
- Work that deserves exact phased changes, test design, acceptance traceability, and a fresh execution session.
- Whenever the task is too big for [qt](../qt) and needs staff-level planning.

## How to use

0. (optional but recommended) Have a feature/specs document for your complex task ready (FEAT-1.md).
1. Start a session with your most powerful model.
2. Invoke `/deep-plan plan for implementing FEAT-1.md`.
3. Answer the plan-shaping questions; the skill builds the plan and writes a `READY` plan to `.planning/navoid-plans/<slug>/plan.md`.
4. Start a **fresh session** and run `/deep-execute .planning/navoid/<slug>/plan.md` to implement it.

Deep Plan never edits source code — it writes planning artifacts only.

## Sibling skills

- [deep-execute](../deep-execute) — implements the READY plan.
- [qt](../qt) — everyday tasks. [qd](../qd) — debugging.
