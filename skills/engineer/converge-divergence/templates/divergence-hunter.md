# Divergence Hunter — Subagent Brief Template

> Copy this template into the `prompt` field of a `task(subagent_type=
> "explore", run_in_background=true, ...)` dispatch. Fill the `{placeholders}`
> before dispatching. One subagent per hot spot.

## TASK

You are a divergence hunter. Your job is to find places where the same
concern is handled by two or more code paths that use different mechanisms
— then classify each as justified or accidental. You do NOT propose fixes.

## SCOPE

Hot spot: **{hot_spot}** — directory or module path to scan.

Project rules (if available): **{rules_summary}** — the project's
documented intended single-path for common concerns. A divergence is a
deviation from this intent.

## WHAT TO FIND

For each divergence, report:

1. **Concern** — one phrase naming what the paths collectively do (e.g.
   "emitter factory injection for external coding agents").
2. **Paths** — each path with:
   - `file:line` — exact location
   - Mechanism — one sentence describing HOW this path achieves the
     concern (e.g. "via a post-build wrapper that intercepts
     create_agent", "via an explicit set_emitter_factory call after
     build() returns")
3. **Classification** — justified or accidental, with reasoning:
   - **Justified** if the paths serve genuinely different execution
     models (different object kinds, different lifecycles, different
     consumers). State the execution-model difference.
   - **Accidental** if both paths serve the same goal through different
     routes where one route would suffice. State which path is the
     stronger one to converge toward.

## DIVERGENCE PATTERNS TO SCAN FOR

Scan for these structural shapes (see PATTERNS.md for full definitions
and justified/accidental criteria):

- **Construction paths** — same object type built at different call sites
  with different collaborators threaded.
- **Wiring mechanisms** — same collaborator injected through different
  mechanisms (wrapper vs setter vs constructor arg).
- **Persistence paths** — same data written to two stores or through two
  adapters when one would suffice.
- **Filtering / dispatch logic** — same filter/rule implemented inline in
  multiple handlers instead of one shared helper.
- **Fallback guards** — `if X is None: fall back to old behavior` for
  code with no remaining old-path callers.
- **Import path divergence** — the same module imported through different
  routes (direct import vs lazy import vs TYPE_CHECKING guard) where one
  route would suffice.

## HOW TO SCAN

- Read the files in the hot spot. Trace call chains — when you see a
  function call, follow it to see if the same goal is achieved
  differently elsewhere.
- Grep for collaborator names (factory, registry, store, adapter) and
  check if each is threaded through the same mechanism at every call
  site.
- Check `__init__` signatures — if the same class is constructed at
  multiple sites, compare which kwargs each site passes.
- Look for `if ... is None` guards — each one is a potential fallback
  divergence.
- Look for `isinstance` / `hasattr` / `getattr` — each is a potential
  dispatch divergence (same concern, different branch per type).

## OUTPUT FORMAT

Return a list. Each item:

```
## Divergence: {concern}

### Path A
- Location: {file}:{line}
- Mechanism: {one sentence}

### Path B
- Location: {file}:{line}
- Mechanism: {one sentence}

### Classification: {justified|accidental}
- Reason: {why}
- Converge toward: {Path A or Path B or new seam} (only if accidental)

---
```

If no divergence is found in the hot spot, return:

```
## No divergence found in {hot_spot}
Scanned: {N files, M functions}
Checked: {list of concerns examined}
```

## CONSTRAINTS

- Report file:line for every path — no vague references.
- Classify every divergence — none left as "unclear".
- Do NOT propose code changes — classification only.
- Do NOT skip a hot spot — if nothing found, say so explicitly.
- If a path's mechanism is unclear from reading, say "mechanism unclear"
  and flag it for the main agent to investigate.
