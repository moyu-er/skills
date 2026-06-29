---
name: change-new
description: Create an OpenSpec change and write its proposal from session/document context.
disable-model-invocation: true
---

Create an OpenSpec change scaffold and write its `proposal.md` by extracting requirements from the current conversation or referenced documents. Then dispatch a read-only subagent to review the proposal before finishing.

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

3. **Get the proposal template and output path**

   ```bash
   openspec status --change "<name>" --json
   ```
   → `artifactPaths.proposal.resolvedOutputPath` is where `proposal.md` goes.

   ```bash
   openspec instructions proposal --change "<name>" --json
   ```
   → `template` is the section structure; `instruction` is the field-by-field guidance; `context` and `rules` are constraints for you — do NOT copy them into the file.

   **Completion criterion**: You hold the template and the resolved output path.

4. **Write `proposal.md` from context**

   Read the current conversation history (or referenced documents) to extract:
   - **Why**: the problem or opportunity being addressed
   - **What Changes**: specific new capabilities, modifications, removals
   - **Capabilities**: kebab-case names; each one will become a `specs/<name>/spec.md`
   - **Impact**: affected code areas, APIs, dependencies

   Fill the template using these as content. Apply `instruction` constraints but never copy `context`/`rules` blocks verbatim — they are guidance for you, not file content.

   **Completion criterion**: `proposal.md` exists at the resolved path, every template section filled, no `<!-- ... -->` placeholders remaining.

5. **Dispatch the proposal reviewer**

   Use the `task` tool to dispatch a read-only `oracle` subagent with the review prompt at [`proposal-review-prompt.md`](proposal-review-prompt.md) in this skill folder. Fill the template placeholders:
   - `{CHANGE_NAME}` — the change name
   - `{PROPOSAL_PATH}` — absolute path to `proposal.md`
   - `{CHANGE_DIR}` — the change directory

   ```text
   task(subagent_type="oracle", prompt=<filled proposal-review-prompt.md>)
   ```

   Wait for the oracle result before proceeding.

   **Completion criterion**: Oracle returns a verdict (Ready / With fixes / Not ready).

6. **Act on review feedback**

   - **CRITICAL** issues: fix immediately, re-dispatch oracle if the fix is substantial.
   - **IMPORTANT** issues: fix or document why you chose not to.
   - **MINOR** issues: fix opportunistically.

   **Completion criterion**: Every CRITICAL and IMPORTANT issue has been resolved or has a stated rationale.

7. **Show final status**

   ```bash
   openspec status --change "<name>"
   ```

   Display:
   - Change name and location
   - Proposal created
   - Review verdict
    - Next step: "Run `change-plan <name>` to create specs, design, and tasks. After that, implement using your preferred method. Mark progress with `change-progress <name>` after each verified task. If a task reveals the plan itself is wrong, call `change-adapt <name>` first to update the affected artifact and record the rationale."

   **Completion criterion**: Status shown and next step communicated.