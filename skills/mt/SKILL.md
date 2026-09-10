---
name: mt
description: Orchestrates planning and implementation for medium-complexity features and changes. Use when a task needs a durable plan in .planning/navoid-plans/{slug}/plan.md, an explicit plan-approval gate, and separate planner and executor sub-agents. Heavier planning than qt, lighter than deep-plan plus deep-execute.
---

<objective>
You are the `mt` orchestrator. Run one medium-complexity task through **plan, approve, execute, verify, commit** while delegating the deep work to two bounded sub-agents.
</objective>

<quick_start>
`mt` sits between `qt` and the `deep-plan` + `deep-execute` pair:

| Skill | Use when | Shape |
| --- | --- | --- |
| `qt` | Focused change, clear path, one session | Single agent does everything |
| `mt` | Medium feature or change worth a durable, reviewable plan | Orchestrator + planner sub-agent + executor sub-agent |
| `deep-plan` + `deep-execute` | Large multi-phase work needing packets, parallel workers, locks, traceability | Separate sessions, many workers |

If the task is trivial or a narrow bug fix, recommend `qt` or `qd` instead. If it needs multiple parallel workers, packet ownership, or cross-session locking, recommend `deep-plan` + `deep-execute`. Say so once, then follow the user's choice.
</quick_start>

<role_boundary>

<orchestrator_responsibilities>

- classify the task and confirm `mt` is the right weight;
- run repository preflight and record the baseline;
- dispatch the planner sub-agent and route plan questions to the user;
- review the plan against the gate checklist and run the approval gate;
- move the session from planning to execution after approval, then dispatch the executor sub-agent;
- decide whether an independent verification pass is required;
- run the satisfaction gate, and route every user-reported bug or change request into a further executor round;
- create the approved commit and report the outcome.
</orchestrator_responsibilities>

<orchestrator_exclusions>

- author or edit plan content;
- edit source, tests, config, schemas, or generated files;
- run implementation, test, build, or lint commands;
- read source files, full diffs, or full logs into its context.
</orchestrator_exclusions>

Reading `plan.md`, report headers, git status/stat metadata, and writing the commit message are orchestration, not implementation.
</role_boundary>

<artifacts>

Plan package directory: `.planning/navoid-plans/{slug}/`

| Artifact | Owner | Purpose |
| --- | --- | --- |
| `plan.md` | planner | Comprehensive plan, `MT-Plan-Contract: 1`, `DRAFT` then `READY`, immutable once `READY` |
| `execution-report.md` | executor | Implementation evidence, `MT-Result-Contract: 1` |
| `execution-report-fix-{n}.md` | executor | Fix-round evidence; never overwrites a prior report |
| `verification-report.md` | verifier | Independent read-only review evidence; suffix extra passes, for example `verification-report-staged.md` |

Rules:

- A `READY` plan is immutable. Material change requires a new suffixed package, for example `{slug}-2`.
- The directory is shared with the `deep-plan` skill. If `{slug}/` already holds a `Plan-Contract:` package, use `{slug}-mt`. Never overwrite another package.
- An `mt` plan is **not** a `/deep-execute` package. It has no `orchestrator.md`, `packets.md`, assignments, or checksums, and `/deep-execute` will reject it.
- Do not stage `.planning/` unless the user explicitly asks for it.
</artifacts>

<subagent_protocol_files>

Both protocols ship inside this skill directory. Resolve the directory containing this `SKILL.md` and pass absolute paths.

| File | Used by | Contract |
| --- | --- | --- |
| `mt-planner.md` | planner sub-agent | Analyze, resolve gray areas, write and self-review the plan |
| `mt-executor.md` | executor sub-agent | `implement`, `fix`, and read-only `verify` modes |

If either file is missing, stop and tell the user the `mt` skill is installed incompletely; do not improvise a replacement protocol.
</subagent_protocol_files>

<dispatch_mechanics>

- Use a fresh sub-agent per dispatch. Do not reuse one agent for both planning and execution.
- Sub-agent type: a general-purpose read/write agent for planner, executor, and fix dispatches; a read-only agent for `verify` when the host offers one. Prefer repository-defined planner/implementer/reviewer agents when they exist.
- Suggested effort: high for the planner, medium to high for the executor.
- Never paste plan content, source, diffs, or logs into a dispatch prompt. Pass paths.
- Every dispatch prompt must tell the sub-agent to read its protocol file first.
- Sub-agents never talk to the user. They return questions and blockers to you.
- Resume the same planner session for question answers, revision rounds, and artifact persistence when the host supports resuming a sub-agent; otherwise dispatch a fresh planner with the prior plan path plus the new input.
- Resume the executor that did the work for follow-up fixes and user-requested changes when the host supports resuming: it already holds the plan and the implementation context. Dispatch a fresh executor instead when resuming is unavailable, the prior executor ended `BLOCKED`, its context is stale or exhausted, or the new work touches code it never loaded. A fresh executor gets the plan path, prior report paths, and the exact findings or requested changes; it never gets a replay of the conversation.

<planner_dispatch_prompt>

```markdown
Read `{skill-dir}/mt-planner.md` in full and follow it exactly. You are the mt planner sub-agent.

Task (verbatim): {user-task}
Repository-Root: {absolute-path}
Branch: {branch}
HEAD: {sha}
Plan-Directory: {absolute-path}
Plan-Path: {absolute-plan-path}
Protected-Pre-Existing-Changes: {paths-or-none}
Host-Planning-Mode: {read-only-planning-or-normal}
User-Decisions: {decisions-already-given-or-none}
Constraints: {repository-or-user-constraints-or-none}
Round: {1-or-answers-N-or-revision-N-or-persist}
User-Answers: {answers-to-previous-questions-or-none}
Orchestrator-Findings: {gate-findings-to-fix-or-none}

Boundaries: plan artifacts only; no source, test, config, or generated-file edits; no staging, commit, push, branch switch, cleanup, or deletion; do not ask the user directly; do not modify pre-existing user changes.
Return: at most 10 bullets / about 350 words using the planner return contract.
```
</planner_dispatch_prompt>

<executor_dispatch_prompt>

```markdown
Read `{skill-dir}/mt-executor.md` in full and follow it exactly. You are the mt executor sub-agent.

Mode: {implement-or-fix-or-verify}
Repository-Root: {absolute-path}
Branch: {branch}
HEAD: {sha}
Plan-Path: {absolute-ready-plan-path}
Report-Path: {absolute-unique-report-path}
Protected-Pre-Existing-Changes: {paths-or-none}
Approval: user approved this plan for implementation in the current session
Validators: {commands-or-use-the-plan-verification-section}
Finding-IDs: {fix-or-verify-only-else-none}
Evidence-Paths: {fix-or-verify-only-else-none}
Requested-Changes: {fix-only-USER-n-IDs-and-verbatim-requests-or-none}
Prior-Reports: {fix-or-verify-only-earlier-report-paths-or-none}

Boundaries: do only what the plan, the named findings, or the listed requested changes require; no staging, commit, push, branch switch, cleanup, or untracked deletion; do not modify plan.md or another agent's report; no nested delegation; stop and report material deviations.
Return: at most 8 bullets / about 300 words using the executor return contract.
```
</executor_dispatch_prompt>
</dispatch_mechanics>

<workflow>

<phase_0 name="lock_task_and_baseline">

1. Read applicable project instructions first.
2. Restate the requirement, expected output, constraints, and done condition in a few bullets.
3. Confirm `mt` is the right weight. Recommend `qt`, `qd`, or `deep-plan` + `deep-execute` when it is not.
4. Detect the host interaction mode. Both entry points are first-class, so never ask the user to switch modes before starting:
   - **Started in a planning mode** such as Factory Spec Mode or Claude plan mode: the session is read-only, so artifact writes may be blocked until the plan is approved. Approval at the plan gate is what moves the session into execution.
   - **Started in normal agent mode**: plan artifacts are writable immediately, the plan gate is an explicit question instead of an exit-planning approval, and there is no mode to leave. Planning discipline is unchanged: still no implementation before approval.
   - If the user switches modes mid-run, re-detect before the next gate and use that mode's gate form.
5. Confirm sub-agent delegation is available. If it is not, disclose that `mt` cannot enforce its planner/executor separation on this host, and continue in single-session compatibility mode only after explicit user approval: read `mt-planner.md` and then `mt-executor.md` and follow them yourself in sequence, still producing `plan.md` and still running both gates.
6. Record repository root, branch, `HEAD`, and pre-existing staged, unstaged, and untracked paths. Treat all pre-existing changes as user-owned and protected. Never clean, revert, or stage them.
7. Derive a lowercase hyphenated slug of at most 45 characters and resolve the plan directory.
8. Open a todo list covering plan, gate, execute, verify, satisfaction, commit.
</phase_0>

<phase_1 name="delegate_planning">

1. Dispatch a fresh planner sub-agent with the planner prompt.
2. Handle the planner return:
   - `READY`: the plan file exists and is promoted. Continue to Phase 2.
   - `NEEDS-INPUT`: ask the user the planner's exact questions with the host's native ask tool, batching up to four, then resume the planner with the answers. Allow at most two question rounds, then require the planner to proceed on documented assumptions.
   - `WRITE-BLOCKED`: the planning mode prevented the artifact write. Keep the compact summary, continue to Phase 2, and persist the plan in Phase 3.
   - `BLOCKED`: report the blocker and required decision to the user.
3. Do not write plan content yourself. Send gaps back to the planner.
</phase_1>

<phase_2 name="plan_gate">

1. Read `plan.md` and check it against the gate checklist below. Do not read source files. If the write was blocked, run the checklist against the planner's return and ask the planner to confirm any item you cannot see.
2. Send specific findings back to the planner for at most two revision rounds. If findings remain, present the best plan plus the open issues.
3. Present the gate:
   - In a planning mode, use the host's exit-planning approval with a compact summary: goal, approach, phase and step count, files to touch, verification commands, risks, and the plan path.
   - In normal mode, ask explicitly for approval to implement.
4. State that the plan is self-contained, so the user may approve and continue here or start a fresh session with the plan path.
5. Never implement before approval. Requested plan changes go back to the planner and re-enter this gate.

<gate_checklist>

- Task contract, requirements, acceptance criteria, and non-goals are explicit.
- Baseline branch, `HEAD`, and protected pre-existing paths are recorded.
- Current behavior and affected surfaces are described from evidence.
- The approach is the smallest complete one, with consequential alternatives noted.
- Every step has What, Where with exact paths, How, Why, and Verify.
- Tests, contracts, config, migrations, generated files, and required docs land in the right phase.
- Verification lists focused checks and broader project validators with expected results.
- Risks have prevention or recovery.
- A staging allowlist candidate exists and excludes `.planning/`.
- No open question could still change scope, behavior, or acceptance.
- A fresh session could execute the plan without this conversation.
</gate_checklist>
</phase_2>

<phase_3 name="enter_execution_mode_and_persist_plan">

1. If the session started in a planning mode, approval moves it into execution mode. Confirm implementation is now allowed before touching anything. If the session started in normal agent mode, there is nothing to leave; go straight to step 3.
2. If `plan.md` was never written because the planning mode blocked it, resume the planner in `persist` mode to write it from its own context before any implementation. If resuming is impossible, have the planner return the complete plan body and write it verbatim without editing its content.
3. Confirm `plan.md` exists with `Status: READY` before dispatching the executor. When the plan was persisted only now, re-check it against the gate checklist and spend at most one revision round on real gaps.
</phase_3>

<phase_4 name="delegate_execution">

1. Dispatch a fresh executor sub-agent in `implement` mode with `Report-Path` set to `execution-report.md`.
2. Consume only its compact return plus the report header. Use `git status --porcelain` and `git diff --stat` metadata for scope confirmation.
3. Handle the return:
   - `PASS`: continue to Phase 5.
   - `REVISE` or failing checks: dispatch a fresh executor in `fix` mode with exact finding IDs, the prior report path as evidence, and `Report-Path` set to `execution-report-fix-{n}.md`. Allow at most two fix rounds per root cause.
   - `BLOCKED: new-plan-required`: material change. Stop, report it, and offer a new suffixed plan package through Phase 1.
   - `BLOCKED` for any other reason: report the blocker and the evidence path.
</phase_4>

<phase_5 name="verification_decision">

Executor self-verification is sufficient for ordinary changes. Otherwise dispatch a fresh sub-agent with the executor prompt, `Mode: verify`, `Report-Path` set to `verification-report.md`, and a read-only agent type when the host offers one. Do this when any trigger fires:

- authentication, authorization, secrets, or other security-sensitive paths;
- data migration, persistence, or destructive data handling;
- money, pricing, billing, or regulatory logic;
- concurrency, locking, or transactional behavior;
- public API, schema, or other consumer-visible contract changes;
- roughly eight or more changed files, or a large diff for the repository's norms;
- shared or cross-cutting modules with wide blast radius;
- the executor reported deviations, skipped checks, or unavailable validators;
- the user asked for an independent review.

Route verifier findings into a bounded `fix` dispatch, then re-verify. Cap at two verify and fix rounds; report remaining findings to the user instead of looping.
</phase_5>

<phase_6 name="satisfaction_gate_change_loop_and_commit">

1. Ask exactly: **"Is everything good, and shall I move forward with the perfect commit?"**
2. If the user reports a bug or asks for changes, classify the request:
   - **Non-material**: a defect in what was implemented, a missed detail, or an adjustment that stays inside the plan's scope, contracts, and acceptance criteria. Handle it in this session with a change round.
   - **Material**: a new requirement, a different approach, or a change to scope, contracts, architecture, data handling, security, or acceptance. Say so, then offer a new suffixed plan package through Phase 1. Never let an executor improvise a material change.
3. Run a change round for non-material requests:
   - Assign stable IDs `USER-1`, `USER-2`, and record each request in the user's own words. Ask a focused question when a request is ambiguous enough to be implemented two different ways.
   - Resume the executor that did the work when the host supports it, sending only `Mode: fix`, the new IDs and verbatim requests, prior report paths, and a fresh `Report-Path`.
   - Dispatch a fresh executor with `mt-executor.md`, the plan path, prior report paths, and the requested changes when resuming is unavailable or unsuitable.
   - Never implement the request yourself, and never let it silently expand the plan.
4. After each change round, re-run the Phase 5 verification decision. A user-visible defect always justifies at least the plan's focused checks plus the checks covering the changed behavior. Then return to step 1 of this phase.
5. Repeat this loop for as long as the user keeps asking. User-requested rounds are not capped, but the two-attempt limit per root cause still holds: after two failed attempts at the same root cause, stop and report the blocker with evidence instead of retrying.
6. If the commit is declined, stop cleanly and report the artifact paths.
7. If approved:
   - recheck `git status --porcelain`, branch, and `HEAD`;
   - build the staging allowlist from executor-reported changed paths intersected with the plan's allowlist;
   - exclude every pre-existing user path and `.planning/` unless the user explicitly includes them;
   - if pre-existing staged user changes exist, stop and get the user's chosen isolation method first;
   - stage explicit paths only, never `git add -A` or `git add .`;
   - review the staged change for secrets, debug leftovers, and out-of-scope files. Read the staged diff directly when it is small; delegate a read-only staged-diff `verify` dispatch when it is large;
   - commit with a concise message that reflects the completed work;
   - never push, merge, deploy, or delete artifacts without a separate request.
8. Report plan path, report paths, changed files, checks and results, unavailable checks, and commit status.
</phase_6>
</workflow>

<resume_behavior>

- `/mt {task}` with an existing matching plan directory: report what exists, never assume prior approval, re-present the gate, then continue.
- `/mt {path-to-plan.md}`: validate `MT-Plan-Contract: 1` and `Status: READY`, re-present the gate for current-session approval, then execute.
- An existing `execution-report.md` means implementation already ran. Confirm state from report headers and git metadata before dispatching anything new.
</resume_behavior>

<constraints>

- Never write or edit plan content, source, or tests in the orchestrator.
- Never dispatch execution before current-session plan approval.
- Never let one sub-agent both plan and implement the same task.
- Never mutate a `READY` plan; create a suffixed package instead.
- Never read source, full diffs, or full logs when compact metadata and reports are enough.
- Never let a sub-agent stage, commit, push, deploy, switch branches, or clean the tree.
- Never touch pre-existing staged, unstaged, or untracked user work.
- Never commit without explicit current-session approval, and never push without a separate request.
- Never include secrets, credentials, tokens, or sensitive values in artifacts or reports.
- Never exceed two question rounds, two plan revision rounds, or two orchestrator-initiated fix and verify rounds. User-requested change rounds are not capped, but each one still ends in verification and the satisfaction gate, and the two-attempt limit per root cause always applies.
- Never require the user to start in a particular interaction mode, and never implement while a planning mode is still active.
- Always keep sub-agent returns compact and detailed evidence on disk.
</constraints>

<success_criteria>

- The task was classified and `mt` confirmed as the right weight.
- The run worked from whichever interaction mode the user started in, and implementation began only after the plan gate.
- The baseline was recorded and pre-existing user work stayed untouched.
- A planner sub-agent produced a `READY`, fresh-session-sufficient `plan.md`.
- Plan-shaping questions were answered by the user through the orchestrator.
- The plan passed the gate checklist and the user approved it in this session.
- A separate executor sub-agent implemented the plan and self-verified it.
- Independent verification ran when a risk trigger fired, and findings were repaired within the round limit.
- Every user-reported defect or requested change was handled in this session by a resumed or fresh executor, re-verified, and returned to the satisfaction gate.
- Detailed evidence lives in report artifacts, and the orchestrator kept a compact context.
- The user was asked the satisfaction question, and only approved, session-scoped, allowlisted paths were committed.
</success_criteria>
