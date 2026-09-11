---
name: generate-feature-spec
description: Turns a brain dump or source document into a concise, implementation-ready senior product-manager feature specification after grounding it in the repository. Use when defining, refining, or handing off a product feature for planning and build work.
---

<objective>
Convert an unstructured product brain dump into one precise feature specification that another agent can plan and implement without rediscovering intent. Ground the specification in the repository, preserve confirmed requirements, expose genuine decisions, and avoid invented product behavior.
</objective>

<quick_start>
When given pasted text or a document, read it fully, inspect the relevant repository context, then save and return the completed feature specification. Prefer Markdown unless the user explicitly requests another document format.
</quick_start>

<workflow>
1. **Ingest the source.** Read the complete brain dump or document, including linked or adjacent material needed to interpret it. Extract a private requirements ledger: goals, users, behaviors, constraints, examples, dependencies, open questions, and explicit exclusions. Preserve important wording where precision matters.

2. **Ground it in the repository.** Start with the project instructions and top-level documentation. Use targeted search to find related code, routes, models, APIs, tests, configuration, design artifacts, and existing specifications. Read the most relevant files deeply. Identify what already exists, what is planned, what conflicts, and the conventions the feature must respect. Do not scan the whole repository by default.

3. **Resolve only material ambiguity.** Reconcile the brain dump with repository evidence. Treat the user's source as the product-intent authority and the repository as the implementation-state authority unless the user says otherwise. Ask one concise question when a missing or contradictory decision would change scope, user behavior, or acceptance. Otherwise make a clearly labeled, low-risk assumption only when it follows from supplied evidence.

4. **Create the specification.** Write a self-contained, concise document using the output contract below. Turn each in-scope behavior into objectively testable acceptance criteria. State user-visible outcomes and rules before implementation choices. Include existing behavior only when it affects the feature.

5. **Persist the artifact.** Follow the artifact-persistence rules exactly. Save the same final content that is returned inline; do not save a draft or a truncated version.

6. **Run a completion pass.** Check the private ledger: every confirmed requirement is represented, intentionally excluded, or listed as an open decision. Check that acceptance criteria cover happy paths, important alternatives, validation/errors, permissions or states when relevant, and non-regression expectations. Remove generic filler, speculative architecture, repeated context, and implementation tasks masquerading as requirements.
</workflow>

<artifact_persistence>
The feature workspace is `.planning/navoid-plans/<feature-slug>/`, relative to the repository root. It is the shared working ground for every feature artifact produced or consumed by `generate-feature-spec`, `discuss-gray-areas`, `mt`, `deep-plan`, and `deep-execute`.

The canonical output path is `.planning/navoid-plans/<feature-slug>/initial-specs.md`; `<feature-slug>` is the derived feature slug.

- Use a user-provided slug when one is explicitly supplied. Otherwise derive one from the feature name by lowercasing it, replacing each run of non-alphanumeric characters with a hyphen, collapsing repeated hyphens, and trimming hyphens.
- Once selected, retain the same `<feature-slug>` for all artifacts of this feature. Do not write feature artifacts outside its feature workspace.
- Read an existing target file before writing. Treat it as part of the existing specification status; reconcile it with the new source and repository evidence, retaining still-valid decisions and surfacing meaningful changes.
- Create the target directory when needed and write the complete final Markdown document to `initial-specs.md` using the repository's normal file-editing mechanism.
- In the final response, identify the saved artifact path and also provide the document inline unless the user explicitly asks for file-only output.
</artifact_persistence>

<repository_investigation>
Inspect in this order, stopping when the feature context is sufficient:

- Repository instructions, README, product or architecture docs, and existing specs/roadmaps.
- The closest existing feature and its tests.
- Interfaces touched by the feature: UI, API, schema, background work, configuration, analytics, permissions, and integrations as applicable.
- Recent history only when it clarifies intended or incomplete work.

Report repository evidence with file paths and a short statement of relevance. Never claim a capability exists without evidence.
</repository_investigation>

<output_contract>
Return one document with clear Markdown headings in this order. Omit a section only when it is truly inapplicable; do not add empty headings.

<document_title>
Feature name — Product specification
</document_title>

<summary>
One paragraph: user problem, intended outcome, and the feature boundary.
</summary>

<context_and_status>
State the source material reviewed; repository evidence with paths; and current status: existing, partially implemented, planned, or new. Call out conflicts plainly.
</context_and_status>

<goals_and_scope>
List measurable goals, in-scope work, and explicit non-goals.
</goals_and_scope>

<users_and_scenarios>
Identify affected users/roles and their primary job-to-be-done or end-to-end scenarios.
</users_and_scenarios>

<requirements>
Give numbered requirements grouped by capability. For each requirement, specify trigger, behavior, rules/validation, visible states or failure handling, and any relevant permission, lifecycle, or compatibility constraint. Use examples only where they eliminate ambiguity.
</requirements>

<acceptance_criteria>
Give testable, behavior-focused criteria mapped to the requirements. Use Given/When/Then when it improves clarity. Include no more than the cases needed to define correct behavior.
</acceptance_criteria>

<experience_and_contracts>
Describe only the user flows, UX copy/state rules, data/API/integration contracts, analytics, or non-functional requirements that are necessary for implementation. Separate confirmed contracts from implementation suggestions.
</experience_and_contracts>

<dependencies_and_rollout>
List dependencies, migration/backfill needs, rollout or feature-flag constraints, risks, and observability only when applicable.
</dependencies_and_rollout>

<open_decisions>
List only unresolved, implementation-blocking decisions. For each, give the decision needed, why it matters, and the recommended default when one is evidence-based. If none remain, state that explicitly.
</open_decisions>
</output_contract>

<quality_rules>
- Be decisive about confirmed facts and explicit about uncertainty.
- Do not turn a vague request into a solution design. Name product outcomes and constraints; leave technical design to the planning agent unless a technical contract is already required.
- Do not invent dates, metrics, personas, endpoints, schemas, UI layouts, owners, or dependencies.
- Keep requirements atomic, observable, and internally consistent. Avoid words such as “seamless,” “intuitive,” or “fast” unless made measurable.
- Do not include implementation tickets, estimates, or a plan. This is the handoff artifact that enables those later steps.
- Keep the final document skimmable: dense with decisions, sparse with explanation.
</quality_rules>

<success_criteria>
The result is successful when a planner can identify the exact feature boundary, existing-state impact, required behaviors, testable acceptance criteria, constraints, and any remaining decision without rereading the brain dump or searching the repository for basic context.
</success_criteria>
