---
name: mt
description: Orchestrates medium-complexity changes with a durable plan in .planning/navoid-plans, explicit plan approval, and separate planner and executor sub-agents. Use for work that needs more structure than qt but not the multi-worker deep-plan and deep-execute workflow.
---

<objective>
Run the requested medium task through one bounded sequence: plan, review, approve, execute, verify, satisfy the user, and commit. The orchestrator coordinates; a planner owns the plan and a separate executor owns implementation.
</objective>

<quick_start>
Follow the eight workflow steps in order whenever `mt` is invoked.
</quick_start>

<essential_constraints>

- Start this workflow when `mt` is invoked; do not reclassify the task or redirect to another skill.
- Keep all artifacts in `.planning/navoid-plans/<slug>/`: `plan.md`, `execution-report.md`, unique `execution-report-fix-<n>.md` files, and `verification-report.md`.
- The plan lifecycle is `DRAFT` → `REVIEW` → `READY`. Only `READY` is immutable and executable. The planner owns every plan edit and promotion.
- The orchestrator never edits plans, source, tests, configuration, or generated files, and never runs implementation validators. Its sole write exception is copying a write-blocked planner's complete, approved plan verbatim when that planner cannot resume. It may read artifact headers, plan content, reports, `git status --porcelain`, and `git diff --stat`.
- Use distinct planner and executor identities. Resume an agent only within its original role for answers, plan revisions, artifact persistence, or scoped fixes; never let the planner implement.
- When resumption is unavailable, stale, blocked, or unsuitable for the touched code, dispatch a fresh same-role agent with the plan and relevant report or evidence paths.
- Protect all pre-existing staged, unstaged, and untracked user changes. Neither sub-agent may stage, commit, push, switch branches, clean, reset, stash, or delete untracked files.
- Never dispatch implementation before current-session approval. Never commit without the user's explicit current-session approval, and never push, merge, deploy, or delete artifacts without a separate request.
- Keep detailed evidence in artifacts and sub-agent returns compact.
</essential_constraints>

<protocols>

Resolve the directory containing this file and pass absolute paths to these required protocols:

| Role | File | Use |
| --- | --- | --- |
| Planner | `mt-planner.md` | Research, resolve material questions, write and revise the plan. |
| Executor | `mt-executor.md` | Implement, fix, or independently verify an approved plan. |

If a protocol is missing, stop and report an incomplete `mt` installation; do not improvise a replacement.
</protocols>

<dispatch_contracts>

Never paste plan bodies, source, diffs, or logs into a dispatch; pass paths instead. Every sub-agent must read its protocol before acting and must not talk directly to the user.

<planner_dispatch>

```markdown
Read `{skill-dir}/mt-planner.md` in full and follow it exactly. You are the mt planner.

Task: {verbatim-user-task}
Repository-Root: {absolute-path}
Branch: {branch}
HEAD: {sha}
Plan-Directory: {absolute-path}
Plan-Path: {absolute-path}
Feature-Input-Artifacts: {paths-or-none}
Protected-Pre-Existing-Changes: {paths-or-none}
Host-Planning-Mode: {normal-or-write-blocked}
User-Decisions: {decisions-or-none}
Constraints: {constraints-or-none}
Round: {plan|answers-N|revision-N|promote|persist}
User-Answers: {answers-or-none}
Orchestrator-Findings: {findings-or-none}
Persistence-Fallback: {resume-available|return-complete-plan-body-if-write-blocked}

Boundaries: plan artifacts only; no implementation, git mutation, cleanup, deletion, or direct user questions.
Return: the planner return contract, at most 10 bullets / about 350 words.
```
</planner_dispatch>

<executor_dispatch>

```markdown
Read `{skill-dir}/mt-executor.md` in full and follow it exactly. You are the mt executor.

Mode: {implement|fix|verify}
Repository-Root: {absolute-path}
Branch: {branch}
HEAD: {sha}
Plan-Path: {absolute-ready-plan-path}
Report-Path: {absolute-unique-report-path}
Protected-Pre-Existing-Changes: {paths-or-none}
Approval: user approved this plan for implementation in the current session
Validators: {commands-or-use-plan-verification-section}
Finding-IDs: {IDs-or-none}
Evidence-Paths: {paths-or-none}
Requested-Changes: {USER-IDs-and-verbatim-requests-or-none}
Prior-Reports: {paths-or-none}

Boundaries: do only planned or named work; no plan/report overwrite, nested delegation, git mutation, cleanup, or deletion. Report material deviations.
Return: the executor return contract, at most 8 bullets / about 300 words.
```
</executor_dispatch>
</dispatch_contracts>

<workflow>

<step number="1" name="preflight">

1. Read applicable project instructions.
2. Record repository root, branch, `HEAD`, and pre-existing staged, unstaged, and untracked paths. Treat them as protected user work.
3. Reuse the slug from relevant `initial-specs.md` or `final-specs.md`; otherwise derive a lowercase hyphenated slug of at most 45 characters. Set the plan directory to `.planning/navoid-plans/<slug>/`. If it contains a non-`mt` plan package, use `<slug>-mt` instead.
4. Detect whether this host can write planning artifacts. Preserve write-blocked planning support; do not require a mode change before planning.
5. Confirm delegation is available. If it is not, disclose that role separation cannot be enforced and continue in compatibility mode only with the user's explicit approval, following the planner protocol and then the executor protocol yourself.
</step>

<step number="2" name="plan">

Dispatch a fresh planner in `plan` round. Handle its return in sequence:

- `NEEDS-INPUT`: ask up to four exact questions, resume the planner with answers, and allow at most two question rounds before documented assumptions are required.
- `REVIEW`: continue to Step 3.
- `WRITE-BLOCKED`: continue to Step 3 using the planner's compact plan summary and checklist attestation; preserve the planner session for Step 4.
- `BLOCKED`: report the blocker and stop.
</step>

<step number="3" name="review_and_approve">

1. If `plan.md` exists, require `MT-Plan-Contract: 1` and `Status: REVIEW`, then review it against the checklist below. Do not inspect source files.
2. Send specific gaps to the planner in `revision-N` round. Allow at most two revision rounds. A planner revision remains `REVIEW`; it never mutates `READY`. If material gate gaps remain, report them and stop for a user decision.
3. Present the user with the goal, approach, phase and step count, touched paths, verification commands, risks, and plan path. Ask explicitly for approval to implement.
4. If the user requests plan changes, return them to the planner and repeat this step. Do not implement before approval.

<plan_gate_checklist>

- Requirements, acceptance criteria, non-goals, baseline, and protected paths are explicit.
- Current behavior, affected surfaces, and consequential decisions are based on evidence.
- Each step has What, Where, How, Why, Verify, and Expected; tests, contracts, configuration, migrations, generated files, and docs are accounted for.
- Focused checks, broader validators, risks, recovery, staging allowlist, and fresh-session execution detail are present.
- No unresolved question can change scope, behavior, or acceptance.
</plan_gate_checklist>
</step>

<step number="4" name="persist_and_lock">

1. After approval, resume the planner in `promote` round to change the reviewed on-disk plan to `READY` without changing its content.
2. If planning was write-blocked, resume the planner in `persist` round to write its retained, approved plan directly as `READY`. If resumption is unavailable, request the complete plan body from that planner and write it verbatim without changing it.
3. Require a `READY` `plan.md` at the selected path before execution. Recheck only its artifact contract and baseline; a new finding now requires a new suffixed package, not an edit to the locked plan.
</step>

<step number="5" name="execute">

1. Dispatch a fresh executor in `implement` mode with `execution-report.md`.
2. Consume its compact return and report header, then use `git status --porcelain` and `git diff --stat` only to confirm scope.
3. On `PASS`, continue. On `REVISE` or a failing check, dispatch or resume that executor in `fix` mode with exact finding IDs, evidence paths, a new fix report path, and no more than two attempts per root cause. On `BLOCKED: new-plan-required`, report the material change and start a new suffixed package. On another `BLOCKED`, report the evidence and stop.
</step>

<step number="6" name="independent_verification">

Dispatch a fresh verifier in `verify` mode—read-only when the host supports it—when the change affects security, persistence or destructive data handling, money or regulation, concurrency, public contracts, broad or cross-cutting code, or when the executor reports deviations, skipped checks, unavailable validators, or the user requests review. Route findings into bounded executor fix rounds and re-verify; stop after two verification/fix rounds and report unresolved findings.
</step>

<step number="7" name="satisfaction_and_changes">

Ask exactly: **"Is everything good, and shall I move forward with the perfect commit?"**

- For a non-material defect or adjustment inside the plan's scope, assign `USER-<n>` IDs, dispatch or resume the executor in `fix` mode with a fresh report path, have it run the relevant checks, repeat Step 6, and ask again. Repeat for each user-requested change round.
- For a material request that changes scope, contracts, architecture, data handling, security, or acceptance, report that a new suffixed plan package is required. Do not let the executor improvise it.
- End cleanly if the commit is declined.
</step>

<step number="8" name="commit">

After explicit approval, recheck branch, `HEAD`, and git status. Stage only explicit, session-scoped paths that are both executor-reported and in the plan allowlist; exclude `.planning/` and protected user paths unless the user expressly includes them. If protected staged work exists, ask for an isolation decision. Review the staged change for secrets, debug leftovers, and scope; use a staged-diff verifier when the diff is too large to inspect directly. Commit with a concise message, and report artifact paths, changed files, checks, unavailable checks, and commit status.
</step>
</workflow>

<resume_behavior>

- For an existing matching package, inspect artifact headers and git metadata, never assume prior approval, then re-present the current plan gate.
- For a direct `plan.md` path, require `MT-Plan-Contract: 1` and `Status: READY`, obtain current-session approval, then continue at Step 5.
- For an existing execution report, confirm its state from headers and git metadata before choosing a fix, verification, satisfaction, or commit step.
</resume_behavior>

<success_criteria>

- A separate planner produced a self-contained reviewed plan, and the user approved its implementation in this session.
- The planner promoted the approved artifact to an immutable `READY` plan before a separate executor implemented it.
- User work remained protected; evidence reports document implementation, fixes, and any independent verification.
- Every non-material user change was verified and returned to the satisfaction gate; material changes were replanned.
- Only explicitly approved, allowlisted session work was committed.
</success_criteria>
