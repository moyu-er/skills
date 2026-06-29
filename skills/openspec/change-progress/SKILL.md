---
name: change-progress
description: Mark an implementation task complete in the current OpenSpec change's tasks.md. Fire after finishing a task when working through a plan, when the plan instructs to call `change-progress`, when the user says "mark done" / "task complete" / "update progress" / "change-progress", or when verifying a unit of work from tasks.md is done before continuing to the next.
---

Mark a completed task in the current OpenSpec change's `tasks.md`: flip `- [ ]` to `- [x]` for the named task, then show progress. Called by the orchestrating agent after each implementation task verifies — never mark a task you have not actually completed and verified.

**Input**: A change name and a task identifier. The change name is usually supplied by `tasks.md`'s header instruction ("call `change-progress <change-name>`"). The task identifier is a task number (e.g., `3.2`), a checkbox line fragment (e.g., the task description), or a `- [ ]` line you just made true.

## `change-progress` vs `change-adapt`

This skill is **not** for changing the plan. Use it only when the task was implemented as planned and you are ready to mark it done.

| | `change-progress` | `change-adapt` |
|---|---|---|
| **Purpose** | Mark a task as done in `tasks.md`. | Update planning artifacts when the plan itself is wrong. |
| **When to call** | After a task is implemented and verified. | Before continuing, when implementation reveals the plan needs to change. |
| **What it edits** | Only `tasks.md` (flips `- [ ]` → `- [x]`). | `proposal.md`, `specs/`, `design.md`, and/or `tasks.md` — whichever is wrong. |
| **Does it change the plan?** | No. It records that the plan was followed. | Yes. It changes the plan and documents why. |
| **Required output** | A flipped checkbox and a progress report. | Updated artifacts plus an **adaptation note** explaining what was discovered, why the change was made, and the new expected behavior. |
| **Typical mistake** | Forgetting to call it after a task. | Silently editing code to match a new understanding without updating artifacts. |

**Rule of thumb**: If the only thing that happened is "the task is done", call `change-progress`. If what happened is "the task exposed that the plan was wrong", call `change-adapt` first, record the rationale, then continue implementation and eventually call `change-progress`.

## Steps

1. **Resolve the change**

   - If a name is provided, use it.
   - Otherwise infer from the conversation context — typically the `tasks.md` you were just reading names the change in its `# Tasks: <change-name>` header.
   - If you cannot find a name and only one active change exists, run `openspec list --json` and use it. If multiple exist, STOP and tell the user you need a change name.

   **Completion criterion**: A change name is in hand.

2. **Check implementation state and locate `tasks.md`**

   ```bash
   openspec status --change "<name>" --json
   ```

   Parse: `changeRoot`, `artifactPaths`, `artifacts[]` (id + status), `isComplete`, `actionContext`.

   **Handle states before locating the task:**

   - If `actionContext.mode: "workspace-planning"` → STOP and explain that workspace progress tracking is not supported in this slice. Do not edit workspace changes as if they were repo-local.
   - If no `tasks` key exists in `artifactPaths` or `artifactPaths.tasks.existingOutputPaths` is empty → STOP: there is nothing to mark. Suggest running `change-plan <name>` to create tasks.
   - If `isComplete: true` OR every checkbox in `tasks.md` is already `- [x]` → STOP: all tasks are done. Suggest running `change-review <name>` before archiving.

   Read `tasks.md` from `artifactPaths.tasks.existingOutputPaths`.

   **Completion criterion**: You hold the `tasks.md` path and the change is in a state where progress can be marked.

3. **Find the target checkbox**

   Read `tasks.md` and locate the task the user/agent just completed:
   - Match by task number (e.g., `3.2`)
   - Or match by description fragment from the task that was just done

   If no `- [ ]` checkbox remains in the file → STOP: all tasks are already complete. Suggest `change-review <name>` before archiving.

   If you cannot uniquely identify the target (multiple matching checkboxes, or the task isn't in the file), STOP and report the candidates to resolve.

   **Completion criterion**: Exactly one `- [ ]` line is identified for flipping.

4. **Verify the task was actually done**

   Before flipping the checkbox, confirm the work:
   - If this turn implemented the task, the implementation should already be in the working tree (run `git status` / `git diff` if in doubt).
   - If tests were part of the task, confirm they pass.
   - If you only just heard the user *say* the task is done without evidence, request the evidence (test output, file listing) before marking. A checkbox flip is a claim of completion — never mark what you have not verified.

   **Completion criterion**: The task's acceptance signal (test pass, file exists, behavior confirmed) is satisfied.

5. **Flip the checkbox**

   Edit `tasks.md`: replace the target `- [ ]` with `- [x]`. Preserve everything else on the line and in the file.

   **Completion criterion**: The file now contains `- [x]` on the target line and the rest is unchanged.

6. **Sync with OpenSpec CLI status**

   ```bash
   openspec status --change "<name>" --json
   ```

   Confirm that the `tasks` artifact reports `status: "done"` if all checkboxes are now `[x]`, or `status: "ready"` / `"in_progress"` if tasks remain. If the CLI status does not reflect the checkbox flip (for example, because of an independent task state cache), re-read `artifactPaths.tasks.existingOutputPaths` and ensure the file you edited is the one the CLI is tracking.

   **Completion criterion**: The CLI status and the edited `tasks.md` are consistent.

7. **Show progress**

   ```bash
   openspec status --change "<name>"
   ```

    Display:
    - The task that was just marked done
    - Overall progress: "N/M tasks complete"
    - Next pending task, if any
    - If all tasks complete: suggest running `change-review <change-name>` for a final check before archiving.
    - If the next pending task reveals that the plan itself needs to change: suggest calling `change-adapt <change-name>` before implementing that task.

   **Completion criterion**: Progress is shown.