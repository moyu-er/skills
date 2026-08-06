# Engineer

Engineering skills implement or modify code: structured implementation and post-change quality cleanup.

## Skills

- **[simplify](./simplify/SKILL.md)** — Launch four parallel agents to review changed code for reuse, quality, efficiency, and altitude, then apply fixes. Quality only — does not hunt for bugs. Use after a change lands, before deeper review.
- **[subagent-implement](./subagent-implement/SKILL.md)** — Serially implement already-specified tasks by delegating each to an implementer subagent, then verifying and accepting each step before continuing. User-invoked only. Use when tasks are few or serially dependent and you want a per-step acceptance gate; the task list and order come from context or a given file (any source — not just tickets).

### Choosing an implementation skill

- One unit of work, no acceptance gate wanted → do it directly.
- Few or serially dependent tasks, each verified before continuing → **subagent-implement**.
