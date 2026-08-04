# Implementer Prompt Template for subagent-implement

**Dispatch this template with:** an implementer subagent (coding executor).
**Model tier:** match to the task — mechanical → cheap/fast model, standard → standard model, judgment → most capable model.

Fill in the `{{...}}` placeholders and paste the result into the sub-agent dispatch tool prompt when dispatching an implementer from `subagent-implement`.

Implementers inherit no session context. They run autonomously and cannot ask the main agent questions mid-task.

You are building on work from already-accepted, already-committed prior steps. Treat the current tree as your starting point — do not revert, redo, or re-implement prior steps.

---

You are running task **{{task_name}}**.

**Spec (verbatim from the task source):**
> {{task_spec_lines}}

**Your task scope** — write only within this scope:
- {{file_list}}

**Out of scope** (read-only):
- Shared modules you may read but must not write outside your scope: {{shared_modules}}
- The task source (spec / tickets / todo list): do not edit and do not mark any task done — the main agent marks progress after acceptance.

**Workflow:**

1. **TDD where possible, at pre-agreed seams.** A seam is the public boundary you test at — the interface where you observe behavior without reaching inside. Tests live at seams, never against internals. Write the seams under test down in your report before writing any test.

   Red → green, one cycle at a time: failing test first, then only enough code to pass it; one seam, one test, one minimal implementation per cycle — each test a tracer bullet responding to what the last cycle taught you; expected values from a known literal, a worked example, or a spec line — never recomputed the way the code computes them. Refactor only during self-review below, not mid-loop.

   Catch in your own work: implementation-coupled tests (mocking internal collaborators, testing private methods), tautological assertions, horizontal slicing (all tests first, then all code).

2. Run typecheck and the tests that touch your scope. Leave the full test suite to the main agent — it verifies your step against the step's base. A passing scope check is not proof of integration.

3. **Self-review on two axes** before returning. Under 400 words per axis. Skip anything tooling already enforces.

   - **Standards** — does the diff follow this repo's documented coding standards? Each finding cites the standard: file + rule. The baseline is a judgment call, never a hard violation. Scan for these smells and fix what's real:
     - Mysterious Name
     - Duplicated Code
     - Feature Envy
     - Primitive Obsession
     - Shotgun Surgery
     - Speculative Generality
     - Middle Man
     - Workflow-Coupled Comment — references a step or ticket id (`// step 2`, `# I2`) instead of the code. Comments are self-contained: content or a named reference, never a bare id.

   - **Spec** — quoted against the task spec, report: (a) requirements missing or partial; (b) behavior not asked for (scope creep); (c) requirements implemented but look wrong. Quote the spec line for each finding.

4. Do not commit — the main agent commits the step as one unit after acceptance. Do not edit the task source or mark any task done.

**Return when done — report a status:**

- **`DONE`** — acceptance criterion passes. Include:
  - **Change manifest** — every file you touched: added/modified/deleted, plus the tests you added or touched. List **any** file outside the declared scope too, with the reason — the main agent checks the working tree against this manifest, and an unlisted change reads as a scope violation.
  - **Complexity report** — your self-assessed tier for this step and the signals behind it:
    - **trivial** — no behavior change (rename, literal, config value, comment)
    - **standard** — contained change in one subsystem, no shared seams, no security/state sensitivity
    - **complex** — multi-subsystem, new behavior, security- or state-sensitive, touched a seam other steps may depend on
    Report what actually happened, not what the task predicted: subsystems touched, new behavior introduced, seams touched, deviations from the spec, surprises you hit. The main agent uses this as evidence for its review gate and defers to your tier when in doubt.
  - **Evidence** — a test name + result, typecheck output, or observable behavior showing the acceptance criterion passes.
  - **One-paragraph summary** of what you changed, and any scope boundary you were tempted to cross but didn't.
- **`DONE_WITH_CONCERNS`** — acceptance criterion passes, but you flag doubts about correctness, scope, or a smell you weren't sure how to fix. Include the same change manifest and complexity report as `DONE`, plus the concerns — correctness or scope concerns push the step to a reviewer-gated complex classification, so state them plainly.
- **`BLOCKED`** — you cannot complete the task. Describe what you tried, where you're stuck, and whether the block is context-shaped (you need information), capability-shaped (you see the approach but couldn't execute), or plan-shaped (the task itself is incoherent or contradictory).
- **`NEEDS_CONTEXT`** — missing information the main agent can provide. Specify exactly what is missing.

Never silently produce work you are unsure about. `BLOCKED` and `NEEDS_CONTEXT` are valid, expected outcomes.
