# Discuss Gray Areas

Reviews the repository and the `initial-specs.md` produced by `generate-feature-spec` to find and resolve product and functional ambiguity before implementation.

## When to use

Use after `generate-feature-spec` when a feature needs a complete decision review rather than more technical design. The skill investigates relevant existing behavior and specifications, asks the user only the material product questions, and saves:

- `.planning/navoid-plans/<feature-slug>/discussed-gray-areas.md` — evidence, questions, answers, and decisions.
- `.planning/navoid-plans/<feature-slug>/final-specs.md` — the standalone, merged implementation handoff.

## How to use

Invoke it with the feature slug or its generated specification:

```text
/discuss-gray-areas
Review .planning/navoid-plans/saved-report/initial-specs.md, resolve all product gray areas with me, and produce the final specifications.
```
