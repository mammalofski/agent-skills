# Medium Task (`mt`)

> **Beta:** this skill is under development and not yet perfect. For now, I recommend using [`gsd-quick`](https://github.com/open-gsd/gsd-core/tree/next/skills/gsd-quick) instead.

## Purpose

`mt` runs one medium-complexity task through an orchestrated **plan, approve, execute, verify, commit** loop. The main agent orchestrates only: a planner sub-agent writes a durable plan, the user approves it, and a separate executor sub-agent implements and verifies it.

It is the middle weight between `qt` and the `deep-plan` + `deep-execute` pair.

| Skill | Use when | Shape |
| --- | --- | --- |
| `qt` | Focused change, clear path | One agent does everything |
| `mt` | Medium feature or change worth a durable, reviewable plan | Orchestrator + planner sub-agent + executor sub-agent, one session |
| `deep-plan` + `deep-execute` | Large multi-phase work needing packets, parallel workers, and locks | Separate sessions, many workers |

## Where the plan lives

```text
.planning/navoid-plans/<slug>/
├── plan.md                     # MT-Plan-Contract 1, DRAFT then READY, immutable once READY
├── execution-report.md         # MT-Result-Contract 1 implementation evidence
├── execution-report-fix-<n>.md # Fix-round evidence, never overwritten
└── verification-report.md      # Independent read-only review evidence
```

The directory is shared with the `deep-plan` skill but the contract differs, so an `mt` plan is not a `/deep-execute` package. The plan is written to be sufficient on its own: approve it and continue in the same session, or hand `plan.md` to a fresh session.

## Starting mode

Start `mt` in either interaction mode; both are first-class and no mode switch is required up front.

- **Spec or planning mode**: the planner still produces the full plan. If the host blocks artifact writes, the plan is presented for approval first, then persisted the moment the session enters execution mode. Approving the plan is what moves the run from planning to implementation.
- **Normal agent mode**: artifacts are writable immediately and the plan gate is an explicit approval question. There is still no implementation before approval.

## Workflow summary

1. Lock the task, confirm `mt` is the right weight, record the git baseline, and protect pre-existing user changes.
2. Dispatch the planner sub-agent; route its plan-shaping questions to the user and resume it with the answers.
3. Review the plan against the gate checklist, send findings back to the planner, and get explicit approval to implement.
4. Leave planning mode, persist the plan if the host blocked artifact writes, and confirm it is `READY` on disk.
5. Dispatch the executor sub-agent to implement the plan, run its checks, and write an evidence report.
6. Dispatch an independent read-only verifier only when a risk trigger fires, then repair findings within the round limit.
7. Ask whether everything is good. Every reported bug or requested change becomes a `USER-<n>` fix round, handled by resuming the executor that did the work or by dispatching a fresh one, then re-verified and brought back to the same question.
8. Commit only approved, session-scoped, allowlisted paths once the user is satisfied.

## Key files

| File | Purpose |
| --- | --- |
| [`SKILL.md`](SKILL.md) | Orchestrator workflow, gates, dispatch prompts, verification triggers, commit rules. |
| [`mt-planner.md`](mt-planner.md) | Planner sub-agent protocol and the `MT-Plan-Contract: 1` plan format. |
| [`mt-executor.md`](mt-executor.md) | Executor sub-agent protocol for `implement`, `fix`, and `verify`, plus the `MT-Result-Contract: 1` report format. |

Both protocol files ship inside this skill directory; the orchestrator passes their absolute paths to each sub-agent.

## Safety

- The orchestrator never writes plan content, source, or tests, and never reads full diffs or logs.
- No execution before current-session plan approval, and no commit before the satisfaction gate.
- A `READY` plan is immutable; material change requires a new suffixed package.
- Sub-agents cannot stage, commit, push, deploy, switch branches, or clean the tree.
- Pre-existing staged, unstaged, and untracked work stays untouched, and `.planning/` is excluded from staging by default.
- Question rounds, plan revisions, and orchestrator-initiated fix or verify rounds are capped at two each. User-requested change rounds are uncapped but always end in verification and the satisfaction gate.
- Material change requests are refused as fix rounds and routed back to planning instead.
