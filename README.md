# Skills

**Agent skills for real engineering — not vibe coding.**

A collection of composable skills that give AI coding agents structured workflows, quality gates, and disciplined practices. Works with OpenCode, Claude Code, Codex, and any agent in the [skills](https://skills.sh) ecosystem.

## Quickstart

```bash
npx skills@latest add moyu-er/skills
```

The installer detects your installed coding agents and lets you pick where to install. Select the skills you want.

## Skills

### [Engineer](./skills/engineer/README.md)

Daily engineering practices — code review, quality enforcement, cleanup, and structured implementation:

- **[simplify](./skills/engineer/simplify/SKILL.md)** — Post-change quality review via four parallel agents (reuse, quality, efficiency, altitude), then apply fixes. Quality only — does not hunt for bugs.
- **[subagent-implement](./skills/engineer/subagent-implement/SKILL.md)** — Serial implementation of an already-specified task list: each task goes to a fresh implementer subagent and is verified, gated, and committed before the next starts.
- **[parallel-implement](./skills/engineer/parallel-implement/SKILL.md)** — Concurrent implementation of a task DAG: each task dispatches the moment its predecessors land, runs in its own git worktree, and is verified, reviewed, and committed individually.
- **[converge-divergence](./skills/engineer/converge-divergence/SKILL.md)** — Scan for divergent paths (one concern handled by multiple mechanisms), classify each as justified or accidental, and propose convergence with a persistent divergence index. Proposal-only — does not implement.

The bucket's [README](./skills/engineer/README.md) has per-skill usage details and a chooser for the implementation skills.
