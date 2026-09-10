# Sub Execute (`sub-execute`)

## Purpose

`sub-execute` is the context-isolated worker protocol used by the `deep-execute` orchestrator. It provides the compact dispatch schema, immutable assignment-file schema, and quality gate used to brief workers without pulling implementation detail into the parent's context.

It is not a whole-task executor. Each invocation handles one bounded packet.

## Modes

| Mode | Purpose |
| --- | --- |
| `prepare` | Convert a legacy plan into bounded runtime worker packets |
| `implement` | Implement one packet with focused tests and verification |
| `repair` | Fix named findings or failed checks within assigned ownership |
| `verify` | Independently review, test, and validate assigned scope |

## Context-preservation model

- Workers receive a minimal dispatch pointing to one checksummed assignment file.
- Assignment files contain exact context paths and prerequisite result artifacts.
- Workers read source directly instead of receiving large pasted context.
- Detailed evidence is persisted to `results/<packet-id>.md`.
- Returns are limited to eight concise bullets / about 300 words.
- Fresh workers are used for implementation, repair, and independent verification.

## Safety

- File ownership and forbidden overlap are explicit.
- Workers never modify the git index, stage, commit, push, merge, deploy, switch branches, or clean files.
- Workers never modify plan, packet, journal, lock, or another worker's result artifacts.
- Nested delegation is forbidden unless explicitly authorized.
- Material deviations are returned as `BLOCKED: new-plan-required`.

## Key file

| File | Purpose |
| --- | --- |
| [`SKILL.md`](SKILL.md) | Worker modes, assignment prompt schema, quality gate, execution workflow, and result contract. |
