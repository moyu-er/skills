---
name: change-archive
description: Archive a completed OpenSpec change via the openspec archive CLI, after confirming artifact and task completion.
disable-model-invocation: true
---

Archive a completed OpenSpec change: confirm artifact and task completion, run `openspec validate` as the normative-compliance gate, then run `openspec archive` to move the change directory to `archive/YYYY-MM-DD-<name>/` and sync any delta specs to main specs.

`change-review` is **recommended** before archiving, but not enforced. The archive step only checks that artifacts and tasks are complete; it does not require a prior review report. If you want the code-reviewer pass, run `change-review <name>` first.

`openspec status: done` is a file-existence gate, not a normative-compliance gate. The OpenSpec validator catches a different class of problem (e.g. "first sentence must contain SHALL/MUST") that `status` does not. This skill runs `openspec validate --strict` explicitly so the same class of error cannot reach the archive step unawares.

**Input**: Optionally specify a change name. If omitted, infer from conversation context. If ambiguous, run `openspec list --json` and let the user select.

## Steps

1. **Resolve the change**

   - If a name is provided, use it.
   - Otherwise infer from conversation context, or auto-select if only one active change exists.
   - If ambiguous, run `openspec list --json` and ask the user to pick.
   - Show only active (non-archived) changes.

   **Completion criterion**: A change name is in hand.

2. **Check artifact completion status**

   ```bash
   openspec status --change "<name>" --json
   ```
   Parse: `schemaName`, `planningHome`, `changeRoot`, `artifactPaths`, `actionContext`, `artifacts[]`.

   If `actionContext.mode: "workspace-planning"` → STOP and explain that workspace archive is not supported; do not move workspace changes into repo-local archives.

   If any artifact has `status != "done"`:
   - Display warning listing incomplete artifacts.
   - Ask the user to confirm they want to proceed.
   - Continue only if confirmed.

   If all artifacts are `done` but `change-review` has not been run, this is allowed — `change-review` is recommended but not a gate for archive. Note the skipped review in the final summary if the user bypassed it.

   **Completion criterion**: Artifact status is known; any incompleteness is surfaced and confirmed.

3. **Check task completion status**

   Read `tasks.md` from `artifactPaths.tasks.existingOutputPaths` (if it exists). Count `- [ ]` vs `- [x]`.

   If incomplete tasks found:
   - Display warning: count of incomplete tasks.
   - Ask the user to confirm they want to proceed.
   - Continue only if confirmed.

   If no `tasks.md` exists, proceed without task warnings.

   **Completion criterion**: Task status is known; any incompleteness is surfaced and confirmed.

4. **Run `openspec validate` (normative-compliance gate)**

   ```bash
   openspec validate "<name>" --type change --strict --json
   ```

   This step runs whether or not `change-review` was previously executed. The validator is the only thing that enforces rules `openspec status` does not — in particular the "first sentence of `### Requirement:` must contain SHALL/MUST" rule, which is otherwise invisible until archive time.

   If validation reports **any error**:
   - Stop. Do not proceed to archive.
   - Show the exact validator output (file + offending requirement heading + first sentence).
   - For "first-sentence SHALL" failures, the recommended fix is to rewrite the first sentence as `The system SHALL <normative statement>. <optional context after period>.` or `The system SHALL <do X> when <condition>.` rather than `If/When <condition>, the system SHALL <do X>.`.
   - For other failures, follow the validator's recommendation or the report in `openspec-archive-validation-errors-*.md` style if the user has provided one.
   - The user must fix the artifacts and re-run this skill. Do not silently bypass errors.

   Warnings (non-blocking) are still reported in the final summary.

   **Completion criterion**: Validator exits 0, or user has been shown the failures and instructed to fix and retry.

5. **Archive via openspec CLI**

   ```bash
   openspec archive "<name>" --yes
   ```

   The CLI will:
   - Move `changeRoot` to `<planningHome.changesDir>/archive/YYYY-MM-DD-<name>/`
   - Sync delta specs into `openspec/specs/<capability>/spec.md` (unless `--skip-specs`)

   Pass `--skip-specs` if the user explicitly says the change is doc-only / tooling / infrastructure and spec sync is not needed.

   **Completion criterion**: CLI exits 0; the change directory is now under `archive/`.

6. **Show summary**

   Display:
   - Change name
   - Schema that was used
   - Archive location (`archive/YYYY-MM-DD-<name>/`)
   - Spec sync status: synced by CLI, or skipped (with reason)
   - Validate result: passed (and whether `change-review` ran or was skipped)
   - Any warnings noted (incomplete artifacts / tasks / skipped change-review / validate warnings)

   **Completion criterion**: Summary is shown.