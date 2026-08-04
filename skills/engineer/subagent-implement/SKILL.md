---
name: subagent-implement
description: Use when there are already-specified implementation tasks in context or a given file, to be done one at a time with each step verified and accepted before continuing.
disable-model-invocation: true
---

# Subagent Implement

`implement` run serially across an existing task list: each task goes to a fresh implementer subagent, and the main agent verifies, accepts, and commits each step before starting the next.

The task list and its order already exist — in context, or in a file the user points to. Every task needs a scope and acceptance criteria. This skill executes the list; it never decomposes, orders, or invents tasks. If the list or its order is missing, stop and ask.

- Few or serially dependent tasks, each verified before continuing → this skill.
- Many independent tasks with dependency edges → `parallel-implement`.
- One unit of work → `implement`.

## Roles

- **Main agent** — the only git writer, the only task-marker, the only verifier. Before step 1, records the original base SHA and any pre-existing tree changes (`git status --porcelain`); pre-existing changes are never staged.
- **Implementer** (`./implementer-prompt.md`) — one per task. Edits the tree, never commits, never marks done. Returns a status, a change manifest, and a complexity report. Cheapest model tier that can do the task.
- **Reviewer** (`./reviewer-prompt.md`) — reviews a step's diff (complex gate) or the whole diff (final review). Read-only. Always the most capable reasoning model.

## Per task

1. **Base** — `git rev-parse HEAD`. This step's diff base.
2. **Dispatch** one implementer, the template filled for this task.
3. **Triage the status:**
   - `DONE` → verify. `DONE_WITH_CONCERNS` → verify; correctness or scope concerns make the step complex (step 5).
   - `NEEDS_CONTEXT`, context-shaped `BLOCKED` → supply context, re-dispatch. Not a failure; never capped.
   - Capability-shaped `BLOCKED` → re-dispatch one model tier up; already at top tier → escalate to the user.
   - Plan-shaped `BLOCKED` (incoherent or contradictory task) → escalate immediately, no retry.
4. **Verify** on the working tree:
   - `git status --porcelain` against the change manifest — changed paths outside it are a scope violation: send back to revert or justify.
   - Typecheck plus the manifest-named tests. New red (green at this step's base) → send back. Pre-existing red → record, flag to the user, proceed.
5. **Gate** — classify **after** implementation, from the implementer's complexity report + manifest + the real diff:
   - **trivial** (no behavior change) → accept on verify alone.
   - **standard** (contained, single subsystem, no concerns or deviations reported) → you review the diff: every acceptance criterion visible in it, plus a baseline smell scan.
   - **complex** (anything else) → dispatch the reviewer against this step's base. `NEEDS_FIXES`: critical findings → send the implementer back with them; optional → fix now or defer to the final review. Reviewer `BLOCKED` → supply context, re-dispatch once, then escalate.
   - Defer upward to the implementer's self-assessed tier; ambiguous → one tier up. Record the tier and a one-line rationale for every step — passing tests never substitute for this.
6. **Commit, then mark** — one commit per step: match repo style (`git log --oneline -10` first); the message describes the change, not the workflow (`step N` / `s3` / `I2` mean nothing outside this run); stage the manifest paths only. After it lands, mark the task done in the source list.

**Failure budget:** max 3 send-backs per step — one counter covering verify red, scope violations, and reviewer criticals. Exhausted → stop, report the step and its status, ask the user. Steps are serial: a stuck step blocks everything after it, so never skip ahead.

## Final review (always runs)

Full test suite + typecheck on the final tree, then dispatch the reviewer against the **original base**, handing it the deferred optional findings to triage. This is the only place cross-step integration gets seen. `NEEDS_FIXES` → one fix-up implementer with the complete findings list, fixes land as one new commit, max 2 rounds, then escalate.
