# Lifecycle Closure

## What to trace

For every in-memory object in the design (coordinator, store, context, consumer, connection, cache, registry, controller):

### Holder
- Who holds the reference?
- Is the holder per-instance, per-run, or shared across instances?
- Is the holder's own lifecycle defined (who holds the holder)?

### Duration
- How long is the object held?
- Is the duration correct for the object's purpose?
  - Per-run: should survive only one `run_async` call
  - Per-instance: should survive the graph instance's logical lifetime (including suspend)
  - Shared: should survive across instances (justified by cross-instance need)

### Release
- When is the reference released?
- Is there a path where it is never released?
- If released in a `finally` block, does the `finally` cover all exit paths (normal, exception, interrupt)?

### GC path
- When the holder is GC'd, does the downstream object also GC?
- Is there a shared holder that prevents GC (e.g., a shared dict that accumulates entries)?
- If the object holds a resource (SQLite connection, file handle), does GC release it?

### Cross-boundary
- Does the object survive across a boundary it should not?
- Common boundaries: `_execute` return, GraphInterrupt suspend → resume, crash → recovery, run → sub-run (nested graph)
- If it should survive (e.g., suspend), does it? If it should not (e.g., completed instance), does it get released?

## Closure test

An object's lifecycle is **closed** when holder → duration → release → GC path are all defined and consistent.

- An object with no holder is **orphaned** — created but nobody keeps a reference.
- An object held by a shared long-lived store when it should be per-instance is a **leak** — accumulates across instances.
- An object released too early (before suspend/resume completes) **loses state** — in-memory data is GC'd.
- An object held too long **accumulates** — completed instances never cleared.
- An object that survives across a boundary it should not **cross-contaminates** — stale state bleeds into the next run.
- A contradiction in the design (§A says "GraphInstance holds it", §B says "_execute call stack holds it") is a **holder ambiguity** — the two are mutually exclusive.

## Common traps

- **Shared store leak**: orchestrator `__init__` creates shared `deliver_store` / `instance_store`. Multiple GraphInstances share one store. Completed instances' data accumulates. Fix: per-instance stores held by coordinator.
- **Suspend loses in-memory state**: coordinator is `_execute`-local. On GraphInterrupt, `_execute` returns, coordinator is GC'd. InMemory NodeState (with suspended state snapshot) is lost. Resume creates new coordinator with empty state. Fix: coordinator held by GraphInstance, survives suspend.
- **Controller unregistered on suspend**: `finally: unregister_engine(gid)` runs on GraphInterrupt. Controller (referencing coordinator) is removed from `_engines` dict. External commands (pause/stop/deliver) during PAUSED have no target. Fix: distinguish "unregister on complete" from "unregister on suspend".
- **Per-node SQLite connection proliferation**: N nodes × 2 stores = 2N connections. Each `SqliteDeliverStore.__init__` opens its own connection. Fix: shared connection per GraphInstance, passed to factories.
- **Cached CompiledGraph shares Node attributes**: if compile is cached, Nodes are shared across GraphInstances. Per-instance state (deliver_consumer, NodeState) stored as Node attributes cross-contaminates. Fix: per-instance state accessed via `ctx.coordinator`, not Node attributes.

## Completion criterion

Every in-memory object has a defined holder with correct scope (per-run / per-instance / shared-justified). Every object has a release path covering all exit paths. No shared holder accumulates per-instance data. No per-instance object is released before its logical lifetime ends. No object survives across a boundary it should not. No design contradiction exists about who holds the object.
