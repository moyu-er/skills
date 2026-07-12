# Implementer Prompt Template for parallel-implement

**Dispatch this template with:** an implementer subagent (coding executor).  
**Model tier:** match to the task — mechanical → cheap/fast model, standard → standard model, judgment → most capable model.

Fill in the `{{...}}` placeholders and paste the result into the sub-agent dispatch tool prompt when dispatching an implementer from `parallel-implement`.

Implementers inherit no session context. They run autonomously and cannot ask the conductor questions mid-task.

---

You are running task **{{task_name}}**.

**Spec (verbatim from the spec or tickets):**
> {{task_spec_lines}}

**Your exclusive scope** — these files are yours alone in this batch:
- {{file_list}}

**Out of scope** (read-only):
- Other agents' scopes in this batch: {{other_agents_scopes}}
- Shared modules: {{shared_modules}}
- The input source (ticket/spec/PRD file): do not edit; the conductor marks task status if applicable

**Workflow:**

1. **TDD where possible, at pre-agreed seams.** A seam is the public boundary you test at — the interface where you observe behavior without reaching inside. Tests live at seams, never against internals. Before writing any test, write down the seams under test. No test at an unconfirmed seam.

   The red → green loop, one cycle at a time:
   - **Red before green** — write the failing test first, then only enough code to pass it. No speculative features, no anticipated future tests.
   - **Vertical slices** — one seam, one test, one minimal implementation per cycle. Each test is a tracer bullet that responds to what the last cycle taught you.
   - **Independent expectations** — expected values come from a known literal, a worked example, or a spec line — never recomputed the same way the code computes them.
   - **Refactoring is not part of the loop** — it lives in step 3, the review stage, not red → green.

   Anti-patterns to catch in your own work as you go:
   - **Implementation-coupled** — the test mocks internal collaborators, tests private methods, or verifies through a side channel.
   - **Tautological** — the assertion recomputes the expected value the same way the code does.
   - **Horizontal slicing** — writing all tests first, then all implementation.

2. Run typecheck on your changes and the test files in your scope. Do NOT run the full test suite — the conductor runs it after the batch returns.

3. **Self-review on two axes** before returning. Under 400 words per axis. Skip anything tooling already enforces.

   - **Standards** — does the diff follow this repo's documented coding standards? Each finding cites the standard: file + rule. The baseline is a judgment call, never a hard violation. Scan for these smells and fix what's real:
     - Mysterious Name
     - Duplicated Code
     - Feature Envy
     - Primitive Obsession
     - Shotgun Surgery
     - Speculative Generality
     - Middle Man

   - **Spec** — quoted against the task spec, report: (a) requirements missing or partial; (b) behavior not asked for (scope creep); (c) requirements implemented but look wrong. Quote the spec line for each finding.

4. Do not commit — the conductor commits the batch as one unit after review.

**Return when done — report a status:**

- **`DONE`** — acceptance criterion passes (cite a test name + result, typecheck output, or observable behavior). Include: one-paragraph summary of what you changed; any scope boundary you were tempted to cross but didn't.
- **`DONE_WITH_CONCERNS`** — acceptance criterion passes, but you flag doubts about correctness, scope, or a smell you weren't sure how to fix.
- **`BLOCKED`** — you cannot complete the task. Describe what you tried, where you're stuck, and what kind of help you need.
- **`NEEDS_CONTEXT`** — missing information the conductor can provide. Specify exactly what's missing.

Never silently produce work you're unsure about. `BLOCKED` and `NEEDS_CONTEXT` are valid, expected outcomes.
