---
name: change-plan
description: Write specs, design, and tasks artifacts for an existing OpenSpec change from its proposal. Also use to continue a partially planned change by creating the next ready artifact (e.g., "continue the change" or "write the remaining artifacts").
disable-model-invocation: true
---

Write the `specs/`, `design.md`, and `tasks.md` artifacts for an existing OpenSpec change by reading its `proposal.md`, then dispatch a read-only subagent to review the multi-file consistency before finishing.

**Input**: Optionally specify a change name. If omitted, infer from conversation context. If ambiguous, run `openspec list --json` and let the user select.

## Steps

1. **Resolve the change**

   - If a name is provided, use it.
   - Otherwise infer from conversation context, or auto-select if only one active change exists.
   - If ambiguous, run `openspec list --json` and ask the user to pick.

   **Completion criterion**: A change name is in hand.

2. **Get artifact status**

   ```bash
   openspec status --change "<name>" --json
   ```
   Parse: `changeRoot`, `artifactPaths`, `artifacts[]` (each has `id` + `status`), `isComplete`. Read `proposal.md` from `artifactPaths.proposal.existingOutputPaths`.

   If `isComplete: true` → STOP and tell the user all artifacts are already created. Suggest implementation or `change-review`.

   If `proposal.md` does not exist → STOP and tell the user to run `change-new` first.

   **Completion criterion**: You have the change root, artifact list, and the proposal content.

3. **Loop: write each ready artifact in dependency order**

   The `spec-driven` schema dependency order is: `proposal` (done) → `specs` → `design` → `tasks`. If some artifacts are already `done` from a previous run, skip them and continue with the next `ready` artifact. After each artifact, re-run `openspec status --change "<name>" --json` to confirm the next is `ready` before proceeding.

   For each `ready` artifact:

   a. **Get its template and instructions**
      ```bash
      openspec instructions <artifact-id> --change "<name>" --json
      ```
      Parse: `template` (the file skeleton), `instruction` (field guidance), `resolvedOutputPath`, `dependencies` (completed artifacts to read for context). `context` and `rules` are constraints for you — do NOT copy them into the file.

   b. **Read dependency artifacts** listed in `dependencies` for context.

   c. **Write the file** using `template` as structure, `instruction` as guidance, and the dependency artifacts (especially `proposal.md`) as source material. Fill every section. No `<!-- ... -->` placeholders may remain.

   d. **Artifact-specific rules**:

      - **`specs/<capability>/spec.md`** — one file per capability listed in the proposal's Capabilities section. Use the capability's kebab-case name (not the change name). Every `### Requirement:` must have at least one `#### Scenario:`. Use SHALL/MUST. Scenarios use exactly 4 hashtags.

      - **`design.md`** — document technical decisions with rationale. Focus on "why this approach". Reference the proposal for motivation and specs for requirements. Include `## Decisions` with alternatives considered. Cut if a change is small enough that no cross-cutting design decision exists — but err on the side of including it.

      - **`tasks.md`** — checkbox tasks in dependency order. **MANDATORY HEADER**:

        ```markdown
        # Tasks: <change-name>

        > **Progress tracking**: After completing each task below and verifying it passes,
        > invoke `change-progress <change-name>` to mark the checkbox before moving on.
        > Do not batch-mark: complete one, mark one, then continue.
        ```

        Each task must name concrete files to create/modify and a concrete action — never "implement feature X". Tasks should be small enough to complete in one focused session.

        When the change involves a data flow, control flow, multi-step process, or dependencies between components, tasks must also reflect that **flow coherence**: earlier tasks must produce the data/state/interfaces that later tasks consume, and the ordering must make the hand-off explicit. Do not list tasks as an unrelated bag of files; explain or name the passing data/interface at each step so the implementer can see the chain.

   **Completion criterion**: For each artifact in `applyRequires` (typically `tasks`), `openspec status` reports `status: "done"` AND `openspec validate "<name>" --type change --strict --json` reports zero errors.

   **Validate-as-gate**: `openspec status: done` only confirms file existence — it does NOT check normative compliance. The OpenSpec validator's strict gate must be run explicitly before treating the planning phase as complete. If validate reports errors, fix them (in this case the artifact set is still mutable) and re-run until zero errors. Do not mark this skill complete on `status: done` alone.

4. **Dispatch the artifacts reviewer**

   Use the `task` tool to dispatch a read-only `oracle` subagent with the review prompt at [`artifacts-review-prompt.md`](artifacts-review-prompt.md) in this skill folder. Fill the template placeholders:
   - `{CHANGE_NAME}` — the change name
   - `{CHANGE_DIR}` — the change directory
   - `{PROPOSAL_PATH}` — absolute path to `proposal.md`
   - `{SPECS_GLOB}` — the concrete spec file paths from `artifactPaths.specs.existingOutputPaths`, newline-separated (e.g., `specs/user-auth/spec.md\nspecs/session-mgmt/spec.md`). Never pass a glob pattern — the oracle reads each path with the Read tool.
   - `{DESIGN_PATH}` — absolute path to `design.md` if it exists, or the literal text `NO_DESIGN_MD` if the change was too small to warrant one. The review-prompt handles both cases.
   - `{TASKS_PATH}` — absolute path to `tasks.md`

   ```text
   task(subagent_type="oracle", prompt=<filled artifacts-review-prompt.md>)
   ```

   Wait for the oracle result before proceeding.

   **Completion criterion**: Oracle returns a verdict (Ready / With fixes / Not ready).

5. **Act on review feedback**

   - **CRITICAL** issues: fix immediately, re-dispatch oracle if the fix is substantial.
   - **IMPORTANT** issues: fix or document why you chose not to.
   - **MINOR** issues: fix opportunistically.

   **Completion criterion**: Every CRITICAL and IMPORTANT issue has been resolved or has a stated rationale.

6. **Show final status**

   ```bash
   openspec status --change "<name>"
   ```

   Display:
   - Change name and location
   - Artifacts created (specs, design, tasks)
   - Review verdict
   - Next step: "Artifacts complete. Implement using your preferred method — each task in `tasks.md` guides you to call `change-progress <name>` after each verified task. When all tasks are done, run `change-review <name>` before archiving."

   **Completion criterion**: Status shown and next step communicated.