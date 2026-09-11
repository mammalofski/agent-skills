# Generate Feature Spec

Turns a product brain dump—pasted into a prompt or supplied in a document—into a concise, implementation-ready feature specification. It grounds the product intent in the repository so the handoff reflects current behavior, conventions, partial work, and real constraints.

## When to use

- A rough idea, meeting notes, customer request, or product brief needs to become build-ready requirements.
- An existing feature needs a clear product definition before planning or implementation.
- A planning agent needs an authoritative, testable handoff instead of raw context.

## How to use

Invoke it with the corpus and point to the repository or document if needed:

```text
/generate-feature-spec
Brain dump: Customers should be able to save a report and share it with their team...

Read the repository and produce the feature specification.
```

The skill investigates relevant repository context, identifies the current specification/implementation status, saves the canonical result to `.planning/navoid-plans/<slug>/initial-specs.md`, and returns a compact Markdown document with scope, requirements, acceptance criteria, and only material open decisions.
