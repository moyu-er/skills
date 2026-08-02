---
name: parallel-implement
description: Use when there are 3+ already-specified implementation tasks with declared dependency edges, independent enough to run concurrently.
disable-model-invocation: true
---

# Parallel Implement

Execute many already-specified tasks concurrently. The conductor keeps a ready queue over the dependency DAG and dispatches each task the moment its predecessors are accepted and its scope is clear — no batches. Each implementer works in its own git worktree; the conductor integrates, verifies, and commits each task individually.

This skill schedules and dispatches; it does not plan, decompose, or invent dependencies. Every task needs scope, dependency edges, and acceptance criteria — if any are missing or ambiguous, ask the user.

- 3+ tasks with declared dependency edges → this skill.
- Few or serially dependent tasks → `subagent-implement`. One task → `implement`.

## Roles

- **Conductor** (the main agent) — the only committer to the main tree, the only task-marker, the only verifier. Before starting, records the original base SHA and any pre-existing tree changes (`git status --porcelain`); pre-existing changes are never staged.
- **Implementer** (`./implementer-prompt.md`) — one per task. Works in its own worktree on a scratch branch; never touches the main tree. Returns a status, a change manifest, and a complexity report. Cheapest model tier that can do the task.
- **Reviewer** (`./reviewer-prompt.md`) — reviews a task's branch diff (complex gate) or the whole diff (final review). Read-only. Always the most capable reasoning model.

## The loop

Repeat until every task is accepted or blocked:

1. **Dispatch everything ready.** A task is ready when all its predecessors are accepted **and** its declared scope is disjoint from every running task's scope. For each ready task:
   - `git worktree add <path> -b wt/<task> HEAD` — based on current HEAD, so it contains all accepted work.
   - Dispatch one implementer in the background, template filled, pointed at the worktree.
   - Shared generated files (lockfiles, dependency manifests) may belong to at most one running task; if two ready tasks declare them, serialize those tasks or ask the user.
2. **On each implementer completion, triage the status:**
   - `DONE` → integrate (step 3). `DONE_WITH_CONCERNS` → integrate; correctness or scope concerns make the task complex (step 3's gate).
   - `NEEDS_CONTEXT`, context-shaped `BLOCKED` → supply context, re-dispatch into the same worktree. Not a failure; never capped.
   - Capability-shaped `BLOCKED` → re-dispatch one model tier up; already at top tier → mark the task blocked and tell the user.
   - Plan-shaped `BLOCKED` (incoherent or contradictory task) → tell the user immediately; keep other tasks flowing.
3. **Integrate and accept** the completed task, in this order:
   - **Manifest check** — `git diff --name-only HEAD...wt/<task>` against the change manifest. Paths outside it are a scope violation: send back to revert or justify.
   - **Gate** — classify from the implementer's complexity report + manifest + the branch diff:
     - **trivial** (no behavior change) → accept.
     - **standard** (contained, single subsystem, no concerns or deviations reported) → you review the branch diff: every acceptance criterion visible in it, plus a baseline smell scan.
     - **complex** (anything else) → dispatch the reviewer on the branch diff. `NEEDS_FIXES`: critical findings → send the implementer back with them; optional → fix now or defer to the final review. Reviewer `BLOCKED` → supply context, re-dispatch once, then escalate.
     - Defer upward to the implementer's self-assessed tier; ambiguous → one tier up. Record the tier and a one-line rationale — passing tests never substitute for this.
   - **Merge** — `git merge --no-commit --no-ff wt/<task>` into the main tree (conflict → `git merge --abort`, rebase the scratch branch onto HEAD, send the implementer back to resolve). Run typecheck plus the manifest-named tests on the integrated tree. New red → send back.
   - **Commit, then mark** — one commit: match repo style (`git log --oneline -10` first), stage the manifest paths only. After it lands, mark the task done, remove the worktree (`git worktree remove`), and re-check the ready queue (step 1).
4. **Failure budget:** max 3 send-backs per task — one counter covering verify red, scope violations, and reviewer criticals. Exhausted → mark the task blocked, tell the user, keep independent tasks flowing. A blocked task never unblocks its dependents.
5. **Quiescence** — nothing running, nothing ready, tasks remain → the rest are blocked or starved by blocked predecessors. Stop and report.

## Final review (always runs)

Full test suite + typecheck on the final tree, then dispatch the reviewer against the **original base**, handing it the deferred optional findings to triage — the only place cross-task integration is fully seen. `NEEDS_FIXES` → one fix-up implementer with the complete findings list, fixes land as one new commit, max 2 rounds, then escalate.
