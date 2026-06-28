# OpenSpec Skills — Agent Notes

## Prompt Duplication Rule

The following review prompts are intentionally duplicated so that each skill is self-contained after installation:

| Master location | Copy location | Why copied |
|---|---|---|
| `change-new/proposal-review-prompt.md` | `change-propose/proposal-review-prompt.md` | `change-propose` inlines the `change-new` workflow and must not rely on cross-skill paths at runtime. |
| `change-plan/artifacts-review-prompt.md` | `change-propose/artifacts-review-prompt.md` | `change-propose` inlines the `change-plan` workflow and must not rely on cross-skill paths at runtime. |

When editing either master prompt, **update its copy in `change-propose/` to match** before considering the change complete. Keep the contents byte-for-byte identical unless there is a deliberate reason for divergence.

To verify the copies are still in sync after any edit:

```bash
# From skills/openspec/
diff change-new/proposal-review-prompt.md change-propose/proposal-review-prompt.md
diff change-plan/artifacts-review-prompt.md change-propose/artifacts-review-prompt.md
```

Both diffs must be empty. If they are not, copy the master file over the `change-propose/` copy.

## change-propose Inlining Rule

`change-propose/SKILL.md` is the **single model-invoked entry point** for creating a full change end-to-end. It is equivalent to running `change-new` followed by `change-plan`. To avoid relying on nested skill invocation:

- The full step sequence from `change-new` and `change-plan` is duplicated inside `change-propose/SKILL.md`.
- `change-new` and `change-plan` are marked `disable-model-invocation: true`. They remain available for human users who want the step-by-step path, but the model should not reach for them automatically.

When you change any of the following in `change-new` or `change-plan`, you **must** mirror the change in `change-propose/SKILL.md`:

- Step order or completion criteria.
- `openspec` CLI commands or JSON fields parsed from status/instructions output.
- Artifact-specific rules for `specs/`, `design.md`, or `tasks.md` (including flow-coherence requirements).
- Review feedback handling rules (CRITICAL / IMPORTANT / MINOR).
- Subagent dispatch instructions or prompt placeholder lists.

## Data Flow / Process Flow Requirement

Any change that involves data flow, control flow, multi-step processes, or component-to-component dependencies must express that flow coherence in `tasks.md`:

- Task order must match the real flow: producer tasks come before consumer tasks.
- Each task must make its input/output contract explicit (data shape, interface, state precondition).
- Hand-off points between tasks must be visible in the task text or in short connecting sentences.

The `artifacts-review-prompt.md` checks this in both `change-plan` and `change-propose`. Keep those checks identical.

## Normative-Compliance Gate (validate as gate, not as status)

`openspec status: done` and "validation passes" are two different things:

- `openspec status` tracks file existence and metadata — it does **not** check normative compliance.
- `openspec validate --strict` is the only thing that enforces the OpenSpec spec rules, including the rule that catches the most common archive-time blocker: **the first sentence of every `### Requirement:` block must contain `SHALL` or `MUST`**.

`change-plan` and `change-propose` must treat `validate --strict` (zero errors) as a hard completion criterion in addition to `status: done`. `change-archive` must run `validate --strict` as an explicit step before invoking the archive CLI; on any error it must stop, surface the validator output, and instruct the user to fix and retry rather than silently bypassing the error.

### First-sentence SHALL rule (the most common archive-time blocker)

The OpenSpec validator judges a `### Requirement:` block by the text before the first `.` followed by whitespace or end of line. Subordinate-clause openings (`If …`, `When …`, `While …`, `Given …`) count as the first sentence — even if the main clause later in the sentence contains `SHALL`, the validator reports "must contain SHALL or MUST" and `openspec archive` is blocked.

Acceptable first sentences:

- `The system SHALL <normative statement>. <optional context after period>.`
- `The system SHALL <do X> when <condition>.`
- `The agent SHALL <do X> for <trigger>.`

Unacceptable first sentences (will block archive):

- `If <condition>, the system SHALL <do X>.`
- `When <condition>, the agent SHALL <do X>.`
- `While <condition>, the system SHALL <do X>.`

The `proposal-review-prompt.md` and `artifacts-review-prompt.md` both enforce this rule. If you edit one, keep both copies in sync.

## Verification Checklist Before Finishing an Edit

- [ ] If you touched `change-new/proposal-review-prompt.md`, copied it to `change-propose/proposal-review-prompt.md`.
- [ ] If you touched `change-plan/artifacts-review-prompt.md`, copied it to `change-propose/artifacts-review-prompt.md`.
- [ ] If you touched `change-new/SKILL.md` or `change-plan/SKILL.md`, checked whether `change-propose/SKILL.md` needs the same update.
- [ ] If you changed anything related to the validate-as-gate or first-sentence SHALL rule, updated the corresponding section above.
- [ ] Ran the two `diff` commands above to confirm prompt copies are in sync.
