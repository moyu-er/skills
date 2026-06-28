---
name: change-review
description: Verify implementation matches the OpenSpec change artifacts before archiving. Dispatches a dedicated code reviewer subagent for thorough inspection, then runs openspec validate for artifact-level checks. Optional — run when you want a final check.
disable-model-invocation: true
---

**Compatibility note**: This skill dispatches a `general-purpose` subagent with the reviewer prompt below. If your agent framework does not provide a `general-purpose` subagent type, fall back to `code-reviewer` or the closest available general subagent type with the same prompt.

Verify an OpenSpec change across two independent passes: a dedicated code reviewer subagent inspects the implementation diff against the artifacts, then `openspec validate` checks artifact-level consistency. Findings from both are combined into a single report.

**Input**: Optionally specify a change name. If omitted, infer from conversation context. If ambiguous, run `openspec list --json` and let the user select.

## Steps

1. **Resolve the change**

   - If a name is provided, use it.
   - Otherwise infer from conversation context, or auto-select if only one active change exists.
   - If ambiguous, run `openspec list --json` and ask the user to pick.

   **Completion criterion**: A change name is in hand.

2. **Collect artifacts and diff**

   ```bash
   openspec status --change "<name>" --json
   ```

   Confirm `actionContext.mode: "repo-local"`. If `workspace-planning`, STOP.

   Parse `artifactPaths` from the status JSON. Read every available artifact from `artifactPaths.<id>.existingOutputPaths` — do not assume standard filenames, use what the CLI returns.

   **Load project rules** (if they exist). These constrain how code should be written in this repository; pass them to the reviewer:
   - `.claude/rules/*.md`
   - `CLAUDE.md` in the repo root
   - `AGENTS.md` in the repo root

   If none exist, pass `"No project rules provided"` as `{PROJECT_RULES_CONTENT}`.

     **Gather the implementation diff** (the agent runs this, not the subagent):

     1. Pick a base commit that represents the state **before this change started**. The base must be the change's own starting point, not the branch fork point:
        - Infer from the change directory: find the first commit that created or touched the change directory (e.g., `openspec/changes/<name>/`) or the files referenced in `artifactPaths`. Use the parent of that commit as `BASE_SHA`.
          - Example: `git log --reverse --format=%H -- <change-dir-or-files> | head -1` gives the first touching commit; its parent is the base.
          - If that first commit is the root commit, use the empty tree:
            - Unix: `git hash-object -t tree /dev/null`
            - Windows: `$null | git hash-object -t tree --stdin`
        - If the user or agent explicitly provided a `--base` commit, use that instead.
        - If inference fails (no history, no change directory, etc.), fall back to `git merge-base HEAD origin/main` (or `origin/master` if your default branch is `master`).
        - If that also fails (no upstream, no remote, etc.), use `git rev-parse HEAD~1`.
        - If the repo has only one commit, use the empty tree as above.
      2. Capture the current state: `HEAD_SHA = git rev-parse HEAD`.
      3. Run `git diff --stat <BASE_SHA>..<HEAD_SHA>`.
      4. Decide how much diff to inline based on size:

         - Count the lines of the full diff: `git diff <BASE_SHA>..<HEAD_SHA> | wc -l` (Unix) or `(git diff <BASE_SHA>..<HEAD_SHA> | Measure-Object -Line).Lines` (PowerShell).
         - **Small change** (< ~1,500 diff lines): inline the full diff with `git diff <BASE_SHA>..<HEAD_SHA>`.
         - **Large change** (≥ ~1,500 diff lines): keep the inline diff small to avoid context bloat:
           - Inline only `git diff --stat`.
           - Identify high-risk files from the stat (new files, files with many changes, files touched by specs/design) and inline their full diffs.
           - Tell the reviewer the remaining files are available via `git diff <BASE_SHA>..<HEAD_SHA> -- <file>` and that they may run git commands to inspect them as needed.
      5. If there are uncommitted changes, also run `git diff HEAD` and append the working-tree diff (or its stat + high-risk files for large changes).

      **Completion criterion**: All artifacts read; diff content captured and ready to pass to the reviewer.

3. **Dispatch the code reviewer subagent**

   Use the `task` tool to dispatch a `general-purpose` subagent with the review template at [`code-reviewer.md`](code-reviewer.md) in this skill folder. Fill the placeholders:

   - `{CHANGE_NAME}` — the change name from step 1
   - `{PROPOSAL_SUMMARY}` — the opening 2-3 sentences of proposal.md
   - `{PROPOSAL_CONTENT}` — full proposal.md content
   - `{SPECS_CONTENT}` — all `specs/*/spec.md` files concatenated
   - `{DESIGN_CONTENT}` — full design.md, or `"No design.md for this change"`
   - `{TASKS_CONTENT}` — tasks.md content
   - `{PROJECT_RULES_CONTENT}` — content of project rules files (`.claude/rules/*.md`, `CLAUDE.md`, `AGENTS.md`), or `"No project rules provided"`
   - `{DIFF_CONTENT}` — the full `git diff` output from step 2 (both `--stat` and full diff)
   - `{BASE_SHA}` — the base commit SHA
   - `{HEAD_SHA}` — the head commit SHA

    ```text
    task(subagent_type="general-purpose", prompt=<filled code-reviewer.md>)
    ```

   Wait for the reviewer result before proceeding.

   **Completion criterion**: Reviewer returns a verdict.

4. **Run openspec validate**

   First run the non-strict check:

   ```bash
   openspec validate "<name>" --type change --json
   ```

   Then run the strict check:

   ```bash
   openspec validate "<name>" --type change --strict --json
   ```

   Parse both JSON outputs and classify failures:

   - **CRITICAL** — structural / schema failures: missing required artifact, missing `### Requirement:`, missing `#### Scenario:`, malformed delta headers, unreadable JSON, etc. These indicate the artifact set is broken or incomplete.
   - **IMPORTANT** — content correctness failures: requirement text does not match the described behavior, scenario missing WHEN/THEN, ambiguous wording.
   - **MINOR** — wording / normative verb nitpicks: e.g. "must contain SHALL/MUST" when the requirement text already contains SHALL, or other linter-style formatting complaints that do not change meaning. If the validator reports a SHALL/MUST issue but the requirement text visibly contains SHALL/MUST, treat it as a validator false-positive, record it as Minor with a note, and do **not** force a rewrite.

   **Completion criterion**: CLI exits, validation results captured and classified.

5. **Check task completion**

   Read `tasks.md` from `artifactPaths.tasks.existingOutputPaths`. Count `- [ ]` vs `- [x]`. Incomplete tasks are CRITICAL — the agent should complete them or use `change-progress` to mark them if already done.

   **Completion criterion**: Task status is known.

6. **Act on findings**

   - **CRITICAL** issues (from reviewer, validate, or incomplete tasks): fix immediately, then re-run the relevant check. Do not generate the final report until Critical issues are resolved or have a stated rationale.
   - **IMPORTANT** issues: fix or document why you chose not to.
   - **MINOR** issues: note for later; do not block.

   **Completion criterion**: Every CRITICAL and IMPORTANT issue has been resolved or has a stated rationale.

7. **Generate unified report**

   Combine the reviewer's findings, openspec validation results, and task completion status:

   ```
   ## Verification Report: <change-name>

   ### Code Review
   [Reviewer's Strengths, Issues, Recommendations, Assessment]

   ### openspec validate
   [Validation results — passing or list of failures]

   ### Task Completion
   [X/Y tasks complete — list any incomplete ones]

   ### Final Assessment
   [Combined verdict: Ready / With fixes / Not ready]
   ```

   Every issue has `file:line` references where applicable. No vague phrases.

   **Completion criterion**: Report displayed with clear final verdict.

## Graceful Degradation

Use the keys in `artifactPaths` from `openspec status --json` to decide what to pass to the reviewer, not assumed filenames:

- No `specs` key in `artifactPaths` → pass `"No specs for this change"` as `{SPECS_CONTENT}`. The reviewer skips spec-related checks.
- No `design` key in `artifactPaths` → pass `"No design.md for this change"` as `{DESIGN_CONTENT}`. The reviewer skips design adherence checks.
- No `proposal` key in `artifactPaths` → pass `"No proposal.md for this change"` as `{PROPOSAL_CONTENT}`. The reviewer skips proposal alignment checks.
- Only `tasks` exists → the reviewer checks task completion only; note which checks were skipped.
- Full artifacts (`proposal`, `specs`, `design`, `tasks`) → all checks run.
- Always state in the final report which checks were skipped and why.
