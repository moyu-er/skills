---
name: design-closure
description: Trace every path in a design document to confirm it connects end-to-end before implementation begins. User-invoked only.
disable-model-invocation: true
---

# design-closure

Verify a design is logically closed before implementation. Not reading for issues — tracing every path to confirm it connects end-to-end.

## When to use

After a design document is written, before implementation begins. When you need to verify no orphan states, missing callers, data appearing from nowhere, objects never released, or paths diverging without justification.

**Input check.** A design document describes structures (data, states, interfaces, objects) and their relationships — not just requirements. If the input lacks describable structures, report "not a design document — cannot trace" and stop.

**Input handling.** If the design spans multiple files, read all and treat the union as the design. If it mixes implemented code with unimplemented design, implemented code is ground truth — trace the design against it, flag contradictions. If it's embedded in code comments or ADRs, extract the structural descriptions before mapping.

## The method

**Trace, don't read.** Pick a path, follow it end-to-end, verify every step exists in the design. A missing step is a gap.

**Enumerate before tracing.** List every path before tracing any. Prevents premature completion — tracing three paths and declaring done when ten exist.

**Name the consequence.** A finding states the specific runtime consequence: what breaks, what data is lost, what state is orphaned. If you cannot name it, it is not a finding.

**Flag ambiguity, don't guess.** If the design is ambiguous about whether something is a state, a data item, an interface, or out of scope — report it as an ambiguity (cite location, state interpretations, state trace impact) before tracing.

## Phase 0: Build the map

Before selecting dimensions or tracing, inventory everything the design contains. Without this, you trace only what you noticed.

Scan the design and produce a flat inventory with five lists:

- **Data items** — every named data structure, field, message, or artifact created, transported, stored, or consumed.
- **States** — every status, phase, mode, or lifecycle stage: any value that changes over time.
- **Interfaces** — every method, function, endpoint, or callable unit, including ABC methods, factory functions, framework callbacks.
- **Objects** — every in-memory holder, store, cache, or container with a lifespan.
- **Concerns** — every behavior handled more than one way: fallback or default branches; "if X is None" guards; two mechanisms for the same concern; coordinator + bypass paths.

For each item: name, design-section location, one-line role.

**Scope boundary.** Items inside the design's defined components are in-scope. External systems (databases, queues, third-party APIs) are terminal boundaries — record as "exits to external system X," do not trace further.

Write the map to a file.

**Completion.** Five lists produced, every design item accounted for. An item traced later that isn't in the map means the map was incomplete — rebuild it.

## Phase 1: Trace dimensions

Select dimensions relevant to the design. Select if ANY signal matches:

- **data-flow** (`references/data-flow.md`) — a data structure crosses a component boundary; data is persisted or queued; a pipeline or transport is described.
- **state-machine** (`references/state-machine.md`) — an enum/field with >2 values; a transition verb (enter/exit/suspend/resume); a recovery or retry flow.
- **interface** (`references/interface.md`) — an ABC or protocol; a factory or registry; multi-layer wiring; framework callbacks.
- **lifecycle** (`references/lifecycle.md`) — per-instance objects; a shared store or cache; suspend/resume of in-memory state; cross-boundary object survival.
- **convergence** (`references/convergence.md`) — fallback or default branches; "if X is None" guards; two mechanisms for the same concern; coordinator + bypass paths.

Load the reference for each selected dimension. Trace every path per its reference's checklists. Write each path as a row in the closure matrix (see Output format).

### Parallelize for complex designs

For large or multi-file designs, dispatch each selected dimension to a parallel subagent using `templates/dimension-tracer.md`. Fill the `{design_doc}`, `{map_file_path}`, `{dimension_name}`, and `{dimension_reference_content}` placeholders before dispatching. Each subagent returns its matrix rows and findings. You merge. If a subagent returns incomplete or inconsistent results, re-trace that dimension yourself serially.

Cross-dimension seams (Phase 2) and finding verification (Phase 3) stay with you — never delegate them.

**Completion.** Every selected dimension's reference completion criterion met. Every path in the dimension has a matrix row. Dimension selection record written (selected, skipped, why).

## Phase 2: Trace seams

After each dimension is traced, trace the seams between them. Real gaps live at seams — each dimension closes, but the system has a gap no dimension owns.

For each pair of selected dimensions, check whether A's terminal artifact depends on B's artifact:

| Seam | Check |
|------|-------|
| data-flow ↔ lifecycle | If lifecycle releases the holder, what happens to data in-flight? |
| state-machine ↔ data-flow | If data-flow persistence is broken, does state-machine recovery still close? |
| interface ↔ lifecycle | If lifecycle releases the holder, do interface callers still have a valid reference? |
| state-machine ↔ lifecycle | If lifecycle releases an object mid-transition, what happens to the state? |
| data-flow ↔ interface | If the interface returns unexpected data, does consumption still close? |
| error propagation (cross-cutting) | When any dimension's abnormal path triggers, trace the error across ALL dimensions — where it lands, what state it leaves, who handles it. |

For each seam: if A's closure depends on B's artifact, verify B's artifact is closed AND the dependency holds. A broken seam is a finding like any other. If a pair isn't listed but A's closure depends on B's artifact, trace that seam too — the table covers common cases, not the exhaustive set.

**Completion.** Every selected dimension pair's seam checked. Never delegate — seams require all dimensions' results.

## Phase 3: Verify findings

Before reporting, re-read each finding's cited design location and confirm:

1. The location exists and says what you claim it says.
2. Each link in the consequence chain references something in the design (not invented).
3. The proposed fix addresses the traced gap, not a different concern.

If a finding fails any check, drop it.

**Completion.** Every finding re-verified against the cited location. Unverified findings dropped.

## The bar

A finding is valid only if ALL three hold:

1. **Specific location** — cite the design section and the exact path.
2. **Named consequence** — state what breaks at runtime: "X is never called because Y, so Z accumulates and recovery re-consumes them, causing double-effect." Not "might be problematic."
3. **Concrete fix** — propose a specific change: "add X call in step Y." Not "consider improving the flow."

A finding that fails any is noise. Drop it. Either the path closes or it doesn't — "minor issue" and "nice to have" do not exist.

**Suspected gaps.** A path that does not close but whose consequence you cannot fully articulate. Report as a suspected gap: cite the location, state what you traced, state what's missing, state why the consequence is unclear. Suspected gaps are a separate output category — they stay suspected, never promoted to findings or dropped to meet the bar.

## Output format

**Closure matrix** — one row per path. Columns: `dimension | path | checkpoints | status | note`

Status values:
- `closed` — all checkpoints connect end-to-end.
- `gap` — a checkpoint is missing or broken; linked finding ID.
- `assumption-closed` — path closes IF a stated assumption holds; record the assumption.
- `deferred` — design explicitly defers; record deferral target.
- `external` — path exits to external system; terminal, not traced.

**Findings** — one per `gap` row. Each meets the bar (location + consequence + fix).

**Suspected gaps** — separate list. Each: location + what's traced + what's missing + why consequence is unclear.

**Ambiguities** — separate list. Each: location + interpretations + trace impact.

**Dimension selection record** — selected, skipped, why.

## Completion criterion

Verification is complete when ALL hold:

1. Every structure-map item appears in ≥1 closure-matrix row.
2. Every row has a status (no blanks).
3. Every `gap` row has a finding meeting the bar.
4. Every `assumption-closed` row has its assumption recorded.
5. Cross-dimension seams (Phase 2) traced for all selected dimension pairs.
6. Every finding re-verified (Phase 3).

A map item with no matrix row = incomplete work. A matrix row with no status = incomplete work. The matrix is the proof — "I traced everything" without it is a claim, not verification.

A genuine design gap is a finding, not incomplete verification — the verification is complete when the gap is found and reported.

## Execution: context management

Tracing multiple dimensions with the design doc in context fails for non-trivial designs.

- Write the map and closure matrix to files — append each row immediately after tracing a path, not at the end.
- Load one dimension reference at a time. Trace fully, write rows, unload before loading the next.
- Keep the design doc in context throughout. If it exceeds ~40% of context, summarize inactive sections and keep only the active section in full.
- If serial tracing still exceeds context: dispatch dimensions to parallel subagents (see Phase 1).

## Scope boundaries

This skill verifies **logical closure** — every path connects end-to-end. It does NOT verify:

- **Concurrency safety** — race conditions, deadlocks, ordering guarantees. Verify separately.
- **Implementation quality** — error-handling style, performance, idiomatic code. Flag only if a logical path is broken.
- **External system correctness** — systems outside the design are terminal boundaries, assumed correct.
