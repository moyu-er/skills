# Engineer

Daily engineering practices — design verification, code review, quality enforcement, and cleanup.

## Skills

- **[simplify](./simplify/SKILL.md)** — Launch four parallel agents to review changed code for reuse, quality, efficiency, and altitude, then apply fixes. Quality only — does not hunt for bugs. Use after a change lands, before deeper review.
- **[subagent-implement](./subagent-implement/SKILL.md)** — Serially implement already-specified tasks by delegating each to an implementer subagent, then verifying and accepting each step before continuing. Use when tasks are few or serially dependent and you want a per-step acceptance gate; the task list and order come from context or a given file (any source — not just tickets).
- **[converge-divergence](./converge-divergence/SKILL.md)** — Scan a codebase for divergent paths (the same concern handled by multiple mechanisms that should be one), classify each as justified or accidental, design convergence, and write a persistent divergence index. Proposal-only — does not implement. Use before a refactor sprint or when structural debt has accumulated. Not an implementation skill; sits outside the chooser below.
- **[design-closure](./design-closure/SKILL.md)** — Verify a design is logically closed before implementation by tracing every path end-to-end across five dimensions (data flow, state machine, interface, lifecycle, convergence) in a four-phase workflow: build a structure map, trace each dimension, trace cross-dimension seams, verify findings against cited locations. Produces a closure matrix as proof of completeness. Parallelizable via subagent dispatch for complex designs. User-invoked only. Use after a design document is written, before implementation begins. Not an implementation skill; sits outside the chooser below.

### Choosing an implementation skill

- One unit of work, no acceptance gate wanted → do it directly.
- Few or serially dependent tasks, each verified before continuing → **subagent-implement**.
