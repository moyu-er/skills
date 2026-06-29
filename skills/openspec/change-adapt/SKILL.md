---
name: change-adapt
description: Update an existing OpenSpec change's artifacts during implementation when a design, spec, or task issue is discovered. Preserves artifact coherence and records the rationale for the change.
---

Update an OpenSpec change's artifacts mid-implementation. Call this skill when implementation reveals that the plan no longer matches reality — a task is unworkable in its current form, a design decision needs to change, a requirement needs to be added or removed, or the scope needs to be clarified.

This skill does **not** implement code. It is a governance step that records the discovered change back into the artifact set so that `tasks.md`, `design.md`, `specs/`, and (when necessary) `proposal.md` remain the ground truth for the implementation that is actually being built.

**Input**: A change name (kebab-case) and a description of what was discovered during implementation. If the change name is omitted, infer from conversation context; if ambiguous, run `openspec list --json` and ask the user to pick.

## When to Call This Skill

- A task in `tasks.md` is impossible, misleading, or needs to be split/combined/reordered.
- A design decision in `design.md` turns out to be wrong, incomplete, or no longer applicable.
- A requirement in `specs/<capability>/spec.md` is missing, wrong, or needs to be reworded.
- Implementation reveals a scope-level issue that requires updating `proposal.md`.
- The implementer is about to diverge from the artifacts and needs to record why.

Do **not** use this skill for typo fixes or formatting-only edits. Do **not** use it to mark task progress — that is `change-progress`.

## `change-progress` vs `change-adapt`

These two skills sit at the same point in the implementation workflow but solve different problems. Do not confuse them.

| | `change-progress` | `change-adapt` |
|---|---|---|
| **Purpose** | Mark a task as done in `tasks.md`. | Update planning artifacts when the plan itself is wrong. |
| **When to call** | After a task is implemented and verified. | Before continuing, when implementation reveals the plan needs to change. |
| **What it edits** | Only `tasks.md` (flips `- [ ]` → `- [x]`). | `proposal.md`, `specs/`, `design.md`, and/or `tasks.md` — whichever is wrong. |
| **Does it change the plan?** | No. It records that the plan was followed. | Yes. It changes the plan and documents why. |
| **Required output** | A flipped checkbox and a progress report. | Updated artifacts plus an **adaptation note** explaining what was discovered, why the change was made, and the new expected behavior. |
| **Typical mistake** | Forgetting to call it after a task. | Silently editing code to match a new understanding without updating artifacts. |

**Rule of thumb**: If the only thing that happened is "the task is done", call `change-progress`. If what happened is "the task exposed that the plan was wrong", call `change-adapt` first, record the rationale, then continue implementation and eventually call `change-progress`.

1. **Resolve the change**

   - If a name is provided, use it.
   - Otherwise infer from conversation context, or auto-select if only one active change exists.
   - If ambiguous, run `openspec list --json` and ask the user to pick.
   - Show only active (non-archived) changes.

   **Completion criterion**: A change name is in hand.

2. **Load the current artifact set**

   ```bash
   openspec status --change "<name>" --json
   ```

   Parse `changeRoot`, `artifactPaths`, and `artifacts[]` (id + status). Read every existing artifact from `artifactPaths.<id>.existingOutputPaths` — at minimum `proposal.md`, all `specs/<capability>/spec.md` files, `design.md`, and `tasks.md`.

   **Completion criterion**: You hold the current state of every existing artifact.

3. **Classify the discovered issue**

   Based on the user's description of what implementation revealed, decide which artifact(s) need to change:

   | Issue type | Affected artifact | Example |
   |---|---|---|
   | Task ordering, scope, or executability | `tasks.md` | Task 3 must come before task 2; a task needs to be split. |
   | Design decision is wrong or incomplete | `design.md` | Chosen approach fails under load; add an alternative or change the decision. |
   | Requirement is wrong, missing, or mis-scoped | `specs/<capability>/spec.md` | A scenario was missed; a SHALL statement needs to change. |
   | Scope or motivation has changed | `proposal.md` | A new capability is required, or a promised capability is no longer needed. |

   If multiple artifacts are affected, update them in dependency order: `proposal.md` (if needed) → `specs/` → `design.md` → `tasks.md`. Later artifacts must remain consistent with earlier ones.

   **Completion criterion**: The affected artifact(s) are identified and the update order is clear.

4. **Update the affected artifacts and record the rationale**

   For each affected artifact:

   a. **Read it** from its existing output path.

   b. **Apply the minimum change** needed to reflect the new reality. Preserve the existing structure, section headers, and OpenSpec formatting rules.

   c. **Record an adaptation note**. Every artifact change must be accompanied by a short, explicit note explaining:
   - **What implementation revealed** — the concrete problem or new fact.
   - **Why the artifact changed** — how the old content was wrong or incomplete.
   - **What the new expected behavior is** — the replacement decision, requirement, or task.

      Use the standard format:

      ```markdown
      > **Adaptation note**: <what was discovered>. <why the change was made>. <new expected behavior>. Updated during implementation via `change-adapt`.
      ```

      Place the note immediately next to the changed content, not buried at the end of the file:
      - For `design.md`: under the relevant `## Decisions` entry that changed, or in a new `## Revisions` section if multiple decisions changed.
      - For `specs/<capability>/spec.md`: directly under the changed `### Requirement:` or `#### Scenario:` heading.
      - For `tasks.md`: directly under the changed task, or in a new `## Revisions` section if task order/structure changed globally.
      - For `proposal.md`: in a new `## Scope Revisions` section, or next to the changed section if the change is localized.

      **A content change without an adaptation note is incomplete.** Do not silently edit text; the rationale is part of the change.

   d. **Ensure normative compliance**. Every `### Requirement:` first sentence must contain `SHALL` or `MUST` with no leading subordinate clause (`If`, `When`, `While`, `Given`). Every requirement must have at least one `#### Scenario:`.

   **Completion criterion**: All affected artifacts are updated, each changed section has a visible adaptation note, and all artifacts are compliant with OpenSpec formatting rules.

5. **Run the normative-compliance gate**

   ```bash
   openspec validate "<name>" --type change --strict --json
   ```

   If validation reports any error:
   - Stop and show the exact validator output.
   - Fix the artifact error and re-run until zero errors.
   - Do not proceed while validation fails.

   **Completion criterion**: `openspec validate --strict` reports zero errors.

6. **Assess scope impact**

   If `proposal.md` was updated or a capability was added/removed:
   - Summarize the scope change for the user.
   - Ask whether to continue within the same change or create a new change for the added scope.
   - If the user chooses a new change, stop `change-adapt` and guide them to `change-propose` for the new scope.

   **Completion criterion**: Any scope-level change is acknowledged and either folded into the current change or assigned to a new change.

7. **Show summary and next step**

   Display:
   - Change name and location
   - Which artifacts were updated and why
   - Validation result
   - Next step: "Continue implementation. Remember to call `change-progress <name>` after each verified task. If the adaptation changed task order or content, resume from the updated `tasks.md`."

   **Completion criterion**: Summary shown and next step communicated.

## Guardrails

- **Minimum change**: Only update artifacts that are actually affected by the discovered issue. Do not rewrite unrelated sections.
- **One issue at a time**: If multiple independent issues are discovered, handle them one by one unless they clearly belong to the same root cause.
- **Preserve evidence**: Always record what implementation revealed and why the artifact changed. Future reviewers (and your future self) need the rationale.
- **Validate after every artifact change**: Run `openspec validate --strict` after the full adaptation is done; if you updated multiple artifacts, re-validate after each batch to catch drift early.
- **Do not implement code here**: This skill updates planning artifacts only. If you find yourself writing application code, stop and route to an implementation skill.
- **Do not archive here**: Archiving is `change-archive`'s job. After adaptation and continued implementation, run `change-review` before archiving.

## What This Skill Is Not

- Not a replacement for `change-progress` (task checkbox flipping).
- Not a replacement for `change-plan` (initial artifact generation from a proposal).
- Not a replacement for `change-review` (final implementation vs artifact verification).
- Not an implementation skill. It records change; it does not write production code.
