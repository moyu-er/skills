# Data Flow Closure

## What to trace

For every data item in the design (deliver, state_update, invocation record, metadata, payload, event, message, snapshot):

### Origin
- Where is it created? Who creates it?
- Is the creation site defined in the design (method, step, interface)?

### Transport
- How does it move from origin to consumer?
- Through what mechanism (parameter, store, queue, dispatch, route)?
- Is the transport step defined?

### Consumption
- Who reads it? Through what interface?
- Is the consumption site defined?
- Is there a transformation step (integrate, serialize, deserialize)?

### Termination
- Is it cleaned up? How? When?
- Or is it intentionally retained (audit log, version chain, history)?
- If retained, is the retention bounded or unbounded?

## Closure test

A path is **closed** when origin → transport → consumption → termination all exist, connect, and match in format/contract — origin's output type matches transport's input; transport's output matches consumption's input. A transformation step must preserve semantic integrity, not just exist.

- A missing link is a **gap** — data created but never consumed, or consumed but never created.
- A format mismatch is a **contract gap** — origin produces type X, consumption expects type Y; the path "connects" but is semantically broken.
- A termination that never happens is a **leak** — data accumulates without bound.
- A transport step that exists in one mode but not another (e.g., coordinator mode vs退化 mode) is a **divergence** — route to `references/convergence.md`.

## Common traps

- **Asymmetric capture**: suspend captures full state snapshot, but complete only captures `state_update`. If nodes make imperative mutations not in `state_update`, those are lost on recovery.
- **Double dispatch**: `deliver()` routes through one mechanism, `submit()` routes through another. If both fire, data is duplicated.
- **Stale reference in transport**: a dispatch was pending (in metadata) but the target already completed. The dispatch is stale — does re-dispatch create a duplicate?
- **Cross-boundary data loss**: data stored in an in-memory object that is released across a boundary (e.g., `_execute` return → suspend → resume). The data is GC'd with the object.

## Completion criterion

Every data item has origin → termination, all links present and connected. No data item is created without a consumer. No data item is consumed without a creator. Every data item has a defined termination path (cleanup or justified retention).
