# Quick Task (qt)

Efficient end-to-end workflow for most day-to-day tasks like single feature implementations, refactors, performance improvement, UI updates, etc.: lock the task, read context, build and stress-test a plan, execute precisely, verify thoroughly, and commit only with explicit approval.

The idea is ALL implementations should do all these steps, but they don't, so just use `/qt` before any small to medium task to implement efficiently and avoid your repo from becoming an AI slop.

## When to use

- Most small to medium day-to-day coding tasks in an existing repo.
  - refactors, UI updates, performance improvements, 
  - `Add X to Y`, `improve Z`, `replace A with B and remove C`, etc.
  - medium tasks like `Add optimized multi-layer filtering to the tickets to both UI and API.`, `Analyze API response time and add in-memory caching to improve performance`, etc.
- Any task that fits a concise analyze → plan → execute → verify loop.
- As the discipline to attach when delegating work to sub-agents.

## How to use

Just replace your regular day-to-day prompts asking the agent to do something like `speed up the main page` to `/qt speed up the main page` and see how much more optimized and performant the agent will do the task instead.

Invoke in Claude Code or your agent of choice:

```
/qt Add day/night theme changing capability.
```

The skill drives the session through 8 steps (lock → read → plan → stress-test → execute → verify → confirm → close) and never commits without your explicit confirmation. Uses sub-agent orchestration implementations when necessary.

## Sibling skills

- [qd](../qd) — evidence-first debugging.
- [deep-plan](../deep-plan) + [deep-execute](../deep-execute) — large, multi-phase tasks.
