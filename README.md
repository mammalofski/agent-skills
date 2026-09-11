# Everyday Agent Skills

Practical software-development workflows that prioritize scoped work, evidence-based verification, protection of existing changes, and approval-gated commits.

## Skills

### [qt - Quick Task](./skills/qt)

Use this for most focused small-to-medium changes with a clear path.

Workflow summary:

- Lock the task; ask clarifying questions if needed.
- Read the right context.
- Build a laser-cut plan. Stress-test it.
- Execute precisely.
- Verify thoroughly.
- Confirm user satisfaction, and close cleanly.

### [qd - Quick Debug](./skills/qd)

Use this for bugs, regressions, failed tests, and unexpected behavior. It diagnoses with evidence, plans the smallest safe fix, implements it, and verifies the original failure is gone.

### [mt - Medium Task](./skills/mt)

Use this for a medium-complexity feature or change that needs a durable, reviewable plan without the full packetized deep-workflow overhead. An orchestrator delegates planning and implementation to separate sub-agents, with plan approval, risk-based independent verification, and an approval-gated commit.

### [deep-plan - Deep Plan](./skills/deep-plan)

Use this for large or complex work. It produces a versioned `Plan-Contract: 2` handoff with bounded worker packets, exact file ownership, impact closure, and a compact control plane for a separate execution session.

### [deep-execute - Deep Execute](./skills/deep-execute)

Use this in a fresh session to execute a `READY` deep plan. It preserves the parent agent's context by orchestrating bounded workers for repository analysis, implementation, repairs, and verification. It can also prepare legacy `Plan-Contract: 1` plans at runtime.

### [sub-execute - Sub Execute](./skills/sub-execute)

This is the bounded-worker protocol used by `deep-execute`. It is installed alongside `deep-plan` and `deep-execute`, not invoked as a standalone whole-task workflow.

### [generate-feature-spec](./skills/generate-feature-spec)

Use this to turn a product brain dump or source document into a concise, repository-grounded feature specification. It saves the canonical result to `.planning/navoid-plans/<slug>/initial-specs.md` and captures existing-state context, product requirements, testable acceptance criteria, and only the open decisions that block implementation.

### [discuss-gray-areas](./skills/discuss-gray-areas)

Use this after `generate-feature-spec` to investigate the relevant repository, resolve every material product or functional ambiguity with the user, and produce `.planning/navoid-plans/<slug>/discussed-gray-areas.md` plus the canonical merged `.planning/navoid-plans/<slug>/final-specs.md`.

## Choosing a skill

| Task shape | Skill |
| --- | --- |
| Focused implementation or refactor | `qt` |
| Debugging a known failure or regression | `qd` |
| Medium change needing an approved durable plan | `mt` |
| Large, multi-phase work with isolated worker packets | `deep-plan` then `deep-execute` |
| Product brain dump needing a build-ready definition | `generate-feature-spec` |
| Feature specification needing product-decision closure before implementation | `discuss-gray-areas` |
