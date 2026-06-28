---
name: change-propose
description: Create an OpenSpec change and write all its artifacts (proposal, specs, design, tasks) end-to-end from session/document context.
disable-model-invocation: true
---

Create an OpenSpec change and write all planning artifacts in one invocation. This skill is equivalent to running `change-new` followed by `change-plan`, but the workflow is inlined here so it does not depend on nested skill invocation.

**Input**: A change name (kebab-case) OR a description of what to build. If omitted, derive from conversation context.

## Steps

1. **Resolve the change name**

   - If a kebab-case name is provided, use it.
   - If a description is provided, derive a kebab-case name (e.g., "add user authentication" → `add-user-auth`).
   - If neither, ask the user what they want to build.
   - If a change with that name already exists, suggest continuing it via `change-plan` instead.

   **Completion criterion**: A valid kebab-case name is in hand.

2. **Create the change scaffold**

   ```bash
   openspec new change "<name>"
   ```

   Verify the change directory was created (check CLI output or `openspec status --change "<name>" --json` for `changeRoot`).

   **Completion criterion**: CLI exits 0 and the change directory exists on disk.

3. **Write `proposal.md`**

   a. Get the proposal template and output path:
      ```bash
      openspec status --change "<name>" --json
      openspec instructions proposal --change "<name>" --json
      ```
      → `artifactPaths.proposal.resolvedOutputPath` is where `proposal.md` goes.
      → `template` is the section structure; `instruction` is field guidance; `context` and `rules` are constraints for you — do NOT copy them into the file.

   b. Extract from the current conversation or referenced documents:
      - **Why**: the problem or opportunity being addressed
      - **What Changes**: specific new capabilities, modifications, removals
      - **Capabilities**: kebab-case names; each one will become a `specs/<name>/spec.md`
      - **Impact**: affected code areas, APIs, dependencies

   c. Fill the template. Apply `instruction` constraints but never copy `context`/`rules` verbatim.

   **Completion criterion**: `proposal.md` exists at the resolved path, every template section filled, no `<!-- ... -->` placeholders remaining.

4. **Review `proposal.md`**

   Dispatch a read-only `oracle` subagent with the prompt at [`proposal-review-prompt.md`](proposal-review-prompt.md) in this skill folder. Fill `{CHANGE_NAME}`, `{PROPOSAL_PATH}`, and `{CHANGE_DIR}`.

   Wait for the verdict (Ready / With fixes / Not ready).

   - **CRITICAL** issues: fix immediately, re-dispatch oracle if the fix is substantial.
   - **IMPORTANT** issues: fix or document why you chose not to.
   - **MINOR** issues: fix opportunistically.

   **Completion criterion**: Every CRITICAL and IMPORTANT issue has been resolved or has a stated rationale.

5. **Write `specs/`, `design.md`, and `tasks.md`**

   a. Get artifact status:
      ```bash
      openspec status --change "<name>" --json
      ```
      Parse `changeRoot`, `artifactPaths`, `artifacts[]` (id + status), `isComplete`, and `applyRequires`.

   b. Loop through ready artifacts in dependency order. For the `spec-driven` schema this is: `specs` → `design` → `tasks`.

      For each `ready` artifact:
      - Get instructions: `openspec instructions <artifact-id> --change "<name>" --json`
      - Read dependency artifacts listed in `dependencies`.
      - Write the file using `template` as structure and `instruction` as guidance.
      - Fill every section. No `<!-- ... -->` placeholders may remain.
      - After each artifact, re-run `openspec status --change "<name>" --json` to confirm the next artifact is `ready`.

      Artifact-specific rules:
      - **`specs/<capability>/spec.md`** — one file per capability from the proposal. Use the capability's kebab-case name. Every `### Requirement:` must have at least one `#### Scenario:`. Use SHALL/MUST.
      - **`design.md`** — document technical decisions with rationale. Include `## Decisions` with alternatives considered. Cut if the change is too small for cross-cutting decisions, but err on the side of including it.
      - **`tasks.md`** — checkbox tasks in dependency order. **Mandatory header**:

        ```markdown
        # Tasks: <change-name>

        > **Progress tracking**: After completing each task below and verifying it passes,
        > invoke `change-progress <change-name>` to mark the checkbox before moving on.
        > Do not batch-mark: complete one, mark one, then continue.
        ```

        Each task must name concrete files and a concrete action.

        When the change involves a data flow, control flow, multi-step process, or dependencies between components, tasks must also reflect **flow coherence**: earlier tasks must produce the data/state/interfaces that later tasks consume, and the ordering must make the hand-off explicit. Do not list tasks as an unrelated bag of files; explain or name the passing data/interface at each step so the implementer can see the chain.

   **Completion criterion**: For each artifact in `applyRequires` (typically `tasks`), `openspec status` reports `status: "done"` AND `openspec validate "<name>" --type change --strict --json` reports zero errors.

   **Validate-as-gate**: `openspec status: done` only confirms file existence — it does NOT check normative compliance. The OpenSpec validator's strict gate must be run explicitly before treating the planning phase as complete. If validate reports errors, fix them and re-run until zero errors. Do not mark this skill complete on `status: done` alone.

6. **Review the planning artifacts**

   Dispatch a read-only `oracle` subagent with the prompt at [`artifacts-review-prompt.md`](artifacts-review-prompt.md) in this skill folder. Fill `{CHANGE_NAME}`, `{CHANGE_DIR}`, `{PROPOSAL_PATH}`, `{SPECS_GLOB}`, `{DESIGN_PATH}`, and `{TASKS_PATH}`.

   Wait for the verdict. Handle issues as in step 4.

   **Completion criterion**: Every CRITICAL and IMPORTANT issue has been resolved or has a stated rationale.

7. **Show final summary**

   ```bash
   openspec status --change "<name>"
   ```

   Display:
   - Change name and location
   - All artifacts created and reviewed
   - Next step: "Artifacts complete. Implement using your preferred method — each task in `tasks.md` guides you to call `change-progress <name>` after each verified task. When all tasks are done, run `change-review <name>` before archiving."

   **Completion criterion**: Status shown and next step communicated.
