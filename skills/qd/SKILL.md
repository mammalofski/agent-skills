---
name: qd
description: Quick Debug = Fast, evidence-first debugging for bugs, regressions, failing tests, runtime errors, and unexpected API or UI behavior. Use when you need a concise diagnose-plan-fix-verify workflow without guesswork or verbosity.
---

# Quick Debug

Debug fast, but do not guess. Diagnosis and planning come before edits.

## Workflow

1. Understand the bug
   - Capture expected behavior, actual behavior, exact reproduction, error text, and scope.
   - Restate the failure briefly before proceeding.
   - If a critical fact is missing, ask only for that fact.

2. Read the relevant code
   - Inspect the failing surface, nearby logic, data flow, and likely origin points.
   - Read enough surrounding code to understand how the area is supposed to work and where side effects could appear.
   - Match existing patterns and conventions.

3. Diagnose with evidence
   - Form the smallest plausible set of root-cause hypotheses.
   - Check the highest-signal hypotheses first and eliminate alternatives with evidence.
   - Do not implement until the root cause is confirmed with high confidence, or the remaining uncertainty is clearly bounded.

4. Plan the fix carefully
   - Prefer the smallest safe change that fixes the confirmed cause.
   - Check blast radius: callers, state, schemas, persistence, API/UI contracts, concurrency, and regressions.
   - Decide how the fix will be verified before coding.

5. Implement precisely
   - Follow the plan and keep the diff focused.
   - Avoid speculative refactors, unrelated cleanup, or stacking multiple fixes without evidence.
   - Preserve architecture and style.

6. Verify the bug is gone
   - Re-run the original repro path and confirm the failure is gone.
   - Run targeted tests first, then broader validators required by the project.
   - Add or update a regression test when practical.
   - Ask the user to verify only for checks the agent cannot perform directly, such as visual UX or external-system behavior.

7. Confirm user satisfaction
   - After verification, ask: "Is everything good, and shall I move forward with the perfect commit?"
   - If the user is not satisfied, continue the same debug workflow until the issue is fully resolved.
   - Do not commit without explicit user approval.

8. Commit cleanly if approved
   - Commit only the files changed during this session and nothing else.
   - Write a concise, high-quality commit message that reflects the verified root cause fix.
   - Report what was fixed, how it was verified, and whether the commit was created.

## Constraints

- NEVER guess or patch blindly.
- NEVER skip diagnosis or planning.
- NEVER ask the user to do work the agent can do itself.
- NEVER commit without explicit user confirmation after verification.
- NEVER include unrelated files in the commit; stage only files changed in the current session.
- ALWAYS keep explanations concise: problem, root cause, fix, verification.
- If blocked, state exactly what is missing and what single next check would unblock progress.

## Done Criteria

- Root cause is identified with evidence.
- The fix is minimal and safe.
- The original issue is verified fixed.
- Regression risk is checked with tests or the best available validation path.
- The user was asked whether everything is good and whether to proceed with the perfect commit.
- If approved, only session-scoped files were committed with a strong commit message.
