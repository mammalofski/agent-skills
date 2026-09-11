# Deep Plan (`deep-plan`)

## Purpose

`deep-plan` creates a versioned, delegation-first implementation contract for a separate `deep-execute` session. It combines product and engineering analysis with context-bounded worker packets so the main executor can remain an orchestrator rather than consuming its context on implementation.

## Artifact protocol

```text
.planning/navoid-plans/<feature-slug>/
├── plan.md               # Detailed immutable Plan-Contract 2
├── orchestrator.md       # Compact control plane for execute
├── packets.md            # Packet graph and index
├── assignments/          # Prompt-ready Sub-Execute-Assignment 1 files
└── package.sha256        # Immutable package checksums
```

Optional supporting artifacts are checksummed and indexed. All feature artifacts stay in this shared workspace; `execution.md`, `results/`, and `.execution.lock/` are reserved for execution.

## Workflow summary

1. Lock the task and repository baseline.
2. Resolve material product and engineering ambiguity.
3. Analyze current behavior, architecture, contracts, tests, and validators.
4. Select the smallest complete approach.
5. Write exact phased implementation steps.
6. Design tests afterward and use them to challenge the plan.
7. Run the change-impact gate: search every mutated symbol and structural collection for consumer and test assertions, then build the mutation-impact matrix and exact-path modification inventory.
8. Split phases into context-bounded worker packets and prompt-ready assignment files.
9. Close ownership, focused checks, and staging over every predicted modified file.
10. Write a compact orchestrator control plane with disjoint ownership, dependencies, result paths, and barriers.
11. Map requirements through steps, packets, and verification.
12. Stress-test the package as a fresh orchestrator plus fresh workers, have an independent reviewer challenge impact closure, then promote all package artifacts to `READY`.

## Delegation bias

- Every implementation phase is delegated to `sub-execute`.
- The executor reads the compact control plane, not the detailed plan or assignment bodies.
- Large phases are split before they can overflow one worker context.
- Integration files have one owner.
- Independent verification uses fresh workers.
- The main executor retains only compact orchestration state.
- There is no main-agent implementation fallback.

## Handoff

Start a fresh session with:

```text
/deep-execute .planning/navoid-plans/<feature-slug>/plan.md
```

Install `deep-plan`, `deep-execute`, and `sub-execute` together for the complete workflow.

## Key file

| File | Purpose |
| --- | --- |
| [`SKILL.md`](SKILL.md) | Full planning workflow, Plan-Contract 2, and Packets-Contract 1 formats. |

## Safety

- Planning writes only inside the selected plan directory.
- Existing plans and repository changes are user-owned.
- Parallel packets never share write ownership.
- A package is not `READY` while material questions, oversized packets, traceability gaps, or open impact-closure rows remain.
- Ownership is fixed by the immutable plan; the executor never expands it at runtime.
- Secrets and sensitive values never belong in planning artifacts.
