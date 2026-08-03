# Convergence

## What to trace

For every concern in the design (routing, persistence, input model, lifecycle management, error handling, validation, cleanup, recovery):

### Path count
- Is there one path or multiple for this concern?
- If multiple, list each path with its location in the design.

### Justification
- If multiple paths, are they justified by genuinely different execution models?
- Test: "if you deleted one path and routed its callers through the other, would behavior be identical?" If yes, it is divergence, not justified.
- A "fall back to old behavior if X is None" guard is **not** justified — it is a backward-compat shim for code just written.
- A "different strategy implementation" (Null / InMemory / SQLite) **is** justified — different execution models, same interface.

### Consistency
- Do all paths produce the same observable behavior?
- If one path routes through a store and another through a parameter, do they deliver the same data?
- If one path persists and another doesn't, is the non-persisting path's data loss intentional and documented?

### Interface alignment
- Do all paths use the same ABC?
- Or does each path have its own interface for the same concern?
- If different interfaces, is the difference justified (e.g., sync vs async) or accidental (copy-paste drift)?

## Closure test

A concern is **converged** when it has one path, or multiple paths with execution-model justification, all using the same interface.

- Two paths for one concern without execution-model justification is a **convergence violation** — must converge to one.
- Two paths with different interfaces for the same concern is a **divergence** — must align interfaces.
- A "fall back if X is None" guard for code just written is a **shim** — remove it, use Null implementation instead.
- A concern with two input models (e.g., deliver_store-based vs parameter-based) is a **model divergence** — must converge to one input model.

## Common traps

- **coordinator=None dual path**: Node.run() has coordinator mode (full lifecycle) and退化 mode (coordinator=None, old behavior). Two code paths, two input models. Rule 15 violation. Fix: always pass coordinator, use Null implementation for no-persistence.
- **External deliver bypasses coordinator**: internal delivers go through `coordinator.route_deliver`, external delivers (DELIVER_TO_NODE) go through shared `GraphControlService._deliver_store`. Two paths for the same concern (deliver routing). Fix: external delivers route through coordinator.
- **Two validation mechanisms**: registry validates config via `config_model.model_validate()`, factory also validates manually via `isinstance`. Two mechanisms for one concern. Fix: factory declares config_schema, registry validates, factory trusts.
- **Shared vs per-instance stores**: orchestrator creates shared stores, coordinator creates per-instance stores. Two ownership models for the same concern (store lifecycle). Fix: converge to per-instance.
- **SUBMITTED vs CONSUMED status**: old design marks delivers as SUBMITTED, new design marks as CONSUMED. If the old term remains in the flow description, it is stale residue. Fix: update all references.

## Completion criterion

Every concern has one path, or multiple with execution-model justification. No "fall back if None" guards for code just written. No concern has two input models. No concern has two interfaces. No stale references to removed mechanisms.
