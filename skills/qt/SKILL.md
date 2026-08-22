---
name: qt
description: Quick Task = Efficient end-to-end task execution workflow for understanding requirements, planning thoroughly, executing safely, and verifying results. Use for most implementation, refactor, and bug-fix tasks that need a concise analyze-plan-execute-verify flow.
---

# Quick Task

Work efficiently, but never skip understanding or planning. Analyze first, plan second, execute only after the plan is sound.

## Workflow

1. Lock the task
   - Restate the requirement, expected output, constraints, and done condition in a few concise bullets.
   - If an essential fact is missing, ask only that question; otherwise proceed autonomously.

2. Read the right context
   - Inspect the exact code paths involved and enough surrounding code to understand intent, data flow, dependencies, and side effects.
   - Match existing architecture, conventions, and patterns instead of inventing new ones unnecessarily.
   - Check related tests, schemas, configs, and interfaces when relevant.

3. Build the plan
   - Create a concrete step-by-step plan before editing.
   - Cover all affected surfaces that matter for the task: code, contracts, persistence, config, validation, tests, and compatibility.
   - For larger work, split the task into safe phases with clear validation points.
   - Prefer the smallest complete approach that fully solves the request.

4. Stress-test the plan
   - Re-check assumptions, edge cases, dependencies, and blast radius.
   - Confirm the plan is safe, complete, and aligned with project rules before changing files.
   - If uncertainty still blocks safe execution, ask the user; otherwise commit to the plan.

5. Execute precisely
   - Follow the plan in order and keep the diff focused.
   - Avoid speculative refactors, unrelated cleanup, and changes outside scope.
   - If new evidence invalidates the plan, pause, update the plan, then continue.

6. Verify thoroughly
   - Prove the result with the strongest checks available: code review, targeted tests, validators, builds, API checks, or UI/E2E/manual flows as appropriate.
   - Start with focused verification, then run the broader required project validators before declaring success.
   - Ask the user to verify only for checks the agent cannot perform directly.

7. Confirm user satisfaction
   - After verification, ask: "Is everything good, and shall I move forward with the perfect commit?"
   - If the user is not satisfied, continue the same workflow until the task is fully done.
   - Do not commit without explicit user approval.

8. Close cleanly
   - Confirm the requested output exists and the original requirement is satisfied.
   - If the user approves, commit only the files changed during this session and nothing else.
   - Write a concise, high-quality commit message that accurately reflects the completed work.
   - Return a concise summary of what changed, how it was verified, and whether the commit was created.

## Constraints

- NEVER start editing before understanding the task and the surrounding code.
- NEVER skip planning for non-trivial work.
- NEVER ask the user to do work or provide context the agent can obtain itself.
- NEVER commit without explicit user confirmation after verified completion.
- NEVER include unrelated files in the commit; stage only files changed in the current session.
- ALWAYS optimize for correctness, safety, minimal blast radius, and clean execution.
- ALWAYS keep the workflow concise and outcome-focused.

### Sub-Agent Delegation

When delegating work to sub-agents:

- **Prompt like a staff engineer briefing a peer.** Include: goal, constraints, done condition, relevant file paths, architectural context, and the exact output you expect back.
- **Scope each sub-agent to one clear responsibility.** Prefer focused tasks (e.g., "implement X in these 3 files") over broad mandates ("finish the feature").
- **Attach this workflow.** Either include the full QT workflow in the prompt or instruct the sub-agent to use `/qt` so it follows the same analyze-plan-execute-verify discipline.
- **Provide all context the sub-agent cannot discover itself.** Decisions already made, user constraints, rejected approaches, and any plan steps that must not change.
- **Define the return contract.** State what the sub-agent must return: changed files list, verification results, summary of decisions made, and any unresolved issues.
- **Verify sub-agent output before accepting it.** Review returned changes against the plan and run validators; do not blindly merge delegated work.

## Done Criteria

- The requirement and expected output are understood.
- The relevant context and surrounding code were reviewed.
- A sound plan was created and re-checked before execution.
- The implementation followed the plan, or the plan was intentionally revised based on new evidence.
- The result was verified with the best available checks.
- The user was asked whether everything is good and whether to proceed with the perfect commit.
- If approved, only session-scoped files were committed with a strong commit message.
- The final state satisfies the user request without known collateral regressions.
