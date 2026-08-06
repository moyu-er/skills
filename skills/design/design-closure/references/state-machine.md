# State Machine Closure

## What to trace

For every state in every state machine in the design (invocation status, instance status, consumption status, connection lifecycle, session lifecycle):

### Entry
- What transition brings the system to this state?
- Is the transition triggered by a defined event (method call, exception, timeout)?

### Exit
- What transition leaves this state?
- Is it terminal (no exit) or transitional (has an exit)?
- Is the exit transition defined in the design?

### Abnormal exit
- What happens on **crash** (unhandled exception)?
- What happens on **suspend** (interrupt, HITL pause)?
- What happens on **cancel** (user stop, GraphBubbleUp)?
- What happens on **supersession** (a new entity replaces one in this state — e.g., new invocation supersedes a suspended one)?

### Recovery
- If the system restarts in this state, what happens?
- Is the state persisted (can it survive restart)?
- If persisted, does recovery distinguish this state from others (e.g., crashed vs paused)?

### Transition atomicity
- For every multi-step transition (e.g., `mark_consumed` then `promote_consumed`), is it atomic?
- If non-atomic, trace the crash-between-steps state for every intermediate point — what does recovery see?

## Closure test

A state machine is **closed** when every state has a defined entry, a defined exit (or is terminal), abnormal-exit handling, recovery semantics, and every multi-step transition's atomicity is verified.

- A state with no entry is **unreachable**.
- A state with no exit (and not terminal) is an **orphan**.
- A state with no abnormal-exit handling is a **crash gap** — the system is stuck in this state on failure.
- A state that blocks a new entity (e.g., "at most one non-terminal row") but has no supersession transition is a **deadlock** — the new entity cannot be created.
- A state that is persisted but has no recovery semantics is a **recovery gap** — restart behavior is undefined.

## Common traps

- **Suspended state without supersession**: a suspended invocation (status=RUNNING) blocks the "at most one non-terminal" rule. When a new invocation is created, the suspended one must transition — but to what? If undefined, deadlock.
- **Crash between state transitions**: crash happens between `mark_consumed` (state A) and `promote_consumed` (state B). The data is in state A, but the state machine expected it in state B. Recovery sees state A — is that correct?
- **Mixed dimensions in one enum**: scheduler states (DORMANT/READY) and invocation states (PENDING/RUNNING/COMPLETED) in one enum. They are different dimensions — mixing them causes type-safety violations and confusing transitions.
- **Recovery doesn't filter by state**: recovery re-dispatches all non-terminal states, but some (CANCELED) are deliberate and should not be re-dispatched.

## Completion criterion

Every state has entry → exit (or terminal). Every state has abnormal-exit handling. Every state has recovery semantics. Every multi-step transition has verified atomicity. No state blocks supersession without a defined transition. No enum mixes states from different dimensions.
