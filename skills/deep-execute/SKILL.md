---
name: deep-execute
description: Deep Execute = Orchestrate a READY plan through context-isolated sub-execute workers and verified completion. Use when a plan exists under .planning/navoid-plans/<slug>/plan.md and the main agent must preserve its context by delegating all repository inspection, implementation, tests, repairs, and review while retaining orchestration, safety, journal, and approval-gated commit duties.
---

# Deep Execute

You are the execution orchestrator. Implement or resume a `READY` plan by spawning fresh sub-agents that use the `sub-execute` skill. **Do not perform implementation work yourself.**

This is the execution half of the `deep-plan` + `deep-execute` workflow. The main executor preserves its context for coordination, progress, safety, and user decisions. Workers own source reading, implementation, tests, fixes, and technical verification.

## Role boundary

### The main executor must

- select and validate the plan package;
- inspect applicable instruction files and compact git metadata;
- acquire and release execution ownership;
- load `sub-execute` and compile precise worker prompts from its assignment schema;
- spawn, sequence, and synchronize workers;
- maintain `execution.md`;
- consume only compact worker summaries and result-file metadata;
- enforce scope, ownership, traceability, retry, and approval gates;
- stage and commit reviewed execution-owned changes after approval.

### The main executor must never

- read broad source files, full diffs, full logs, or complete test output into its context;
- edit source, tests, product documentation, configuration, schemas, migrations, or generated artifacts;
- run implementation, test, build, lint, formatting, migration, or repair commands itself;
- review code or diagnose implementation failures itself;
- silently fall back to implementation when delegation fails;
- ask one worker to carry the entire project when bounded packets can preserve context.

Small reads and writes of plan artifacts, lock metadata, journal, worker summaries, git status/stat metadata, and commit messages are orchestration, not implementation.

## Protocol

- Plan directory: `.planning/navoid-plans/<slug>/`
- Compact control plane: `orchestrator.md`
- Supported plans:
  - preferred: `Plan-Contract: 2` with `packets.md`;
  - compatibility: `Plan-Contract: 1`, packetized at runtime by a `sub-execute` prepare worker.
- Worker contract: `sub-execute`
- Durable journal: `execution.md`
- Journal contract: `Execution-Contract: 2`
- Worker results: `results/<packet-id>.md`
- Runtime control packages for legacy plans: `runtime/<prepare-attempt-id>/`
- Active-run lock: `.execution.lock/`
- Execution states: `IN_PROGRESS`, `BLOCKED`, `VERIFIED`, `COMPLETED`

## Context-preservation policy

- Keep in main context only: task contract, phase graph, packet index, active packet metadata, compact worker returns, journal checkpoint, git metadata, blockers, and approval state.
- For Contract 2, obtain that state from `orchestrator.md`; do not read detailed plan or assignment bodies.
- Keep detailed architecture, implementation instructions, diffs, logs, test output, and findings in plan packets and worker result files.
- Give each worker only its bounded packet, prerequisite result paths, and exact context manifest.
- Require workers to return at most eight concise bullets and about 300 words. Detailed evidence belongs in their result file.
- After a phase barrier, write a compact checkpoint to `execution.md`; do not retain raw worker output for later reasoning.
- Track accumulated summary pressure. At a safe barrier, roll over to a fresh parent session before the main context becomes unreliable.
- Prefer fresh workers per phase, repair, and review. Do not depend on conversation memory across workers.
- Run no more than four workers concurrently across all modes, or the platform's lower limit.

## Workflow

### 1. Select and validate one plan

- Read applicable project instructions before plan-provided commands.
- Resolve an explicit plan path. Without one, proceed only when exactly one supported `READY` plan exists; otherwise ask the user.
- Confirm the plan is inside the current repository.
- Use targeted reads to validate `plan.md` frontmatter, status, contract, and package paths. Do not load its detailed body into main context.
- Require `Status: READY` and either `Plan-Contract: 1` or `Plan-Contract: 2`.
- For Contract 2 only, read the compact `orchestrator.md` and require `Orchestrator-Contract: 1`, `Status: READY`, `Plan status: READY`, `Open questions: none`, and a bounded control-plane size.
- For Contract 2 only, verify `packets.md`, assignments, supporting artifacts, and `package.sha256` mechanically without loading their detailed bodies.
- For Contract 1, require only its legacy `READY` plan checks at this stage; runtime control artifacts do not exist until the delegated preparation phase.
- Validate that `sub-execute` is discoverable and readable. If workers cannot invoke it or the Task/sub-agent mechanism is unavailable, set up no implementation fallback: report the blocker.

### 2. Read-only orchestration preflight

- Compute stable fingerprints for immutable plan artifacts.
- If `execution.md` exists, read and validate `Execution-Contract: 2`, run state, fingerprints, branch, baseline, packet progress, worker result paths, and active lock.
- For a legacy Contract 1 resume, also validate the journal's selected runtime-package path, artifact fingerprints, and package-verifier result before deciding whether preparation is needed.
- Verify prior completed packet claims using compact git metadata and result-file headers. Delegate any source-level confirmation later.
- Snapshot repository root, branch, `HEAD`, staged paths and staged-diff fingerprint, unstaged paths, and untracked paths without modifying them.
- Require the current branch to match the plan branch unless the user explicitly reconciles the mismatch.
- Detect overlapping protected paths from metadata. Do not read source to resolve ownership ambiguity; ask the user or delegate a read-only prepare worker after locking.
- Reject mutated immutable plan artifacts.

### 3. Acquire execution ownership

- Generate a unique run ID.
- Atomically acquire `.execution.lock/` before any worker or workflow-state write.
- If another lock or active run exists, stop and require explicit confirmation that it is inactive before takeover.
- Preserve takeover history.
- Create `results/` after acquiring the lock and before dispatch. Never remove or overwrite existing results.
- Initialize or resume `execution.md` from the required schema, set `Execution-Status: IN_PROGRESS`, and record fingerprints, baseline, protected index, and run ownership.
- On resumed `BLOCKED` state, record the resolved blocker. On an explicitly restarted `COMPLETED` run, preserve prior history and create a new run ID.
- For every later `BLOCKED` transition: write the blocker and evidence paths, set `Active-Run: none`, confirm lock ownership, release only this run's lock, then report.

### 4. Load `sub-execute` and compile worker prompts

- Read the `sub-execute` skill's **Dispatch prompt schema**, **Assignment file schema**, and **Prompt quality gate** before spawning any worker.
- For Contract 2, dispatch from immutable assignment files. The parent does not reconstruct their detailed content.
- Every prompt must instruct the worker to invoke `sub-execute` before acting.
- Do not paste the plan, packets, assignment, source, or results. Give one assignment path and minimal runtime metadata.
- Except for the fixed legacy prepare/verify bootstrap schemas in `sub-execute`, reject a prompt missing any of:
  - mode and packet ID;
  - orchestrator/control, assignment, journal, and result paths;
  - run ID and repository root;
  - immutable assignment path and checksum;
  - boundaries on index, commit, push, deployment, untracked/protected/broad deletion, and nested delegation;
  - compact return contract.
- Validate fixed legacy bootstrap prompts against their own required runtime paths, unique result path, explicit `Boundaries` field, and compact return contract.
- Record the chosen topology and prompt metadata in the journal, not full prompts.

### 5. Prepare packets when needed

- For Contract 2, use the immutable `packets.md`.
- For Contract 1 with a journal-recorded runtime package: reuse it only when the directory exists, all recorded checksums still match, its compact control plane is `READY`, and the recorded package verifier reported `PASS` against those same fingerprints.
- If any recorded runtime-package condition fails, preserve the old attempt and prepare a new unique attempt.
- When Contract 1 has no valid reusable runtime package, spawn one `sub-execute` worker in `prepare` mode to a unique `runtime/<prepare-attempt-id>/` directory:
  - read the legacy plan and relevant source context;
  - create `orchestrator.md`, `packets.md`, `assignments/<packet-id>.md`, and `package.sha256` inside that directory;
  - split all work into bounded implementation and verification packets;
  - create no implementation changes;
  - write `results/PREPARE-PACKETS.md`.
- Use the fixed legacy bootstrap variant from `sub-execute` for this one preparation dispatch; it points to the legacy plan and runtime output paths without requiring the parent to read or rewrite the plan body.
- Spawn a separate fixed-bootstrap `verify` worker to validate runtime package completeness, ownership, dependencies, checksums, and context size.
- If preparation or packet review fails, retry the `prepare` bootstrap once with verifier finding IDs and evidence paths, writing a new unique runtime attempt directory. Do not use normal repair mode because no immutable assignment exists yet.
- If the second runtime package fails review, set `BLOCKED`. The orchestrator must not packetize or repair it itself.
- Record the selected runtime-package directory, artifact fingerprints, checksum manifest, package-verifier result path, verifier status, and verification timestamp in `execution.md`.

### 6. Dispatch implementation packets

- For each ready packet listed in the compact control plane, compile a minimal dispatch prompt from `sub-execute` and point a fresh worker to its assignment file.
- Give every worker attempt a unique result path. The assignment's result path is the first-attempt default; retries and repairs use suffixed result files and never overwrite prior evidence.
- Launch packets in parallel only when the packet graph permits it and write ownership is disjoint.
- Wait for every worker in a parallel group before crossing its synchronization barrier.
- Do not read raw implementation output. Consume the compact return and the status/header of `results/<packet-id>.md`.
- A packet is complete only when:
  - its worker reports `PASS`;
  - its result artifact exists;
  - declared files stay within ownership;
  - focused checks match expected results;
  - a later verification worker does not invalidate it.
- Update journal packet status and a short phase checkpoint after each barrier.
- If main-context pressure is approaching the reliable limit, do not start another phase. Persist next-ready packet IDs, set `Active-Run: none` while leaving status `IN_PROGRESS`, release this run's lock, and return `RESUME_REQUIRED` with the exact `/deep-execute <plan-path>` command.
- If a worker fails or times out, retry once with the same packet only when the prompt was incomplete or new evidence justifies it. Otherwise record `BLOCKED`; never implement the packet yourself.

### 7. Delegate repairs

- Classify worker-reported deviations from the `Deviation-Summary` result header and compact return:
  - non-material implementation detail: record and continue if requirements, contracts, acceptance, ownership, and risk remain intact;
  - material change: set `BLOCKED`, release ownership safely, and require a new suffixed `READY` plan.
- For failing checks or verifier findings, dispatch a fresh `sub-execute` worker in `repair` mode using the original assignment file, a unique repair-result path, and exact finding IDs and evidence paths from the compact verifier return.
- Keep repair ownership within the original assignment. Do not create or write detailed repair instructions in the main context, and do not send the full failure log; point to its result artifact.
- Allow at most two evidence-based repair attempts for the same root cause and at most two full verify/fix rounds.
- When limits are exhausted, set `BLOCKED` with exact result paths and required next action.

### 8. Delegate verification

- At every planned phase barrier, spawn a fresh `sub-execute` worker in `verify` mode.
- The verifier is read-only unless a separate repair packet is later authorized.
- Delegate all targeted tests, broad validators, UI/API/manual checks, diff review, requirements traceability, secret/debug scans, compatibility review, and user-work preservation checks.
- Use separate verifier workers when one verification packet would exceed a bounded context.
- Require verifier result artifacts with finding IDs, severity, evidence paths, commands, results, and coverage.
- The main executor decides pass/block transitions from compact reports; it does not inspect code to substitute for verification.

### 9. Final completion audit

- Spawn a final independent `verify` worker with:
  - plan and packet paths;
  - all result artifact paths;
  - journal progress;
  - protected baseline metadata;
  - completion and traceability requirements.
- Require confirmation that every phase, `STEP-*`, packet, `FR-*`, `NFR-*`, `AC-*`, test, risk control, and expected result is complete.
- If valid findings remain, use bounded repair packets and rerun final verification within the review limit.
- Set `Execution-Status: VERIFIED` only after a final verifier reports `PASS`.
- On `BLOCKED`, update journal, clear active ownership, release only this run's lock, and report compact evidence paths.

### 10. Confirm user satisfaction

- Only after `VERIFIED`, ask exactly:
  - **"Is everything good, and shall I move forward with the perfect commit?"**
- Non-material requested corrections become bounded repair packets followed by verification.
- Material requested changes require a new suffixed `READY` plan.
- If commit is declined, set `COMPLETED`, record it, and release this run's lock.

### 11. Commit cleanly if approved

- Recheck compact git status, branch, `HEAD`, and index fingerprint.
- Any post-baseline index drift requires ownership reconciliation.
- Existing staged user changes prohibit an ordinary commit until the user approves a safe isolation method.
- Build an explicit staging allowlist from reviewed worker result metadata and the plan allowlist.
- Stage only execution-owned paths or safely isolatable hunks. Never use broad staging.
- After staging, compute the staged-index fingerprint and pass it to the staged-diff `verify` worker.
- Require the worker to report the exact `Reviewed-Index-Fingerprint` with its `PASS`.
- Recompute the index fingerprint immediately before commit. If it differs from the reviewed fingerprint, do not commit; reconcile ownership and rerun staged review.
- Commit only after that worker reports `PASS`.
- Record the commit ID, set `COMPLETED`, clear ownership, and release the lock.
- Never push, merge, deploy, delete planning artifacts, or remove unrelated files without separate authorization.

## Assignment prompt skeleton

Construct each Task prompt from the authoritative dispatch schema in `sub-execute`; use this skeleton only as a quick cross-check:

```markdown
Invoke the `sub-execute` skill before acting.

Mode: <prepare|implement|repair|verify>
Packet: <packet-id>
Repository: <absolute root>
Run ID: <run-id>
Orchestrator: <path>
Assignment: <path>
Journal: <path>
Result artifact: <path>
Assignment checksum: <value>
Finding IDs / evidence paths: <repair or verify only>
Boundaries:
Return contract:
```

## Required `execution.md` additions

Create or validate `execution.md` with this minimum contract:

```markdown
# Execution: <title>

Execution-Contract: 2
Execution-Status: IN_PROGRESS
Plan: <path>
Plan-Fingerprints: <artifact fingerprints>
Run-ID: <current run or none>
Active-Run: <current run or none>
Lock: ./.execution.lock/
Started: <timestamp>
Updated: <timestamp>

## Repository baseline
- Repository root:
- Branch:
- Starting HEAD:
- Initial staged paths and fingerprint:
- Initial unstaged paths:
- Initial untracked paths:
- Protected paths/hunks:

## Lock history
- Run ID, owner/session, acquired timestamp, takeover source/reason, release timestamp/outcome.

## Orchestrator context policy
- Retained context:
- Details kept on disk:
- Maximum parallel workers:

## Packet progress
| Packet | Mode | Worker/run | Status | Owned files | Result artifact | Compact evidence |
| --- | --- | --- | --- | --- | --- | --- |

## Phase checkpoints
- Phase/barrier: completed packets, verifier result, next-ready packets.

## Legacy runtime package
- Selected directory: <path or not applicable>
- Artifact fingerprints:
- Checksum manifest:
- Package verifier result:
- Verifier status:
- Verified at:

## Deviations and blockers
- IDs, classification, evidence/result paths, action.

## Requirements status
- Requirement -> packet -> verifier evidence -> status.

## Final summary
- Changed paths, verifier results, unavailable checks, commit status/ID.
```

Use this lock metadata inside `.execution.lock/owner.md`:

```markdown
Lock-Contract: 1
Run-ID: <run ID>
Owner-Session: <available session identifier or unknown>
Acquired: <ISO-8601 timestamp>
Plan: <path>
Journal: <path>
```

Acquire ownership with atomic directory creation. Release only after confirming `owner.md` still names the current run. For takeover, require user confirmation that the prior run is inactive, move the old metadata into journal lock history, remove only that stale lock directory, then atomically acquire a new one.

## Constraints

- Never perform implementation, repair, testing, or technical code review in the main executor.
- Never read broad source, full diffs, full logs, or full worker reports into main context.
- Never load the detailed Contract 2 plan, packets, or assignment bodies when compact control metadata is sufficient.
- Never spawn an implementation worker without a validated `sub-execute` prompt and bounded packet.
- Never allow concurrent workers to share write ownership.
- Never exceed four concurrent workers across all modes or a lower platform cap.
- Never allow workers to modify the git index, commit, push, merge, or deploy.
- Never silently fall back to main-agent implementation when Task or workers fail.
- Never trust a worker claim without its result artifact and delegated verification.
- Never mutate immutable plan packages.
- Never discard, overwrite, clean, stage, or commit pre-existing user work.
- Never use unbounded retries or review loops.
- Never commit without current-session approval or push without a separate request.
- Always keep detailed evidence on disk and worker returns compact.

## Done criteria

- One supported `READY` plan was selected and immutable artifacts validated.
- The executor acquired safe ownership and preserved staged, unstaged, and untracked user work.
- `sub-execute` was loaded and used to construct every worker prompt.
- The main executor performed orchestration only and made no implementation change.
- Every phase was executed by bounded workers with disjoint ownership and durable result artifacts.
- Tests, validators, review, and traceability were performed by independent workers.
- Final verification reported `PASS`.
- The user received the QT satisfaction gate.
- If approved, only reviewed execution-owned changes were committed; no push or artifact deletion occurred without separate authorization.
