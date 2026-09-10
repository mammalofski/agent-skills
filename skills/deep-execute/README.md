# Deep Execute (`deep-execute`)

## Purpose

`deep-execute` is a context-preserving orchestrator. It implements or resumes a `READY` plan by spawning fresh workers that use `sub-execute`; the main agent performs no source implementation, testing, repair, or technical code review.

## Artifact protocol

```text
.planning/navoid-plans/<slug>/
├── plan.md             # Immutable READY plan
├── orchestrator.md     # Compact executor control plane
├── packets.md          # Packet graph and index
├── assignments/        # Prompt-ready worker assignments
├── package.sha256      # Package integrity
├── runtime/            # Versioned legacy preparation attempts
│   └── <attempt>/      # Orchestrator, packets, assignments, checksums
├── execution.md        # Durable orchestration state
├── results/            # Detailed per-worker evidence
└── .execution.lock/    # Active-run ownership
```

## Orchestrator model

- Main executor: compact control-plane validation, safety, minimal dispatch prompts, Task calls, journal, barriers, approval, staging, and commit.
- `sub-execute` workers: source reading, implementation, tests, repair, and verification.
- Detailed plan, assignment, source, diff, log, and test context remains outside the main window.
- Worker returns are limited to eight bullets / about 300 words.
- Fresh workers isolate each phase and review.
- No more than four workers run concurrently across all modes.
- Safe phase barriers support journal-backed rollover to a fresh parent session before context pressure becomes unsafe.

## Workflow summary

1. Select one `READY` package and load its compact `orchestrator.md`.
2. Perform read-only branch, drift, index, and journal preflight.
3. Acquire active-run ownership.
4. Load `sub-execute` and dispatch each immutable assignment with a minimal schema-driven prompt.
5. Delegate runtime packet preparation for legacy plans.
6. Dispatch bounded implementation packets in safe sequential or parallel groups.
7. Delegate repairs and independent phase verification.
8. Delegate final completion and requirements audit.
9. Ask the QT satisfaction question.
10. After approval, stage exact execution-owned changes, delegate staged-diff review, and commit.

## Invocation

```text
/deep-execute .planning/navoid-plans/<slug>/plan.md
```

`sub-execute` must be installed and available to spawned workers.

## Key file

| File | Purpose |
| --- | --- |
| [`SKILL.md`](SKILL.md) | Full orchestrator boundary, prompt compilation, dispatch, context, verification, and commit workflow. |

## Safety

- The main executor never implements or silently falls back when workers fail.
- Every worker gets explicit ownership and a unique result path.
- Parallel workers never share write ownership.
- User work, branch, index, immutable plans, and active runs remain protected.
- Workers cannot stage, commit, push, merge, deploy, switch branches, or clean.
- Commit and push remain separately authorized.
