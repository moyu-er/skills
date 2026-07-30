# Divergence Patterns

The catalogue of divergence shapes this skill hunts for. Each pattern
names a _structural_ form — the concern that splits — not a domain
specifics. Every codebase has them; the skill classifies each instance
as justified or accidental using the execution-model test.

## Construction paths

The same type of object built through different call sites that thread
different collaborators. E.g. in a session-based system: one path passes
`session_registry`, another passes `None`; one passes `output_adapter`,
another skips it. The result: one path has a capability the other
silently lacks.

**Justified** when the construction is genuinely different (e.g. a
subagent built per-invocation vs a main agent built once at boot —
different _kinds_ of object, even if the same type).

**Accidental** when both paths build the same kind of object for the
same purpose but one forgot to thread a collaborator the other has.

## Wiring mechanisms

The same collaborator (e.g. emitter factory, session registry, config
loader, hook runner) injected through different _mechanisms_ for different
callers. One caller gets it via a post-build wrapper; another gets it via
an explicit setter call; a third gets it via a constructor arg. All
achieve injection, but through different routes that can drift.

**Justified** when the mechanism difference is structural (e.g. one
caller goes through a factory wrapper that also does other post-build
work; the other can't use the wrapper because it bypasses the factory).

**Accidental** when both callers could use the same mechanism but one
was written before the mechanism existed and never updated.

## Persistence paths

The same data written to two stores, or through two adapters, when one
would suffice. E.g. in a session-based system: a parent-child
relationship stored both in `SessionInfo.parent_session_id` (via
`SessionStore`) _and_ in a separate field on `ExternalSessionMapStore` —
two sources of truth for one relationship.

**Justified** when the two stores serve different consumers with
different access patterns (e.g. one is a transactional store for runtime
lookups, the other is an append-only audit log).

**Accidental** when both stores serve the same consumer and the same
query — one is a redundant copy.

## Filtering / dispatch logic

The same filter or dispatch rule (e.g. pool resolution, event routing,
permission check, session-to-emitter mapping) implemented inline in
multiple handlers instead of one shared helper. E.g. each handler has
its own `if agent_name not in pool_map: continue` — same logic,
different sites, no shared function.

**Justified** when the filter logic genuinely differs per handler (e.g.
one handler needs case-insensitive matching, another needs prefix
matching).

**Accidental** when the logic is identical and could be one function
call — the inline copies exist because each handler was written
independently.

## Fallback guards

A new path added alongside an old one, with a `if X is None: fall back
to old behavior` guard — code you just wrote, protected by a backward-
compatibility shim that has no real consumer. The old path stays alive
not because anything needs it, but because removing it felt riskier than
keeping it.

**Justified** when the old path has real callers you haven't migrated
yet (a deprecation timeline exists).

**Accidental** when the old path's callers were already rerouted — the
guard is dead code protecting nothing.

## Import path divergence

The same module imported through different routes at different call
sites — one uses a direct top-level import, another uses a lazy import
inside a function, a third guards it behind `TYPE_CHECKING`. Each route
has a different runtime cost and a different failure mode, but they all
resolve to the same module.

**Justified** when the import route difference is structural (e.g. a
circular dependency forces a lazy import at one site; a `TYPE_CHECKING`
guard is needed for type-only imports that shouldn't exist at runtime).

**Accidental** when all sites could use the same import route but one
was copy-pasted from a different context or written before the module
was promoted to a stable location.
