# Interface Closure

## What to trace

For every method, ABC, factory, and parameter in the design:

### Caller
- Who calls this method? Trace from the call site to the definition.
- If nobody calls it, is it dead code or a hypothetical seam (only one implementation, no second consumer)?

### Parameters
- Are all parameters available at the call site?
- Is the data the method needs in scope when it is called?
- If a parameter is passed through multiple layers, does each layer forward it?

### Return
- Who consumes the return value?
- Does it reach a destination that uses it?
- If the return is ignored, is that intentional (fire-and-forget) or a gap?

### Definition
- Does the design reference a method or type that is never defined?
- Is the method signature complete (types, defaults, keyword-only)?
- If the method is on an ABC, are all abstract methods implemented by concrete classes?

### Wiring
- If the method is called through a coordinator or facade, does the coordinator have access to the target?
- If the method needs a factory, is the factory available at the call site?
- If the method is registered (e.g., register_node), when does registration happen? Is it before the first call?

## Closure test

An interface is **closed** when caller → parameter → method → return → consumer all exist, connect, and match in type/signature — caller's argument type matches parameter; return type matches consumer's expected input.

- A method with no caller is a **hypothetical seam** — only one implementation, no second consumer.
- A parameter never passed at the call site is **dead** — the method expects it but nobody provides it.
- A type mismatch at any link is a **contract gap** — caller passes X, parameter expects Y; the call "connects" but is semantically broken.
- A return value nobody reads is **wasted** — the method produces data but it goes nowhere.
- A referenced-but-undefined method is a **forward-reference gap**.
- A method that needs a coordinator/factory but the call site doesn't have access is a **wiring gap**.
- Registration that happens after the first call is a **timing gap**.

## Common traps

- **Coordinator doesn't expose accessor**: Node.run() calls `self.deliver_consumer`, but coordinator only exposes `get_deliver_store`. The consumer exists conceptually but has no access path.
- **Factory not available at call site**: coordinator's `__init__` takes `DeliverStoreFactory`, but the orchestrator creates coordinator in `_execute` without the factory in scope.
- **Registration timing**: design says "register at compile time", but the compiler doesn't have the coordinator. If compile is cached, registration targets the wrong instance.
- **Parameter removal without caller update**: design removes `enforce_deliver` parameter, but if any caller still passes it, the call breaks.
- **ABC method never called by the framework**: the ABC defines it, concrete classes implement it, but no framework code invokes it. It is dead.

## Completion criterion

Every method has at least one caller. Every parameter is available at every call site. Every return value has a consumer. Every referenced type is defined. Every wiring path (coordinator → target, factory → call site) is connected. Every registration happens before the first call.
