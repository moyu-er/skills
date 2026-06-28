# Skills

**Agent skills for real engineering — not vibe coding.**

A collection of composable skills that give AI coding agents structured workflows, quality gates, and disciplined practices. Works with OpenCode, Claude Code, Codex, and any agent in the [skills](https://skills.sh) ecosystem.

## Quickstart

```bash
npx skills@latest add moyu-er/skills
```

The installer detects your installed coding agents and lets you pick where to install. Select the skills you want from each bucket.

## Skills

Skills are organized into buckets under `skills/`. Each bucket has its own `README.md` with usage details, workflows, and prerequisites.

### [OpenSpec](./skills/openspec/README.md)

Spec-driven change management powered by the OpenSpec CLI. Guide an agent through proposal, planning, implementation tracking, verification, and archival — with oracle review at every planning stage.

### [Engineer](./skills/engineer/README.md)

Daily engineering practices — code review, quality enforcement, and cleanup. Currently includes `simplify` for post-change code quality review via parallel subagents.
