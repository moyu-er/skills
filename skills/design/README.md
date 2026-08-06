# Design

Design skills produce decisions, documents, proposals, and design verification. They do not modify production code. When design work approves a change, implementation moves to the [Engineer](../engineer/README.md) bucket; see its README for the implementation chooser.

## Skills

- **[deliberate](./deliberate/SKILL.md)** — Resolve an open technical or design choice through a grounded recommendation shaped by its constraints and tradeoffs. Model-invoked.
- **[codify](./codify/SKILL.md)** — Canonicalize ambiguous project terminology and preserve already-settled, qualifying architectural decisions as ADRs. Model-invoked.
- **[deliberate-with-docs](./deliberate-with-docs/SKILL.md)** — Conduct and preserve a deliberation through `deliberate`, `codify`, and a numbered decision record. User-invoked only.
- **[converge-divergence](./converge-divergence/SKILL.md)** — Scan a codebase for divergent paths (the same concern handled by multiple mechanisms that should be one), classify each as justified or accidental, design convergence, and write a persistent divergence index. Proposal-only — does not implement. User-invoked only. Use before a refactor sprint or when structural debt has accumulated.
- **[design-closure](./design-closure/SKILL.md)** — Verify a design is logically closed before implementation by tracing every path end-to-end across five dimensions (data flow, state machine, interface, lifecycle, convergence) in a four-phase workflow: build a structure map, trace each dimension, trace cross-dimension seams, verify findings against cited locations. Produces a closure matrix as proof of completeness. Parallelizable via subagent dispatch for complex designs. User-invoked only. Use after a design document is written, before implementation begins.
