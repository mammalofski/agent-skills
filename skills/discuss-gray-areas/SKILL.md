---
name: discuss-gray-areas
description: Resolves product and functional ambiguity in a repository-grounded feature specification before implementation. Use after generate-feature-spec has produced initial-specs.md and a feature needs a complete product-decision review and final-specs.md handoff.
---

<objective>
Turn a generated feature specification into an implementation-ready product contract. Research the relevant code and specifications, resolve every material functional uncertainty with the user, and leave no product decision for the implementation agent to make.
</objective>

<input>
Use `.planning/navoid-plans/<feature-slug>/initial-specs.md` as the required source of product intent. This feature workspace is the shared working ground for every feature artifact produced or consumed by `generate-feature-spec`, `discuss-gray-areas`, `mt`, `deep-plan`, and `deep-execute`. Reuse its `<feature-slug>` and read the document in full. If it is absent, ask the user to run `generate-feature-spec` or provide the missing specification; do not invent a substitute.
</input>

<steps>
1. **Load the starting contract.** Read the initial specification, repository instructions, and related plans/specifications. Extract a private ledger of each confirmed behavior, stated assumption, open decision, and explicit exclusion.

2. **Map the feature to existing behavior.** Use targeted search to find the closest current user flow, then trace its entry points, rules, state transitions, permissions, stored data, integrations, and tests. Read adjacent specifications that govern the same product area. Stop once the existing product behavior and its constraints are clear; do not scan the repository indiscriminately.

3. **Compare intent and evidence.** For every initial requirement, decide whether repository or specification evidence confirms it, reveals an existing behavior that it changes, or conflicts with it. Treat `initial-specs.md` as the authority for requested product intent and the repository as the authority for existing-state facts. A conflict is a product decision for the user, not an implementation detail.

4. **Find material product gray areas.** Inspect only behavior that changes what a user can do, observe, receive, or expect: actors and permissions; triggers; happy and alternative paths; defaults and validation; state/lifecycle changes; ownership, visibility, and retention; cancellation, failure, and retry outcomes; compatibility/rollout behavior; feature boundaries; and acceptance outcomes. Do not turn architecture, code structure, naming, or implementation choices into questions.

5. **Resolve evidence-backed items.** Record a decision directly when explicit source or repository evidence answers it. Do not ask the user to reconfirm a conclusion that is unambiguous and within the stated product intent.

6. **Ask the complete question set.** Batch every remaining material decision that the same investigation could reveal. For each numbered question, provide: the decision required, the user-visible consequence, relevant evidence or conflict with file paths, and a recommended answer only when evidence supports one. Keep questions answerable and grouped by flow. Ask a follow-up only when the user's answer creates a new material ambiguity.

7. **Close the decision loop.** Add each user answer to the ledger, update affected requirements and acceptance criteria, then re-run the end-to-end gray-area check. Repeat steps 4–7 until all material functional decisions and conflicts are explicitly resolved. If the user delegates a decision, record the chosen decision and its scope. Do not publish the final artifacts while an implementation-blocking product decision remains open.

8. **Write `discussed-gray-areas.md`.** Save `.planning/navoid-plans/<feature-slug>/discussed-gray-areas.md` with concise sections for: inputs and repository evidence; gray areas assessed; questions and confirmed answers; evidence-backed decisions; conflicts and resolutions; and closure stating that no implementation-blocking product gray areas remain.

9. **Write `final-specs.md`.** Save `.planning/navoid-plans/<feature-slug>/final-specs.md` as the standalone canonical specification. Start with `initial-specs.md`, preserve every still-valid requirement, and integrate each resolved decision into the relevant scope, requirement, user flow, contract, and acceptance-criteria section. Retain its document structure; replace its open-decisions content with an explicit statement that none remain. Do not make an implementation agent reconstruct behavior from the discussion document or conversation.
</steps>

<constraints>
- Preserve the original feature scope unless the user explicitly changes it.
- Keep every feature artifact in `.planning/navoid-plans/<feature-slug>/`; do not create a parallel discussion or specification artifact elsewhere.
- Be decisive about confirmed facts and explicit about uncertainty; do not manufacture gray areas or make generic discovery questions.
- Keep `discussed-gray-areas.md` as the concise decision audit and `final-specs.md` as the final contract, not a Q&A transcript.
- Do not add implementation plans, estimates, or speculative technical designs.
</constraints>

<done_condition>
`final-specs.md` lets the next implementation agent act without choosing product behavior: users, scope, rules, flows, edge outcomes, compatibility constraints, and testable acceptance criteria are clear. `discussed-gray-areas.md` explains how each material ambiguity was resolved.
</done_condition>
