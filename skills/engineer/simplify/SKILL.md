---
name: simplify
description: Review the changed code for reuse, quality, efficiency, and altitude, then apply the fixes. Quality only — it does not hunt for bugs.
---

# Simplify: Code Review and Cleanup

Review all changed files for reuse, quality, efficiency, and altitude. Fix any issues found.

## Phase 0 — Gather the Diff

Run `git diff @{upstream}...HEAD` (or `git diff main...HEAD` / `git diff HEAD~1` if there's no upstream) to get the unified diff under review. If there are uncommitted changes, or the range diff is empty, also run `git diff HEAD` and include the working-tree changes in scope. If a PR number, branch name, or file path was passed as `$ARGUMENTS`, review that target instead.

## Phase 1: Launch Four Review Agents in Parallel

Use the Agent tool to launch all four agents concurrently in a single message. Pass each agent the full diff so it has the complete context.

Each agent returns its findings with `file`, `line`, a one-line `summary`, and the concrete cost (what is duplicated, wasted, or harder to maintain).

### Agent 1: Code Reuse Review

For each change:

1. **Search for existing utilities and helpers** that could replace newly written code. Look for similar patterns elsewhere in the codebase — common locations are utility directories, shared modules, and files adjacent to the changed ones.
2. **Flag any new function that duplicates existing functionality.** Suggest the existing function to use instead.
3. **Flag any inline logic that could use an existing utility** — hand-rolled string manipulation, manual path handling, custom environment checks, ad-hoc type guards, and similar patterns are common candidates.

### Agent 2: Code Quality Review

Review the same changes for hacky patterns:

1. **Redundant state**: state that duplicates existing state, cached values that could be derived, observers/effects that could be direct calls
2. **Parameter sprawl**: adding new parameters to a function instead of generalizing or restructuring existing ones
3. **Copy-paste with slight variation**: near-duplicate code blocks that should be unified with a shared abstraction
4. **Leaky abstractions**: exposing internal details that should be encapsulated, or breaking existing abstraction boundaries
5. **Stringly-typed code**: using raw strings where constants, enums (string unions), or branded types already exist in the codebase
6. **Unnecessary JSX nesting**: wrapper Boxes/elements that add no layout value — check if inner component props (flexShrink, alignItems, etc.) already provide the needed behavior
7. **Unnecessary comments**: comments explaining WHAT the code does (well-named identifiers already do that), narrating the change, or referencing the task/caller — delete; keep only non-obvious WHY (hidden constraints, subtle invariants, workarounds)

### Agent 3: Efficiency Review

Review the same changes for efficiency:

1. **Unnecessary work**: redundant computations, repeated file reads, duplicate network/API calls, N+1 patterns
2. **Missed concurrency**: independent operations run sequentially when they could run in parallel
3. **Hot-path bloat**: new blocking work added to startup or per-request/per-render hot paths
4. **Recurring no-op updates**: state/store updates inside polling loops, intervals, or event handlers that fire unconditionally — add a change-detection guard so downstream consumers aren't notified when nothing changed. Also: if a wrapper function takes an updater/reducer callback, verify it honors same-reference returns (or whatever the "no change" signal is) — otherwise callers' early-return no-ops are silently defeated
5. **Unnecessary existence checks**: pre-checking file/resource existence before operating (TOCTOU anti-pattern) — operate directly and handle the error
6. **Memory**: unbounded data structures, missing cleanup, event listener leaks
7. **Overly broad operations**: reading entire files when only a portion is needed, loading all items when filtering for one

### Agent 4: Altitude Review

Check that each change is implemented at the right depth, not as a fragile band-aid:

1. **Special cases on shared infrastructure** — new conditionals layered on a shared utility or API are a sign the fix isn't deep enough. Prefer generalizing the underlying mechanism over adding special cases that each caller must check.
2. **Wrong owner** — logic placed in a component or module that shouldn't own it. The right fix may belong in a lower-level abstraction or a different part of the stack entirely.
3. **Inverted dependency** — the change adds a dependency in the wrong direction (e.g. a utility importing from a feature module). The cost is invisible coupling that blocks future refactoring.

For each altitude issue, name the deeper fix — don't just flag the problem.

## Phase 2: Fix Issues

Wait for all four agents to complete. Deduplicate findings that point at the same line or mechanism, then fix each remaining issue directly. Skip any finding whose fix would change intended behavior, require changes far outside the reviewed diff, or that you judge to be a false positive — note it and move on, do not argue with the finding.

When done, briefly summarize what was fixed and what was skipped (or confirm the code was already clean).
